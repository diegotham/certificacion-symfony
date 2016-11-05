Los **password encoders** permiten establecer el algoritmo con el que encriptar las contraseñas de los usuarios de una aplicación.

**Indice de contenido**

1.  Crear un formulario personalizado de autenticación
2.  Elegir un password encoder dinámicamente

### 1\. Crear un formulario personalizado de autenticación

Vamos a crear un **password authenticator** hacer que sólo sea posible acceder a un sitio web entre las 14 y las 16 horas UTC. Podemos hacer esto desde un formulario de login.

Primero creamos una nueva clase que implemente **SimpleFormAuthenticatorInterface**. Esto permitirá crear lógica personalizada para autenticar al usuario:

```
// src/Acme/HelloBundle/Security/TimeAuthenticator.php
namespace Acme\HelloBundle\Security;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authentication\Token\UsernamePasswordToken;
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;
use Symfony\Component\Security\Core\Exception\CustomUserMessageAuthenticationException;
use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Http\Authentication\SimpleFormAuthenticatorInterface;

class TimeAuthenticator implements SimpleFormAuthenticatorInterface
{
    private $encoder;

    public function __construct(UserPasswordEncoderInterface $encoder)
    {
        $this->encoder = $encoder;
    }

    public function authenticateToken(TokenInterface $token, UserProviderInterface $userProvider, $providerKey)
    {
        try {
            $user = $userProvider->loadUserByUsername($token->getUsername());
        } catch (UsernameNotFoundException $e) {
            // CUIDADO: este mensaje se devolverá al cliente
            // (no muestres mensajes de error aquí)
            throw new CustomUserMessageAuthenticationException('Usuario o contraseña inválidos');
        }

        $passwordValid = $this->encoder->isPasswordValid($user, $token->getCredentials());

        if ($passwordValid) {
            $currentHour = date('G');
            if ($currentHour < 14 || $currentHour > 16) {
                // CUIDADO: este mensaje se devolverá al cliente
                // (no muestres mensajes de error aquí)
                throw new CustomUserMessageAuthenticationException(
                    'Sólo puedes entrar de 14:00 a 16:00!',
                    100
                );
            }

            return new UsernamePasswordToken(
                $user,
                $user->getPassword(),
                $providerKey,
                $user->getRoles()
            );
        }

        // CUIDADO: este mensaje se devolverá al cliente
        // (no muestres mensajes de error aquí)
        throw new CustomUserMessageAuthenticationException('Usuario o contraseña inválidos');
    }

    public function supportsToken(TokenInterface $token, $providerKey)
    {
        return $token instanceof UsernamePasswordToken
            && $token->getProviderKey() === $providerKey;
    }

    public function createToken(Request $request, $username, $password, $providerKey)
    {
        return new UsernamePasswordToken($username, $password, $providerKey);
    }
}
```

Ahora ya podemos establecer la configuración, pero vamos a ver qué hace cada uno de los métodos anteriores:

*   _**createToken**_. Cuando Symfony comienza a analizar el request, llama a _createToken()_, el cual crea un objeto **TokenInterface** que contiene cualquier información que necesite _authenticateToken()_ para autenticar al usuario (por ejemplo nombre de usuario y contraseña).
*   _**supportsToken**_. Una vez que Symfony llama a _createToken()_, llamará a _supportsToken()_ en la clase (y a cualquier otro listener de autenticación) para averiguar quién manejará el token. Esta es una forma de permitir varios mecanismos de autenticación en el mismo firewall (de esta forma se puede primero autenticar al usuario con un API key y sino volver al formulario de login). 
    Simplemente has de cercionarte de que este método devuelve **true** para un token creado por _createToken()_. 
*   _**authenticateToken**_. Si _supportsToken_ devuelve true, **Symfony** llamará ahora a _authenticateToken()_. La tarea ahora sería comprobar que el token puede logearse obteniendo primero el objeto User a través del **user provider** y después, comprobando la contraseña y la fecha actual.
    El flujo de cómo obtienes el objeto User y determinas si el token es válido (por ejemplo comprobando el password), puede variar en base a tus necesidades.

El fin último es devolver un nuevo objeto **token autenticado** (tiene al menos un **role** establecido) y el cual tiene el objeto user en su interior.

En este método se necesita el **password encoder** para comprobar la validez del password:

```
$passwordValid = $this->encoder->isPasswordValid($user, $token->getCredentials());
```

Es un _service_ disponible en **Symfony** que emplea el algoritmo de passwords configurado en el archivo de configuración de seguridad (_security.yml_) bajo la **key** encoders. Para inyectarlo en **TimeAuthenticator**, primero tenemos que configurar éste como un service:

```
# app/config/config.yml
services:
    # ...

    time_authenticator:
        class:     Acme\HelloBundle\Security\TimeAuthenticator
        arguments: ["@security.password_encoder"]
```

Entonces activarlo en la sección **firewalls** de la configuración de seguridad mediante el key _simple_form_:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        secured_area:
            pattern: ^/admin
            # ...
            simple_form:
                authenticator: time_authenticator
                check_path:    login_check
                login_path:    login
```

La key _simple_form_ tiene las mismas opciones que la opción normal _form_login_, pero con el key authenticator adicional que apunta al nuevo service. 

### 2\. Elegir un password encoder dinámicamente

Normalmente el mismo **password encoder** se emplea para todos los usuarios configurándolo para todas las instancias de una clase específica:

```
# app/config/security.yml
security:
    # ...
    encoders:
        Symfony\Component\Security\Core\User\User: sha512
```

Otra opción es emplear un **encoder nombrado** y seleccionar qué encoder quieres emplear, de forma dinámica.

En el ejemplo anterior, has establecido al algoritmo **sha512** para _Acme\UserBundle\Entity\User_. Este puede ser suficientemente seguro para un usuario regular, pero si quieres que tus administradores tengan un algoritmo más fuerte, tendrías que aplicar uno como **bcrypt**. Esto puede hacerse con encoders nombrados:

```
# app/config/security.yml
security:
    # ...
    encoders:
        harsh:
            algorithm: bcrypt
            cost: 15
```

Esto crea un encoder llamado **harsh**. Para que una instancia de **User** lo emplee, la clase debe implementar **EncoderAwareInterface**. La interface requiere un método, _getEncoderName_, que debería devolver el nombre del encoder a usar:

```
// src/Acme/UserBundle/Entity/User.php
namespace Acme\UserBundle\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\Encoder\EncoderAwareInterface;

class User implements UserInterface, EncoderAwareInterface
{
    public function getEncoderName()
    {
        if ($this->isAdmin()) {
            return 'harsh';
        }

        return null; // use the default encoder
    }
}
```