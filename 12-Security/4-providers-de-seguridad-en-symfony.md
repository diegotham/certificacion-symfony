Los **providers** permiten organizar y reutilizar partes de una aplicación. En esta sección vamos a ver cómo crear un **provider de autenticación** y otro de **usuarios**, además de ver cómo aplicar **múltiples providers**. A pesar de que ya existen providers por defecto y bundles de autenticación y usuarios, saber cómo se construyen ayuda a entender el concepto de los providers con profundidad.

**Indice de contenido**

1.  Authentication provider
2.  User provider
3.  Múltiples User providers

### 1\. Authentication provider

Lo siguiente es un ejemplo de un **authentication provider** para autenticación [WSSE](http://www.xml.com/pub/a/2003/12/17/dive.html). WSSE es un conjunto de especificaciones de seguridad para [web services](http://diego.com.es/introduccion-a-los-web-services), como **REST** o **SOAP**. El protocolo WSSE proporciona varios beneficios:

1.  Encriptación de usuario y password
2.  Seguridad frente a ataques de repetición
3.  No hace falta configuración en el servidor

Lo básico de WSSE en su adaptación a Symfony es que se encuentran las credenciales encriptadas en el **request header**, se verifican mediante un **timestamp** y un **nonce**, y se autentica al usuario solicitado mediante un **password digest**.

#### El Token

El rol del **token** en el contexto se la seguridad de Symfony es importante. Un token representa los **datos de autenticación del usuario presentes en el request**. Una vez que un request es autenticado, el token permanece en los datos del usuario, y se envían los datos al sistema de seguridad. Primero se crea una **clase Token**. Esto permitirá pasar toda la información relevante al **authentication provider**:

```
// src/AppBundle/Security/Authentication/Token/WsseUserToken.php
namespace AppBundle\Security\Authentication\Token;

use Symfony\Component\Security\Core\Authentication\Token\AbstractToken;

class WsseUserToken extends AbstractToken
{
    public $created;
    public $digest;
    public $nonce;

    public function __construct(array $roles = array())
    {
        parent::__construct($roles);

        // Si el usuario tiene roles, considéralo autenticado
        $this->setAuthenticated(count($roles) > 0);
    }

    public function getCredentials()
    {
        return '';
    }
}
```

La clase WsseUserToken extiende la clase del componente de seguridad [AbstractToken](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Token/AbstractToken.html), que proporciona **funcionalidad básica para tokens**. Implementa [TokenInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Token/TokenInterface.html) en cualquier clase para usar como token.

#### El listener

Ahora necesitaremos un listener que atienda al firewall. El listener es responsable de filtrar los requests al firewall y llamar al authentication provider. Un listener debe ser una instancia de [ListenerInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Http/Firewall/ListenerInterface.html). Un **listener de seguridad** debe manejar el evento **GetResponseEvent** y establecer un token autenticado en el **token storage** si es exitoso:

```
// src/AppBundle/Security/Firewall/WsseListener.php
namespace AppBundle\Security\Firewall;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\Security\Core\Authentication\AuthenticationManagerInterface;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Http\Firewall\ListenerInterface;
use AppBundle\Security\Authentication\Token\WsseUserToken;

class WsseListener implements ListenerInterface
{
    protected $tokenStorage;
    protected $authenticationManager;

    public function __construct(TokenStorageInterface $tokenStorage, AuthenticationManagerInterface $authenticationManager)
    {
        $this->tokenStorage = $tokenStorage;
        $this->authenticationManager = $authenticationManager;
    }

    public function handle(GetResponseEvent $event)
    {
        $request = $event->getRequest();

        $wsseRegex = '/UsernameToken Username="([^"]+)", PasswordDigest="([^"]+)", Nonce="([^"]+)", Created="([^"]+)"/';
        if (!$request->headers->has('x-wsse') || 1 !== preg_match($wsseRegex, $request->headers->get('x-wsse'), $matches)) {
            return;
        }

        $token = new WsseUserToken();
        $token->setUser($matches[1]);

        $token->digest   = $matches[2];
        $token->nonce    = $matches[3];
        $token->created  = $matches[4];

        try {
            $authToken = $this->authenticationManager->authenticate($token);
            $this->tokenStorage->setToken($authToken);

            return;
        } catch (AuthenticationException $failed) {
            // ... puedes guardar en logs cosas aquí

            // Para denegar la autenticación limpia el token. Esto redireccionará a la página de login.
            // Asegúrate de sólo limpiar el token, no aquellos de otros listeners de autenticación.
            // $token = $this->tokenStorage->getToken();
            // if ($token instanceof WsseUserToken && $this->providerKey === $token->getProviderKey())
            // {
            //     $this->tokenStorage->setToken(null);
            // }
            // return;
            }

        // Por defecto deniega la autorización
        $response = new Response();
        $response->setStatusCode(Response::HTTP_FORBIDDEN);
        $event->setResponse($response);
    }
}
```

El listener comprueba el request para el header esperado **X-WSSE**, enlaza el valor devuelto con la informatión WSSE esperada, crea el token empleando esa información y pasa el token al **authentication manager**. Si no se proporciona la información adecuada, o el authentication manager lanza un **AuthenticationException**, devuelve una respuesta 403.

Una clase que no se emplea en el ejemplo anterior, [AbstractAuthenticationListener](http://api.symfony.com/3.0/Symfony/Component/Security/Http/Firewall/AbstractAuthenticationListener.html), es una clase base muy útil que proporciona funcionalizdad común necesaria en extensiones de seguridad. Esto incluye mantener el token en la **sesión**, proporcionar **handlers** de éxito/fracaso, **URLs** de formularios de login, etc.

Volver del listener prematuramente sólo es relevante si queremos **encadenar authentication providers** (como permitir usuarios anónimos). Si quieres prohibir el acceso a usuarios anónimos y devolver un **error 403**, deberías establecer el código de estado antes de hacer return.

#### El authentication provider

El authentication provider hará la verificación el **WsseUserToken**. El provider verificará si el valor del header **Created** es válido por 5 minutos, el valor del header **Nonce** es único por 5 minutos, y el valor del header **PasswordDigest** coincide con el password del usuario:

```
// src/AppBundle/Security/Authentication/Provider/WsseProvider.php
namespace AppBundle\Security\Authentication\Provider;

use Symfony\Component\Security\Core\Authentication\Provider\AuthenticationProviderInterface;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Core\Exception\NonceExpiredException;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use AppBundle\Security\Authentication\Token\WsseUserToken;

class WsseProvider implements AuthenticationProviderInterface
{
    private $userProvider;
    private $cacheDir;

    public function __construct(UserProviderInterface $userProvider, $cacheDir)
    {
        $this->userProvider = $userProvider;
        $this->cacheDir     = $cacheDir;
    }

    public function authenticate(TokenInterface $token)
    {
        $user = $this->userProvider->loadUserByUsername($token->getUsername());

        if ($user && $this->validateDigest($token->digest, $token->nonce, $token->created, $user->getPassword())) {
            $authenticatedToken = new WsseUserToken($user->getRoles());
            $authenticatedToken->setUser($user);

            return $authenticatedToken;
        }

        throw new AuthenticationException('La autenticación WSSE ha fallado.');
    }

    /**
     * This function is specific to Wsse authentication and is only used to help this example
     *
     * For more information specific to the logic here, see
     * https://github.com/symfony/symfony-docs/pull/3134#issuecomment-27699129
     */
    protected function validateDigest($digest, $nonce, $created, $secret)
    {
        // Comprueba que la fecha de creación no es futura
        if (strtotime($created) > time()) {
            return false;
        }

        // Expira el timestamp después de 5 minutos
        if (time() - strtotime($created) > 300) {
            return false;
        }

        // Valida que el nonce no se ha usado en los últimos 5 minutos
        // si se ha usado, podría ser un ataque de repetición
        if (file_exists($this->cacheDir.'/'.$nonce) && file_get_contents($this->cacheDir.'/'.$nonce) + 300 > time()) {
            throw new NonceExpiredException('Detectado un nonce usado previamente');
        }
        // Si el directorio cache no existe lo creamos
        if (!is_dir($this->cacheDir)) {
            mkdir($this->cacheDir, 0777, true);
        }
        file_put_contents($this->cacheDir.'/'.$nonce, time());

        // Validar Secret
        $expected = base64_encode(sha1(base64_decode($nonce).$created.$secret, true));

        return hash_equals($expected, $digest);
    }

    public function supports(TokenInterface $token)
    {
        return $token instanceof WsseUserToken;
    }
}
```

La **AuthenticationProviderInterface** requiere un método _authenticate_ en el token del usuario, y un método _supports_, que le dice al authentication manager si usar o no este provider para el token dado. En el caso de **múltiples providers**, el authentication manager moverá al siguiente provider de la lista. 

#### La Factory

Ya hemos creado el **token**, el **listener** y el **provider**. Ahora tenemos que unirlos. Para hacer que un único provider esté disponible para todos los firewalls necesitamos una **factory**. Una factory es una pieza clave en el **componente de seguridad**, al que le proporciona el nombre del provider y cualquier opción de configuración necesaria. Primero tenemos que crear una clase que implemente [SecurityFactoryInterface](http://api.symfony.com/3.0/Symfony/Bundle/SecurityBundle/DependencyInjection/Security/Factory/SecurityFactoryInterface.html):

```
// src/AppBundle/DependencyInjection/Security/Factory/WsseFactory.php
namespace AppBundle\DependencyInjection\Security\Factory;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Reference;
use Symfony\Component\DependencyInjection\DefinitionDecorator;
use Symfony\Component\Config\Definition\Builder\NodeDefinition;
use Symfony\Bundle\SecurityBundle\DependencyInjection\Security\Factory\SecurityFactoryInterface;

class WsseFactory implements SecurityFactoryInterface
{
    public function create(ContainerBuilder $container, $id, $config, $userProvider, $defaultEntryPoint)
    {
        $providerId = 'security.authentication.provider.wsse.'.$id;
        $container
            ->setDefinition($providerId, new DefinitionDecorator('wsse.security.authentication.provider'))
            ->replaceArgument(0, new Reference($userProvider))
        ;

        $listenerId = 'security.authentication.listener.wsse.'.$id;
        $listener = $container->setDefinition($listenerId, new DefinitionDecorator('wsse.security.authentication.listener'));

        return array($providerId, $listenerId, $defaultEntryPoint);
    }

    public function getPosition()
    {
        return 'pre_auth';
    }

    public function getKey()
    {
        return 'wsse';
    }

    public function addConfiguration(NodeDefinition $node)
    {
    }
}
```

El **SecurityFactoryInterface** requiere los siguientes métodos:

*   **create**. Método que añade el listener y el authentication provider para el **Dependency Injection container** para el contexto de seguridad apropiado.
*   **getPosition**. Devuelve cuándo ha de llamarse al provider. Puede ser _pre_auth_, _form_, _http_ o _remember_me_.
*   **getKey**. Método que define la **key de configuración** empleada para referenciar al **provider** en la configuración del firewall.
*   **addConfiguration**. Método empleado para definir las **opciones de configuración** bajo la **key de configuración** en la **configuración de seguridad**. Se explica más adelante.

Una clase que no se emplea en este ejemplo es [AbstractFactory](http://api.symfony.com/3.0/Symfony/Bundle/SecurityBundle/DependencyInjection/Security/Factory/AbstractFactory.html), una clase base muy útil que proporciona funcionalidad común para **factories de seguridad**. Puede ser útil cuando se define un authentication provider de otro tipo.

Ahora que hemos creado la clase factory, la key wsse puede usarse como firewall en la configuración de la seguridad.

La razón por la que se necesita una clase especial **factory** para añadir **listeners** y **providers** al **contenedor de inyección de dependencias** es que el **firewall** se puede emplear múltiples veces, para asegurar diferentes partes de la aplicación. Por esto, cada vez que se usa el firewall, se crea un nuevo _service_ en el contenedor. La factory es lo que crea estos _services_.

#### Configuración

Ahora veremos al **authentication provider** en acción. Lo primero es añadir los services anteriores al contenedor. La clase factory anterior hace referencia a _ids_ de services que no existen todavía: _wsse.security.authentication.provider_ y _wsse.security.authentication.listener_. Es hora de definir esos services:

```
# app/config/services.yml
services:
    wsse.security.authentication.provider:
        class: AppBundle\Security\Authentication\Provider\WsseProvider
        arguments:
            - '' # User Provider
            - '%kernel.cache_dir%/security/nonces'
        public: false

    wsse.security.authentication.listener:
        class: AppBundle\Security\Firewall\WsseListener
        arguments: ['@security.token_storage', '@security.authentication.manager']
        public: false
```

Ahora que hemos definido los services, le informamos al sistema de seguridad acerca de la nueva factory:

```
// src/AppBundle/AppBundle.php
namespace AppBundle;

use AppBundle\DependencyInjection\Security\Factory\WsseFactory;
use Symfony\Component\HttpKernel\Bundle\Bundle;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class AppBundle extends Bundle
{
    public function build(ContainerBuilder $container)
    {
        parent::build($container);

        $extension = $container->getExtension('security');
        $extension->addSecurityListenerFactory(new WsseFactory());
    }
}
```

Ya está, ahora ya podemos definir partes de la aplicación bajo protección WSSE:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        wsse_secured:
            pattern:   ^/api/
            stateless: true
            wsse:      true
```

#### Añadir una opción de configuración

Podemos mejorar un poco más el authentication provider anterior permitiendo opciones en la key _wsse_. Por ejemplo, el tiempo permitido antes de que expire el header Created, por defecto es 5 minutos. Vamos a hacerlo configurable, de forma que firewalls diferentes puedan tener diferentes tiempos.

Primero necesitamos editar **WsseFactory** y definir una nueva opción en el método _addConfiguration_. 

```
class WsseFactory implements SecurityFactoryInterface
{
    // ...

    public function addConfiguration(NodeDefinition $node)
    {
      $node
        ->children()
        ->scalarNode('lifetime')->defaultValue(300)
        ->end();
    }
}
```

Ahora en el método _create_ de la factory, el argumento _$config_ contendrá un key _lifetime_, establecido a 5 minutos (300 segundos) a no ser que se establezca en la configuración. Pasa este argumento al authentication provider para usarlo:

```
class WsseFactory implements SecurityFactoryInterface
{
    public function create(ContainerBuilder $container, $id, $config, $userProvider, $defaultEntryPoint)
    {
        $providerId = 'security.authentication.provider.wsse.'.$id;
        $container
            ->setDefinition($providerId,
              new DefinitionDecorator('wsse.security.authentication.provider'))
            ->replaceArgument(0, new Reference($userProvider))
            ->replaceArgument(2, $config['lifetime']);
        // ...
    }

    // ...
}
```

Necesitaremos añadir un tercer argumento a _wsse.security.authentication.provider_, que puede estar vacío, pero que se rellenará con el lifetime de la factory. La clase WsseProvider también necesitará aceptar un tercer argumento constructor, lifetime, que se tendría que usar en lugar de los 300 segundos. Estos dos pasos no se muestran aquí.

Ahora el lifetime de cada **WSSE request** es configurable, y puede establecerse cualquier valor deseado:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        wsse_secured:
            pattern:   ^/api/
            stateless: true
            wsse:      { lifetime: 30 }
```

### 2\. User Provider

Parte del **proceso de autenticación standard de Symfony** depende de **user providers**. Cuando un usuario envía un nombre de usuario y password, el **authentication layer** le pide al **user provider** configurado que devuelva un objeto user para un nombre de usuario dado. Symfony entonces comprueba si el password de este usuario es correcto y genera un **token de seguridad** de forma que el usuario permanece autenticado en la sesión actual. Por defecto, Symfony incorpora dos user providers: un _in_memory_ y un _entity_.

Ahora vamos a crear un user provider propio donde los usuarios tienen el acceso en un web service (aunque también puede hacerse para que lo tengan en una base de datos o un archivo).

#### Crear la clase User

No importa de donde vengan los datos de los usuarios, tendremos que crear igualmente una clase User que represente a esos datos. La clase puede ser como quieras, el único requisito es que la clase implemente [UserInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/User/UserInterface.html). Los métodos obligatorios son: _getRoles()_, _getPassword()_, _getSalt()_, _getUsername()_, _eraseCredentials()_. Puede resultar útil implementar [EquatableInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/User/EquatableInterface.html), que define un método para comprobar si el usuario equivale al actual. Esta interface requiere el método _isEqualTo()_.

```
// src/AppBundle/Security/User/WebserviceUser.php
namespace AppBundle\Security\User;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\EquatableInterface;

class WebserviceUser implements UserInterface, EquatableInterface
{
    private $username;
    private $password;
    private $salt;
    private $roles;

    public function __construct($username, $password, $salt, array $roles)
    {
        $this->username = $username;
        $this->password = $password;
        $this->salt = $salt;
        $this->roles = $roles;
    }

    public function getRoles()
    {
        return $this->roles;
    }

    public function getPassword()
    {
        return $this->password;
    }

    public function getSalt()
    {
        return $this->salt;
    }

    public function getUsername()
    {
        return $this->username;
    }

    public function eraseCredentials()
    {
    }

    public function isEqualTo(UserInterface $user)
    {
        if (!$user instanceof WebserviceUser) {
            return false;
        }

        if ($this->password !== $user->getPassword()) {
            return false;
        }

        if ($this->salt !== $user->getSalt()) {
            return false;
        }

        if ($this->username !== $user->getUsername()) {
            return false;
        }

        return true;
    }
}
```

Si tienes más información acerca de los usuarios puedes añadir propiedades a la clase.

#### Crear el User Provider

Ahora que ya tenemos la **clase User**, creamos el **user provider**, que obtendrá información del usuario de algún **web service**, creará el objeto **WebserviceUser** y lo rellenará con datos:

El user provider es sólo una **clase PHP** que tiene que implementar [UserProviderInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/User/UserProviderInterface.html), la cual exige el uso de tres métodos: _loadUserByUsername($username)_, _refreshUser(UserInterface $user)_, y _supportsClass($class)_.

```
// src/AppBundle/Security/User/WebserviceUserProvider.php
namespace AppBundle\Security\User;

use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\Exception\UnsupportedUserException;

class WebserviceUserProvider implements UserProviderInterface
{
    public function loadUserByUsername($username)
    {
        // llamar al service aquí
        $userData = ...
        // suponemos que devuelve un array, o false si no existe usuario

        if ($userData) {
            $password = '...';

            // ...

            return new WebserviceUser($username, $password, $salt, $roles);
        }

        throw new UsernameNotFoundException(
            sprintf('El usuario "%s" no existe.', $username)
        );
    }

    public function refreshUser(UserInterface $user)
    {
        if (!$user instanceof WebserviceUser) {
            throw new UnsupportedUserException(
                sprintf('Instancias de "%s" no están soportadas.', get_class($user))
            );
        }

        return $this->loadUserByUsername($user->getUsername());
    }

    public function supportsClass($class)
    {
        return $class === 'AppBundle\Security\User\WebserviceUser';
    }
}
```

#### Crear el service para el User Provider

Ahora hacemos disponible al user provider como service:

```
# app/config/services.yml
services:
    app.webservice_user_provider:
        class: AppBundle\Security\User\WebserviceUserProvider
```

Una configuración real del user provider probablemente tenga algunas dependencias, opciones de configuración u otros services, por lo que tendrían que añadirse.

#### Modificar la configuración de seguridad

Tenemos que unir lo anterior en la configuración de seguridad. Añadimos el user provider a la **lista de providers** de la sección _security_. Elegimos un nombre para el user provider (como _webservice_) y facilitamos el _id del service_ que acabamos de definir:

```
# app/config/security.yml
security:
    # ...

    providers:
        webservice:
            id: app.webservice_user_provider
```

**Symfony** también necesita saber cómo codificar passwords que son proporcionados por usuarios web (como mediante un **formulario login**). Podemos hacerlo añadiendo una línea a la sección _encoders_ en la configuración de seguridad:

```
# app/config/security.yml
security:
    # ...

    encoders:
        AppBundle\Security\User\WebserviceUser: bcrypt
```

El valor aquí debe corresponder con cómo se hayan codificado originariamente los passwords cuando se crearon los usuarios. Cuando un usuario envía su password, se codifica empleando este algoritmo y el resultado se compara con el password hasheado devuelto por el método _getPassword()_. 

**Symfony** emplea un método específico para combinar el _salt_ y codificar el password antes de compararlo con el password codificado. Si _getSalt()_ no devuelve nada, el password enviado se codifica simplemente usando el algoritmo especificado en _security.yml_. Si se facilita un _salt_, se crea el siguiente valor y se hashea con el algoritmo:

```
$password.'{'.$salt.'}'
```

Si tus usuarios externos tienen sus contraseñas _salteadas_ con un método diferente, tendrás que hacer algo más para que Symfony codifique apropiadamente la contraseña. Esto no se explica aquí, pero incluiría la subclase **MessageDigestPasswordEncoder** y sobreescribiría el método _mergePasswordAndSalt_.

Además puedes configurar los detalles del algoritmo usado para hashear contraseñas, como por ejemplo el cost del hashing **bcrypt**:

```
# app/config/security.yml
security:
    # ...

    encoders:
        AppBundle\Security\User\WebserviceUser:
            algorithm: bcrypt
            cost: 12
```

### 3\. Múltiples User providers

Cada mecanismo de autenticación (HTTP authentication, formularios de login, etc) utiliza un user provider, y empleará el primer user provider definido por defecto. Pero si por ejemplo queremos especificar a algunos usuarios a través de la configuración y al resto de usuarios en la base de datos, podemos hacerlo creando un nuevo provider que enlaza a los dos juntos:

```
# app/config/security.yml
security:
    providers:
        chain_provider:
            chain:
                providers: [in_memory, user_db]
        in_memory:
            memory:
                users:
                    foo: { password: test }
        user_db:
            entity: { class: AppBundle\Entity\User, property: username }
```

Ahora todos los mecanismos de autenticación emplearán el chain_provider, ya que es el primero que se especifica. Este intentará cargar los usuarios de los providers _in_memory_ y _user_db_.

Puedes también configurar el firewal o mecanismos individuales de autentiación para un provider específico. De nuevo, a no ser que un provider se especifique explícitamente, se emplea el primero:

```
# app/config/security.yml
security:
    firewalls:
        secured_area:
            # ...
            pattern: ^/
            provider: user_db
            http_basic:
                realm: 'Secured Demo Area'
                provider: in_memory
            form_login: ~
```

En este ejemplo, si un usuario trata de acceder a través de autenticación HTTP, el sistema de autenticación usará el provider _in_memory_. Pero si el usuario intenta entrar a través del formulario de login, se empleará el provider _user_db_ (ya que es el por defecto para el firewall).