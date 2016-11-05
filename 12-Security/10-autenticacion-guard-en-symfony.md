Ya sea que tengas que crear un **formulario de login** tradicional, un sistema de **autenticación API token** u otros sistemas de autenticación, el **componente Guard** puede hacer que sea más fácil.

**Indice de contenido**

1.  Crear una clase User y un User Provider
2.  Crear una clase Authenticator
3.  Configurar el Authenticator
4.  Métodos del Guard Authenticator
5.  Personalizar mensajes de error
6.  Casos con Guard Authenticator

### 1\. Crear una clase User y un User Provider

No importa la forma de autenticación, tenemos que crear una clase **User** que implemente **UserInterface** y configurar un user provider. En este ejemplo los usuarios se guardan en la base de datos con **Doctrine**, y cada usuario tiene una propiedad **apiKey** que emplean para acceder a sus cuentas a través de la API:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="user")
 */
class User implements UserInterface
{
    /**
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string", unique=true)
     */
    private $username;

    /**
     * @ORM\Column(type="string", unique=true)
     */
    private $apiKey;

    public function getUsername()
    {
        return $this->username;
    }

    public function getRoles()
    {
        return ['ROLE_USER'];
    }

    public function getPassword()
    {
    }
    public function getSalt()
    {
    }
    public function eraseCredentials()
    {
    }

    // más getters y setters
}
```

La clase User anterior no tiene un **password**, pero puedes añadirle una propiedad password si quieres también permitir que el usuario se logee a través de un formulario de login. Tampoco tiene por qué guardarse con Doctrine, puedes emplear las herramientas que prefieras.

Después configuramos el **user provider** para el usuario:

```
# app/config/security.yml
security:
    # ...

    providers:
        your_db_provider:
            entity:
                class: AppBundle:User

    # ...
```

### 2\. Crear una clase authenticator

Tenemos una API donde los clientes enviarán un header **X-AUTH-TOKEN** en cada request con su **API token**. Nuestra tarea es encontrar al usuario del respectivo token (si es que existe).

Para crear un **sistema de autenticación** simplemente creamos una clase y hacemos que implemente [GuardAuthenticatorInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Guard/GuardAuthenticatorInterface.html) o extendemos a [AbstractGuardAuthenticator](http://api.symfony.com/3.0/Symfony/Component/Security/Guard/AbstractGuardAuthenticator.html). Esto requiere que implementemos seis métodos:

```
// src/AppBundle/Security/TokenAuthenticator.php
namespace AppBundle\Security;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Guard\AbstractGuardAuthenticator;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Doctrine\ORM\EntityManager;

class TokenAuthenticator extends AbstractGuardAuthenticator
{
    private $em;

    public function __construct(EntityManager $em)
    {
        $this->em = $em;
    }

    /**
     * Llamado en cada request. Devolverá las credenciales que quieras,
     * o null para parar la autenticación.
     */
    public function getCredentials(Request $request)
    {
        if (!$token = $request->headers->get('X-AUTH-TOKEN')) {
            // no hay token? Devuelve null y no se llamará a otros métodos
            return;
        }

        // Lo que se devuelva aquí se pasará a getUser() como $credentials
        return array(
            'token' => $token,
        );
    }

    public function getUser($credentials, UserProviderInterface $userProvider)
    {
        $apiKey = $credentials['token'];

        // si es null, la autenticación fallará
        // si es un objeto User se llama a checkCredentials()
        return $this->em->getRepository('AppBundle:User')
            ->findOneBy(array('apiKey' => $apiKey));
    }

    public function checkCredentials($credentials, UserInterface $user)
    {
        // comprueba las credenciales - e.g. asegurar que el password es válido
        // no se necesita comprobación de credenciales en este caso

        // devuelve true con autenticación exitosa
        return true;
    }

    public function onAuthenticationSuccess(Request $request, TokenInterface $token, $providerKey)
    {
        // si es exitoso, dejar que el request continúe
        return null;
    }

    public function onAuthenticationFailure(Request $request, AuthenticationException $exception)
    {
        $data = array(
            'message' => strtr($exception->getMessageKey(), $exception->getMessageData())

            // para traducir este mensaje:
            // $this->translator->trans($exception->getMessageKey(), $exception->getMessageData())
        );

        return new JsonResponse($data, 403);
    }

    /**
     * Llamado cuando se necesita autenticación pero no se ha enviado
     */
    public function start(Request $request, AuthenticationException $authException = null)
    {
        $data = array(
            // puedes traducir este mensaje también
            'message' => 'Authentication Required'
        );

        return new JsonResponse($data, 401);
    }

    public function supportsRememberMe()
    {
        return false;
    }
}
```

Cada uno de los métodos se explican en el apartado 4.

### 3\. Configurar el authenticator

Para finalizar esto registramos la clase como service:

```
# app/config/services.yml
services:
    app.token_authenticator:
        class: AppBundle\Security\TokenAuthenticator
        arguments: ['@doctrine.orm.entity_manager']
```

Finalmente configuramos la key _firewalls_ en el archivo de configuración _security.yml_ para usar este **authenticator**:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        # ...

        main:
            anonymous: ~
            logout: ~

            guard:
                authenticators:
                    - app.token_authenticator

            # si quieres, desactiva guardar al usuario en la sesión
            # stateless: true

            # quizás otras cosas, como form_login, remember_me, etc
            # ...
```

Ya está construído el sistema de autenticación API token. Si tu página de inicio necesitase ROLE_USER, podrías testearlo en diferentes situaciones:

```
# test sin token
curl http://localhost:8000/
# {"message":"Authentication Required"}

# test con un token inválido
curl -H "X-AUTH-TOKEN: FAKE" http://localhost:8000/
# {"message":"Username could not be found."}

# test con un token funcional
curl -H "X-AUTH-TOKEN: REAL" http://localhost:8000/
# se ejecuta el controller de la página principal: la página carga correctamente
```

### 4\. Métodos del Guard Authenticator

Cada authenticator necesita los siguientes métodos:

*   **getCredentials (Request $request)**. Se llama a este método en cada request y nuestra tarea es leer el **token** (o la información de autenticación que sea) del request y devolverlo. Si devuelve null, el resto del proceso de autenticación se para. Sino, se llama a _getUser()_ y el valor devuelto se pasa como primer argumento.
*   **getUser ($credentials, UserProviderInterface $userProvider)**. Si _getCredentials()_ devuelve un valor no null, se llama a este método y su valor devuelto se pasa aquí como el argumento $credentials. La tarea aquí es devolver un objeto que implemente **UserInterface**, y entonces se llama a _checkCredentials()_. Si devuelves null (o lanzas una **AuthenticationException**) la autenticación fallará.
*   **checkCredentials ($credentials, UserInterface $user)**. Si _getUser()_ devuelve un objeto User, se llama a este método. Tu tarea es verificar si las credenciales son correctas. Para un formulario login, aquí es donde compruebas que el password es correcto para el usuario. Para pasar la autenticación, devuelve **true**. Si devuelves cualquier otra cosa (o lanzas una [AuthenticationException](http://symfony.com/doc/current/cookbook/security/guard-authentication.html#guard-customize-error)), la autenticación fallará.
*   **onAuthenticationSuccess (Request $request, TokenInterface $token, $providerKey)**. Se llama después de una autenticación exitosa y la tarea ahora es devolver un objeto **Response** que se enviará al cliente o null para continuar el **request** (como permitir llamar al router/controller de forma normal). Ya que esto este ejemplo es un API donde cada request se autentifica, devolvemos null. 
*   **onAuthenticationFailure (Request $request, AuthenticationException $exception)**. Se llama a este método si la autenticación falla. La tarea es devolver el objeto **Response** que debería enviarse al cliente. **$exception** te dirá que fue mal durante la autenticación.
*   **start (Request $request, AuthenticationException $authException = null)**. Se llama si el cliente accede a una **URI** o **resource** que requiere autenticación, pero no se han enviado detalles de autenticación (por ejemplo _getCredentials()_ devuelve null). La tarea ahora es devolver un objeto **Response** que ayuda al usuario a autenticarse (por ejemplo una respuesta 401 con el mensaje "falta el token!").
*   **supportsRememberMe**. Si quieres tener una funcionalidad de "recordarme", devuelve true con este método. Igualmente necesitarás activar el key _remember_me_ en el **firewall**. Ya que esta es una API stateless, no queremos emplear "recordarme".

### 5\. Personalizar mensajes de error

Cuando se llama a _onAuthenticationFailure()_, se pasa una **AuthenticationException** que describe cómo falló la autenticación a través de sus métodos $e->getMessageKey() o _$e->getMessageData()_. El mensaje será diferente basándose en dónde falla la autenticación (por ejemplo _getUser()_ versus _checkCredentials()_).

Pero puedes devolver un error personalizado lanzando una [CustomUserMessageAuthenticationException](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Exception/CustomUserMessageAuthenticationException.html). Puedes lanzarla desde _getCredentials()_, _getUser()_ o _checkCredentials()_:

```
// src/AppBundle/Security/TokenAuthenticator.php
// ...

use Symfony\Component\Security\Core\Exception\CustomUserMessageAuthenticationException;

class TokenAuthenticator extends AbstractGuardAuthenticator
{
    // ...

    public function getCredentials(Request $request)
    {
        // ...

        if ($token == 'ILuvAPIs') {
            throw new CustomUserMessageAuthenticationException(
                'ILuvAPIs no es una API key real: es una frase sin más'
            );
        }

        // ...
    }

    // ...
}
```

### 6\. Casos con Guard Authenticator

#### Múltiples Authenticators

Se pueden utilizar múltiples **authenticators** pero se ha de elegir sólo un authenticator como _entry_point_. Esto significa que se ha de elegir el método _start()_ del authenticator al que llamar cuando un usuario anónimo intenta acceder a un resource protegido. Si tenemos un _app.form_login_authenticator_ que maneja un formulario de login, cuando un usuario accede a una página protegida de forma anónima, queremos emplear el método _start()_ del form authenticator y redireccionarlo a la página de login (en lugar de devolver una respuesta **JSON**).

#### Emplear con form_login

_form_login_ es una forma de autenticar al usuario, por lo que puedes usarlo y añadir uno o más authenticators. Emplear un guard authenticator no colisiona con otras formas de autenticación.

#### FOSUserBundle

Este bundle realmente no maneja la seguridad, simplemente devuelve un objeto **User** y algunas **routes** y **controllers** para ayudar con el login, registro, recordar contraseña, etc. Cuando utilizas **FOSUserBundle** normalmente empleas _form_login_ para autenticar al usuario. Puedes continuar haciéndolo o utilizar el objeto User de FOSUserBundle y crear tus propios authenticators.