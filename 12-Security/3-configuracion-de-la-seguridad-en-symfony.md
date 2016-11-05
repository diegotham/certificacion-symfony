El **sistema de seguridad de Symfony** es muy potente, aunque tiene varias secciones que configurar.

1.  Autenticación
2.  Autorización
3.  Devolver el objeto User
4.  Cerrar sesión
5.  Comprobar vulnerabilidades

### 1\. Autenticación

El sistema de seguridad se configura en _app/config/security.yml_. La **configuración por defecto** es como sigue:

```
# app/config/security.yml
security:
    providers:
        in_memory:
            memory: ~

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        default:
            anonymous: ~
```

La _key_ **firewalls** es el corázon de la **configuración de la seguridad**. El firewall **dev** no es importante, simplemente se asegura de que las **herramientas de desarrollo de Symfony** (bajo las **URLs** _/_profiler_ y _/_wdt_) no son bloqueadas por la seguridad.

Todas las demás URLs se administran por el firewall **default** (Si no hay **pattern** _key_ es que afecta a todas las URLs). El firewall es como el sistema de seguridad, por lo que normalmente tiene sentido tener sólo un firewall principal. Esto no significa que todas las URLs requieran autenticación, la key **anonymous** se encarga de esto. Si accedes a la página principal en el entorno de desarrollo, en la toolbar puedes ver que estás autenticado como **anon**, pero esto significa simplemente que eres un usuario anónimo:

![Usuario anónimo Symfony](http://symfony.com/doc/current/_images/security_anonymous_wdt.png)

#### Configurar cómo se autenticarán los usuarios

La principal tarea de un firewall es configurar cómo se autenticarán los usuarios: formulario de login, HTTP basic, API token, todos los anteriores, etc.

Primero empezamos viendo cómo es una autenticación **HTTP basic**, añadiendo la key _http_basic_ en el firewall:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        # ...
        default:
            anonymous: ~
            http_basic: ~
```

Vamos a probar esta configuración, requerimos que el usuario haga **log in** para ver una página _/admin_:

```
// src/AppBundle/Controller/DefaultController.php
// ...

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\Response;

class DefaultController extends Controller
{
    /**
     * @Route("/admin")
     */
    public function adminAction()
    {
        return new Response('<html><body>Admin page!</body></html>');
    }
}
```

Ahora añadimos una entrada _access_control_ en _security.yml_ que exige al usuario que haga log in para acceder a esta URL:

```
# app/config/security.yml
security:
    # ...
    firewalls:
        # ...
        default:
            # ...

    access_control:
        # require ROLE_ADMIN for /admin*
        - { path: ^/admin, roles: ROLE_ADMIN }
```

Si ahora intentamos acceder a _/admin_, veremos el aviso de **autenticación HTTP basic**:

![Autenticación HTTP basic](http://symfony.com/doc/current/_images/security_http_basic_popup.png)

Ahora tendríamos que configurar de dónde vienen los usuarios. Podemos utilizar un [formulario típico de login](http://symfony.com/doc/current/cookbook/security/form_login_setup.html) para identificar a usuarios desde algún sitio, o a través de servicios de terceros como **Google**, **Facebook** o **Twitter**, para lo que podemos emplear el bundle [HWIOAuthBundle](https://github.com/hwi/HWIOAuthBundle).

#### Configurar cómo se cargan los usuarios

Cuando escribes tu nombre de usuario, **Symfony** necesita cargar la información del usuario de alguna parte. Esto es lo que se denomina "_user provider_". Symfony tiene una forma incorporada de [identificar a los usuarios desde una base de datos](http://symfony.com/doc/current/cookbook/security/entity_provider.html), o puedes [crear tu propio user provider](http://symfony.com/doc/current/cookbook/security/custom_provider.html).

La forma más fácil (aunque también la más limitada), es configurar Symfony para cargar los usuarios directamente desde el archivo security.yml. Esto se llama un "_in-memory provider_" o "_in configuration provider_":

```
# app/config/security.yml
security:
    providers:
        in_memory:
            memory:
                users:
                    ryan:
                        password: ryanpass
                        roles: 'ROLE_USER'
                    admin:
                        password: kitten
                        roles: 'ROLE_ADMIN'
    # ...
```

Como con **firewalls**, puedes tener **múltiples providers**, pero lo más seguro que sólo necesites uno. Si tienes múltiples, puedes configurar qué provider utilizar para tu firewall bajo su key **provider** (por ejemplo _provider: in_memory_).

Si ahora intentamos entrar con el nombre de usuario _admin_ y el password _kitten_, veremos un error:

```
No encoder has been configured for account "Symfony\Component\Security\Core\User\User"
```

Para arreglarlo, tenemos que facilitar un encoder, con el key **encoders**:

```
# app/config/security.yml
security:
    # ...

    encoders:
        Symfony\Component\Security\Core\User\User: plaintext
    # ...
```

Los **users providers cargan información del usuario y la ponen en un objeto User**. Si cargas usuarios desde una base de datos o alguna otra fuente, emplearás tu propia clase User. Pero cuando empleas el provider "_in memory_", empleas el objeto _Symfony\Component\Security\Core\User\User_.

Cualquiera que sea tu clase User, necesitas decirle a Symfony qué algoritmo se utilizó para codificar los passwords. En este caso, los passwords son simplemente _plaintext_, pero ahora lo cambiaremos a **bcrypt**.

Si ahora refrescas verás que estás **logeado como admin**:

![Usuario admin logeado en Symfony](http://symfony.com/doc/current/_images/symfony_loggedin_wdt.png)

Ya que esta URL requiere **ROLE_ADMIN**, si te hubieras logeado como **ryan** se te hubiera denegado el acceso. 

#### Codificar el password del usuario

Cuando los usuarios se guardan en security.yml, en una base de datos o en algún otro lado, querrás codificar sus passwords. El mejor algoritmo (actualmente) a usar es **bcrypt**:

```
# app/config/security.yml
security:
    # ...

    encoders:
        Symfony\Component\Security\Core\User\User:
            algorithm: bcrypt
            cost: 12
```

Los **passwords de tus usuarios** ahora han de codificarse con este algoritmo. Existe un comando con el que puedes generar el password codificado:

```
php bin/console security:encode-password
```

Te pedirá qué password quieres codificar. Ahora puedes pegar el password en la configuración:

```
# app/config/security.yml
security:
    # ...

    providers:
        in_memory:
            memory:
                users:
                    ryan:
                        password: $2a$12$LCY0MefVIEc3TYPHV9SNnuzOfyr2p/AXIGoQJEDs4am4JwhNz/jli
                        roles: 'ROLE_USER'
                    admin:
                        password: $2a$12$cyTWeE9kpq1PjqKFiWUZFuCRPwVyAZwm4XzMZ1qPUFl7/flCM3V0G
                        roles: 'ROLE_ADMIN'
```

Ahora todo funcionará como antes.

Una aplicación tiene los **usuarios dinámicos** (por ejemplo desde una base de datos), por lo que tendremos que codificar el password antes de insertarlo en la base de datos. No importa que algoritmo configures para tu objeto User, el password hasheado puede determinarse siempre de la siguiente forma desde un controller:

```
// Obtenemos el objeto User:
$user = new AppBundle\Entity\User();
$plainPassword = 'ryanpass';
$encoder = $this->container->get('security.password_encoder');
$encoded = $encoder->encodePassword($user, $plainPassword);

$user->setPassword($encoded);
```

Para hacer que esto funcione hay que asegurarse que tenemos el **encoder** para la **clase User** (_AppBundle\Entity\User_) bajo el key **encoders** en _app/config/security.yml_.

El objeto _$encoder_ también tiene un método _isPasswordValid_, que toma el objeto User como primer argumento y el **plain password** para comprobar como segundo argumento.

Recuerda que cuando permites a un usuario que envíe un password (en un formulario de registro o cambio de password), debe tener una **validación** que asegure que el password es menor a 4096 caracteres.

#### Autenticación stateless

Por defecto, **Symfony** se basa en una **cookie** (la **Session**) para enviar el security context del usuario. Pero si empleas **certificados** o **autenticación HTTP** por ejemplo, y no necesitas guardar datos entre requests, puedes activar la **autenticación stateless** (lo que evitará que Symfony cree la cookie):

```
# app/config/security.yml
security:
    # ...

    firewalls:
        main:
            http_basic: ~
            stateless:  true
```

En un **formulario de login**, Symfony creará la cookie aunque tu stateless sea true.

### 2\. Autorización

Los users pueden ahora logearse en tu app con _http_basic_ o cualquier otro método. Ahora vamos a ver cómo denegar el acceso y trabajar con el objeto User. esto se llama **autorización**, y su función es decidir si un usuario puede acceder a ciertos resources (una URL, un objeto model, una llamada un método, etc).

El proceso de autorización tiene dos lados:

1.  El usuario recibe un **set específico de roles** cuando inicia sesión (por ejemplo, ROLE_ADMIN).
2.  Añades código de forma que ese _resource_ (**URL**, **controler**, etc) requiera un atributo específico (comúnmente un rol como ROLE_ADMIN) para ser accedido.

Además de los roles (como **ROLE_ADMIN**), se puede proteger un resource utilizando otros atributos o strings (como **EDIT**) y utilizar voters o el **sistema ACL de Symfony**.

#### Roles

Cuando un usuario inicia sesión recibirá un set de roles. En el ejemplo anterior los hemos puesto directamente en security.yml. Si cargas usuarios desde una base de datos, los roles estarán probablemente en una columna de la tabla.

Todos los roles que se asignan a un usuario deben comenzar con el prefijo **ROLE_**, sino no serán administrados por el **sistema de seguridad de Symfony**.

Los roles son simples, y son básicamente _strings_ que creas y utilizas según las necesidades. Por ejemplo si necesitas **limitar el acceso a la sección de administración de un blog**, puedes proteger esa sección con un role **ROLE_BLOG_ADMIN**. Este role no necesita estar definido en ningún sitio, puedes simplemente empezar a usarlo.

Asegúrate de que cada usuario tiene al menos un role, o parecerá como si no está identificado. Una convención común es dar a cada usuario el role ROLE_USER.

Puedes también especificar una **jerarquía de roles**, de forma que se hereden los permisos:

```
# app/config/security.yml
security:
    # ...

    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: [ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]
```

En la configuración anterior, los usuarios con el role **ROLE_ADMIN** también tendrán el role **ROLE_USER**. El role **ROLE_SUPER_ADMIN** tiene el ROLE_ADMIN, **ROLE_ALLOWED_TO_SWITCH** y ROLE_USER (heredado de ROLE_ADMIN).

#### Añadir código para denegar acceso

Existen dos formas de denegar el acceso:

1.  **access_control** en _security.yml_ te permite proteger patrones URL (por ejemplo /admin/*). 

La forma más básica de asegurar partes de la aplicación es asegurando una URL entera. Por ejemplo para entrar a _/admin/*_ necesitas un ADMIN_ROLE:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        # ...
        default:
            # ...

    access_control:
        # require ROLE_ADMIN for /admin*
        - { path: ^/admin, roles: ROLE_ADMIN }
```

Esto funciona bien para **secciones enteras**, pero es probable que también quieras **asegurar controllers**, que lo veremos más adelante.

Puedes definir tantos **patrones URL** como necesites, cada uno es una [expresión regular](http://diego.com.es/expresiones-regulares-en-php), pero sólo coincidirá una. Symfony buscará cada una comenzando desde el principio, y parará hasta que encuetnre una entrada de access control que coincida con la URL:

```
# app/config/security.yml
security:
    # ...

    access_control:
        - { path: ^/admin/users, roles: ROLE_SUPER_ADMIN }
        - { path: ^/admin, roles: ROLE_ADMIN }
```

Al ser una expresión regular, ^ significa que sólo coincidirán las URLs comenzando con el patrón. Por ejemplo un path de sólo _/admin_ coincidiría con _/admin/foo_ pero también con URLs como _/foo/admin_.

La sección de **access_control** es muy potente, pero puede ser peligrosa si no entiendes cómo funciona. Además de una URL, access_control puede coincidir con **direcciones IP**, nombres **host** o [métodos HTTP](http://diego.com.es/metodos-http). Puede también emplearse para redirigir a un usuario a la versión _https_ de un patrón URL.

1.  Con el service _security.authorization_checker_.

Puedes denegar el acceso desde el interior de un controller:

```
// ...

public function helloAction($name)
{
    // El segundo parámetro se utiliza para especificar en qué objeto se testea el role.
    $this->denyAccessUnlessGranted('ROLE_ADMIN', null, 'Unable to access this page!');
```

En ambos casos se lanza una excepción [AccessDeniedException](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Exception/AccessDeniedException.html), que lanza una **código de respuesta HTTP 403**. 

Ahora si el usuario no está logeado, se le pedirá que se logee (por ejemplo redirigiendo a la página de login). Si están logeados pero no tienen el role **ROLE_ADMIN**, se les mostrará una página **403 access denied page** (que puedes personalizar). Si está logeado y tiene el role correcto, el código se ejecutará.

Con el **SensioFrameworkExtraBundle** puedes también asegurar tus controllers con annotations:

```
// ...
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;

/**
 * @Security("has_role('ROLE_ADMIN')")
 */
public function helloAction($name)
{
    // ...
}
```

#### Control de acceso en templates

Si quieres comprobar si el usuario actual tiene un role dentro de una template, utiliza la **función helper** _is_granted()_:

```
{% if is_granted('ROLE_ADMIN') %}
    <a href="...">Delete</a>
{% endif %}
```

#### Asegurar otros services

Cualquier cosa en Symfony puede protegerse haciendo algo similar al código empleado para asegurar el controller. Por ejemplo si tenemos un _service_ (una clase PHP) cuya función es enviar emails, podemos restringir el uso de esta clase a sólo ciertos usuarios.

#### Comprobar si un usuario está logeado (IS_AUTHENTICATED_FULLY)

Si queremos comprobar si un usuario está logeado , podemos emplear la constante **IS_AUTHENTICATED_FULLY**:

```
// ...

public function helloAction($name)
{
    if (!$this->get('security.authorization_checker')->isGranted('IS_AUTHENTICATED_FULLY')) {
        throw $this->createAccessDeniedException();
    }

    // ...
}
```

Puede usarse también en access_control.

IS_AUTHENTICATED_FULLY no es un role, aunque actúa como uno, y cada usuario que está logeado lo tendrá. Hay tres atributos especiales como este:

*   **IS_AUTHENTICATED_REMEMBERED**. Todos los usuarios logeados lo tienen, incluso si están logeados por una cookie "Recordarme". Includo si no tienes la funcionalidad de _recordarme_, puedes emplear esto si quieres comprobar si el usuario está logeado.
*   **IS_AUTHENTICATED_FULLY**. Esto es similar a IS_AUTHENTICATED_REMEMBERED pero más fuerte. Si el usuario está logeado por la cookie remember me, no lo tendrá.
*   **IS_AUTHENTICATED_ANONYMOUSLY**. Todos los usuarios, incluídos los anonymous lo tendrán. Esto es útil para garantizar el acceso a ciertas secciones.

En una template también se pueden emplear:

```
{% if is_granted(expression(
    '"ROLE_ADMIN" in roles or (user and user.isSuperAdmin())'
)) %}
    <a href="...">Delete</a>
{% endif %}
```

#### Access Control Lists (ACLs): Asegurar obtetos individuales de bases de datos

Tenemos un blog donde los usuarios pueden comentar en los posts. Queremos que un usuario pueda editar sus propios comentarios, pero no los de otros usuarios. También, como **admin user**, queremos poder editar todos los comentarios.

Para conseguir esto tenemos dos opciones:

*   **Voters**. Permiten escribir la propia **business logic** (el usuario puede editar este post porque es el creador) para determinar el acceso. Probablemente usaríamos esta opción, es suficientemente flexible para solucionar esta situación.
*   **ACLs**. Permiten crear una **estructura de bases de datos** donde podemos asignar a cualquier usuario cualquier acceso (EDIT, VIEW) a cualquier objeto del sistema. Emplea esto si necesitas que un usuario admin pueda permitir el acceso en el sistema a través de alguna **admin interface**.

En ambos casos igualmente denegaremos el acceso empleando métodos similares a los mostrados antes.

### 3\. Devolver el objeto User

Después de la auntenticación, al **objeto User** del usuario actual se puede acceder a través del service _security.toke_storage_. Desde dentro de un **controller**:

```
public function indexAction()
{
    if (!$this->get('security.authorization_checker')->isGranted('IS_AUTHENTICATED_FULLY')) {
        throw $this->createAccessDeniedException();
    }

    $user = $this->getUser();

    // the above is a shortcut for this
    $user = $this->get('security.token_storage')->getToken()->getUser();
}
```

El usuario será un objeto y la clase de ese objeto dependerá de tu **user provider**.

Ahora puedes llamar a los métodos que quieras de tu objeto User. Por ejemplo, si tu objeto User tiene un método _getFirstName()_, puedes emplearlo:

```
use Symfony\Component\HttpFoundation\Response;
// ...

public function indexAction()
{
    // ...

    return new Response('Well hi there '.$user->getFirstName());
}
```

#### Siempre comprueba si el usuario está logeado

Es importante **comprobar si el usuario está autenticado primero**. Si no lo está, _$user_ será _null_. En un principio devuelve el string _anon_, pero el shortcut getUser() del controller lo convierte en _null_ por conveniencia.

La cosa es que siempre hay que comprobar si el usuario está logeado antes de usar el **objeto User**, y usar el método _isGranted_ (o _access_control_) para hacerlo:

```
// Usa esto para ver si el usuario está logeado
if (!$this->get('security.authorization_checker')->isGranted('IS_AUTHENTICATED_FULLY')) {
    throw $this->createAccessDeniedException();
}

// Mal. Nunca comprueba el objeto User para ver si está logeado
if ($this->getUser()) {

}
```

#### Devolver al User en una template

En una template twig se puede acceder a este objeto con la key _app.user_:

```
{% if is_granted('IS_AUTHENTICATED_FULLY') %}
    <p>Username: {{ app.user.username }}</p>
{% endif %}
```

### 4\. Cerrar sesión

Lo normal es hacer que los usuarios puedan también **cerrar sesión**. El firewall puede hacer esto automáticamente cuando activas el parámetro de configuración _logout_:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        secured_area:
            # ...
            logout:
                path:   /logout
                target: /
```

Ahora creamos una route para esta URL (pero no un controller):

```
# app/config/routing.yml
logout:
    path: /logout
```

Eso es todo. Si ahora enviamos al usuario a _/logout_ (o el _path_ que se haya configurado), Symfony desautenticará al usuario actual.

Una vez que el usuario ha salido, será redirigido a cualquier _path_ definido en el parámetro _target_ (por ejemplo _homepage_).

Si quieres hacer algo más interesante después de cerrar sesión, puedes especificar un _logout success handler_ añadiendo el key _success_handler_ y apuntándolo a un _service id_ de una clase que implemente [LogoutSuccessHandlerInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Http/Logout/LogoutSuccessHandlerInterface.html).

### 5\. Comprobar vulnerabilidades

Cuando se usan muchas dependencias alguna puede contener alguna **vulnerabilidad**. Eso es por lo que Symfony incluye un comando llamado _security:check_ que comprueba el archivo _composer.lock_ para **encontrar alguna vulnerabilidad en las dependencias instaladas**:

```
php bin/console security:check
```

Una buena práctica de seguridad es ejecutar este comando regularmente para poder actualizar o reemplazar dependencias comprometidas. Internamente, el comando utiliza la [security advisories database](https://github.com/FriendsOfPHP/security-advisories) publicado por la organización **FriendsOfPHP**.

El comando _security:check_ termina con una salida no equivalente a cero si cualquier dependencia tiene alguna vulnerabilidad, por lo que puedes integrarlo en algún proceso.

Para el comando _security:check_ es necesario tener instalado el [SensioDistributionBundle](https://packagist.org/packages/sensio/distribution-bundle).