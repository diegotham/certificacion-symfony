Un **firewall** es un **sistema de seguridad de red**, ya sea basado en hardware o software, que controla el tráfico de entrada y salida de una red basándose en una serie de reglas. Actúa como barrera entre una red de confianza (_trusted_) y una de desconfianza (_untrusted_), como **Internet**, de forma que controla el acceso a los _resources_ de una red a través de un **modelo de control**. 

Un **firewall de una aplicación web** es un firewall que **monitoriza, filtra y bloquea el tráfico HTTP** de una aplicación web. El firewall protege a una aplicación controlando los datos de entrada y salida y el acceso a la aplicación. Una buena configuración evita [ataques XSS](http://diego.com.es/ataques-xss-cross-site-scripting-en-php), [ataques a sesiones](http://diego.com.es/seguridad-de-sesiones-en-php), [SQL injection](http://diego.com.es/ataques-sql-injection-en-php), etc.

En esta sección vemos algunos aspectos de los **firewalls en Symfony**.

1.  Firewall y autorización
2.  Restringir firewalls a un request específico
3.  Utilizar firewalls preautenticados

### 1\. Firewall y autorización

La autorización es la parte central del **componente de seguridad de Symfony**. Se maneja por una instancia de [AuthorizationCheckerInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authorization/AuthorizationCheckerInterface.html). Cuando son satisfactorios todos los pasos en el **proceso de autenticar al usuario**, puedes preguntar al **authorization checker** si el usuario autenticado tiene acceso a cierta acción o _resource_ de la aplicación:

```
use Symfony\Component\Security\Core\Authorization\AuthorizationChecker;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;

// instancia de Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface
$tokenStorage = ...;

// instancia de Symfony\Component\Security\Core\Authentication\AuthenticationManagerInterface
$authenticationManager = ...;

// instancia de Symfony\Component\Security\Core\Authorization\AccessDecisionManagerInterface
$accessDecisionManager = ...;

$authorizationChecker = new AuthorizationChecker(
    $tokenStorage,
    $authenticationManager,
    $accessDecisionManager
);

// ... autenticar al usuario

if (!$authorizationChecker->isGranted('ROLE_ADMIN')) {
    throw new AccessDeniedException();
}
```

#### Firewall para requests HTTP

Autenticar a un usuario se hace a través del **firewall**. Una aplicación puede tener múltiples áreas aseguradas, de forma que el firewall se configura empleando un mapa de estas áreas aseguradas. Para cada una de estas áreas, el mapa contiene un **request matcher** y una colección de **listeners**. El request matcher le da al firewall la habilidad de averiguar si el request actual apunta a un área segura. Es entonces cuando se pregunta a los listeners si el request actual puede usarse para autenticar al usuario:

```
use Symfony\Component\Security\Http\FirewallMap;
use Symfony\Component\HttpFoundation\RequestMatcher;
use Symfony\Component\Security\Http\Firewall\ExceptionListener;

$map = new FirewallMap();

$requestMatcher = new RequestMatcher('^/secured-area/');

// instancias de Symfony\Component\Security\Http\Firewall\ListenerInterface
$listeners = array(...);

$exceptionListener = new ExceptionListener(...);

$map->add($requestMatcher, $listeners, $exceptionListener);
```

Se proporciona al firewall el mapa firewall como primer argumento, junto con el **event dispatcher** usado por el **HttpKernel**:

```
use Symfony\Component\Security\Http\Firewall;
use Symfony\Component\HttpKernel\KernelEvents;

// the EventDispatcher used by the HttpKernel
$dispatcher = ...;

$firewall = new Firewall($map, $dispatcher);

$dispatcher->addListener(
    KernelEvents::REQUEST,
    array($firewall, 'onKernelRequest')
);
```

El firewall se registra para atender al evento _kernel.request_ que será lanzado por el **HttpKernel** al principio de cada request que procesa. De esta forma, el firewall puede evitar que el usuario avance más de lo permitido.

#### Firewall Listeners

Cuando se notifica al firewall del evento _kernel.request_, pregunta al mapa firewall si el request coincide con alguna de las áreas aseguradas. La primera área asegurada que coincida con el request devolverá un conjunto de **firewall listeners** (que implementan **ListenerInterface**). Se solicitará a estos listeners que manejen el request actual, de forma que averiguen si el request actual contiene información con la que el usuario pueda ser autenticado (por ejemplo el **authentication listener Basic HTTP** comprueba si el request tiene un header llamado PHP_AUTH_USER). 

#### Exception Listener

Si cualquiera de estos listeners lanza una [AuthenticationException](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Exception/AuthenticationException.html), el **exception listener** proporcionado cuando se añaden las áreas aseguradas saltará.

El **exception listener** determina qué ocurrirá después. basándose en los argumentos que recibió cuando se creó. Puede empezar el **procedimiento de autenticación**, por ejemplo preguntar al usuario que introduzca de nuevo sus credenciales (cuando se han autenticado a través de la cookie "Recordarme"), o transformar la excepción en una [AccessDeniedHttpException](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/Exception/AccessDeniedHttpException.html), que resultará en una respuesta _HTTP/1.1 403: Access Denied_. 

#### Entry Points

Cuando el usuario no está identificado de ninguna forma (por ejemplo cuando el **token storage** no tiene un token todavía), se puede llamar al punto de entrada del firewall para comenzar el proceso de autenticación. Un punto de entrada debe implementar [AuthenticationEntryPointInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Http/EntryPoint/AuthenticationEntryPointInterface.html), que tiene sólo un método: [start()](http://api.symfony.com/3.0/Symfony/Component/Security/Http/EntryPoint/AuthenticationEntryPointInterface.html#method_start). Este método recibe el **objeto Request** actual y la **excepción** por la cual fue lanzado el exception listener. El método debe devolver un **objeto Response**. Este puede ser, por ejemplo, la página que contiene el **formulario de login**, or, en el caso de **autenticación Basic HTTP**, una respuesta con un header www-Authenticate, que mostrará la ventana para proporcionar el usuario y password. 

#### Flow: Firewall, Authentication, Authorization

El _flow_ del **sistema de seguridad** es como sigue:

1.  El firewall se registra como un **listener** en el evento _kernel.request_.
2.  Al principiodel **request**, el firewall comprueba el **mapa firewall** para ver si debería activarse cualquier firewall para esta URL.
3.  Si se encuentra un firewall en el mapa para esta URL, se notifica a sus listeners.
4.  Cada listener comprueba si el request actual contiene cualquier información de autenticación, un listener puede:
    1.  Autenticar a un usuario.
    2.  Lanzar una AuthenticationException.
    3.  No hacer nada (porque no hay información de autenticación en el request).
5.  Una vez que el usuario ha sido autenticado, puedes emplear **autorización** para denegar el acceso a ciertos resources.

### 2\. Restringir firewalls a un request específico

Cuando se utiliza el componente de seguridad, puedes crear firewalls que coincidan con cierdas opciones del request. En la mayoría de los casos, coincidir con la URL es suficiente, pero en casos especiales puedes restringir todavía más el proceso con más opciones. 

Puedes emplear las opciones individualmente o juntas para obtener la configuración deseada.

#### Restringir por patrón

Esta es la restricción por defecto y restringe que un firewall sólo se inicie si la **URL del request** coincide con el patrón configurado:

```
# app/config/security.yml

# ...
security:
    firewalls:
        secured_area:
            pattern: ^/admin
            # ...
```

El patrón es una **expresión regular**. En este ejemplo, el firewall sólo se activará si la URL comienza (debido al carácter ^) con /admin. Si la URL no coincide con este patrón, el firewall no se activará y los firewalls subsecuentes tendrán la oportunidad de coincidir con este request.

#### Restringir por host

Si con el patrón no es suficiente, el request también puede coincidir con el **host**. Cuando se establece la opción de configuración _host_, el firewall se restringirá a sólo iniciarse si el host del request coincide con el de la configuración:

```
# app/config/security.yml

# ...
security:
    firewalls:
        secured_area:
            host: ^admin\.example\.com$
            # ...
```

El **host** (como el patrón) es una **expresión regular**. Si el **hostname** no es exactamente admin.example.com, no se activará el firewall y los firewalls subsecuentes tendrán la oportunidad de coincidir con este request.

#### Restringir por métodos HTTP

La opción de configuración _methods_ restringe el inicio del firewall a [métodos HTTP](http://diego.com.es/metodos-http):

```
# app/config/security.yml

# ...
security:
    firewalls:
        secured_area:
            methods: [GET, POST]
            # ...
```

En este ejemplo el firewall sólo será activado si el **método HTTP del request** es **GET** o **POST**. De nuevo, como en los anteriores, si los métodos HTTP no son GET o POST, no se activará el firewall y los firewalls subsecuentes tendrán la oportunidad de coincidir con este request. 

### 3\. Utilizar firewalls preautenticados

Algunos **web servers** proporcionan **módulos de autenticación**, incluyendo **Apache**. Estos módulos generalmente establecen algunas variables de entorno que pueden usarse para determinar qué usuario está accediendo a tu aplicación. Symfony soporta la mayoría de mecanismos de autenticación. Estos requests se denominan _pre authenticated requests_ porque el usuario ya está autenticado cuando llega a la aplicación.

#### X.509 Client Certificate Authentication

Cuando se usan certificados de clientes, tu webserver hace el trabajo de autenticación. Con Apache, por ejemplo, emplearías la directiva _SSLVerifYClient Require_.

Activa la x509 authentication para un firewall particular en la configuración de seguridad:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        secured_area:
            pattern: ^/
            x509:
                provider: your_user_provider
```

Por defecto, el firewall proporciona la variable **SSL_CLIENT_S_DN_Email** al user provider, y establece el **SSL_CLIENT_S_DN** como credenciales en el [PreAuthenticatedToken](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Token/PreAuthenticatedToken.html). Puedes sobreescribir este ajuste estableciendo las keys _user_ y _credentiales_ en la **configuración del firewall x509**.

Un **authentication provider** sólo informará al **user provider** del nombre de ususario que realizó el request. Necesitarás crear (o usar) un user provider referenciado por el parámetro de configuración _provider_ (_your_user_provider_ en el ejemplo). Este provider transformará el nombre de usuario en el objeto User elegido.

#### REMOTE_USER Based Authentication

Muchos módulos de autenticación, como _auth_kerb_ para **Apache** proporcionan el nombre de usuario empleando la variable de entorno **REMOTE_USER**. Se puede confiar en esta variable ya que la autorización ocurrió antes de que le alzance el request. 

Para configurar Symfony para que utilice la variable de entorno REMOTE_USER, activa el firewall correspondiente en la configuración de seguridad:

```
# app/config/security.yml
security:
    firewalls:
        secured_area:
            pattern: ^/
            remote_user:
                provider: your_user_provider
```

El firewall entonces proporcionará la variable de entorno REMOTE_USER a tu user provider. Puedes cambiar el nombre de variable estableciendo la key _user_ en la configuración del firewall _remote_user_.

Al igual que en la **autenticación x509**, habrá que configurar un user provider.