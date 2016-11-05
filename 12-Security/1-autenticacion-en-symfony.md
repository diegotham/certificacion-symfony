Cuando un request apunta a un área asegurada y uno de los **listeners** del mapa del firewall puede extraer las credenciales del usuario del objeto **Request** actual, creará un **token** conteniendo estas credenciales. Lo siguiente que debe hacer el listener es pedir al **authentication manager** validar el token dado, y devolver un **authenticated token** si las credenciales recibidas son válidas. El listener guardará entonces el authenticated token empleando el [token storage](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Token/Storage/TokenStorageInterface.html):

```
use Symfony\Component\Security\Http\Firewall\ListenerInterface;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;
use Symfony\Component\Security\Core\Authentication\AuthenticationManagerInterface;
use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\Security\Core\Authentication\Token\UsernamePasswordToken;

class SomeAuthenticationListener implements ListenerInterface
{
    /**
     * @var TokenStorageInterface
     */
    private $tokenStorage;

    /**
     * @var AuthenticationManagerInterface
     */
    private $authenticationManager;

    /**
     * @var string Uniquely identifies the secured area
     */
    private $providerKey;

    // ...

    public function handle(GetResponseEvent $event)
    {
        $request = $event->getRequest();

        $username = ...;
        $password = ...;

        $unauthenticatedToken = new UsernamePasswordToken(
            $username,
            $password,
            $this->providerKey
        );

        $authenticatedToken = $this
            ->authenticationManager
            ->authenticate($unauthenticatedToken);

        $this->tokenStorage->setToken($authenticatedToken);
    }
}
```

Un token puede ser cualquier clase, siempre que implemente **TokenInterface**.

**Indice de contenido**

1.  El Authentication Manager
2.  Authentication Providers
3.  Authentication Events

### 1. El Authentication Manager

El **authentication manager** por defecto es una instancia de **AuthenticationProviderManager**:

```
use Symfony\Component\Security\Core\Authentication\AuthenticationProviderManager;

// instancias de Symfony\Component\Security\Core\Authentication\Provider\AuthenticationProviderInterface
$providers = array(...);

$authenticationManager = new AuthenticationProviderManager($providers);

try {
    $authenticatedToken = $authenticationManager
        ->authenticate($unauthenticatedToken);
} catch (AuthenticationException $failed) {
    // ha fallado la autenticación
}
```

El **AuthenticationProviderManager**, cuando se instancia, recibe varios authentication providers, cada uno soportando un tipo diferente de token.

Puedes crear también tu propio authentication manager, simplemente tiene que implementar [AuthenticationManagerInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/AuthenticationManagerInterface.html).

### 2\. Authentication Providers

Cada provider (ya que implementa [AuthenticationProviderInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Provider/AuthenticationProviderInterface.html)) tiene un método [_supports()_](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Provider/AuthenticationProviderInterface.html#method_supports) mediante el que **AuthenticationProviderManager** puede determinar si soporta el token dado. Si este es el caso, el manager llama entonces al método del provider [_authenticate()_](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Provider/AuthenticationProviderInterface.html#method_authenticate). Este método debe devolver un token autenticado o devolver una [AuthenticationException](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Exception/AuthenticationException.html) (o cualquier otra excepción que la extienda).

#### Autenticar usuarios por su nombre de usuario y password

Un **authentication provider** intentará autenticar a un usuario según las credenciales que haya enviado. Normalmente éstas son usuario y password. La mayoría de aplicaciones guardan el **nombre de usuario** y el **hash del password** del usuario combinados con un _salt_ aleatorio. Esto significa que una autenticación típica consistiría en obtener el salt y el password hasheado de la base de datos del usuario, hashear el password que el usuario ha introducido (mediante un **formulario de login**) con el salt y comparar ambos para comprobar si el password enviado es válido.

La funcionalidad la ofrece el [DaoAuthenticationProvider](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Provider/DaoAuthenticationProvider.html). Obtiene los datos del usuario de una [UserProviderInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/User/UserProviderInterface.html), utiliza una [PasswordEncoderInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Encoder/PasswordEncoderInterface.html) para crear un hash del password y devuelve un token autenticado si el password es válido:

```
use Symfony\Component\Security\Core\Authentication\Provider\DaoAuthenticationProvider;
use Symfony\Component\Security\Core\User\UserChecker;
use Symfony\Component\Security\Core\User\InMemoryUserProvider;
use Symfony\Component\Security\Core\Encoder\EncoderFactory;

$userProvider = new InMemoryUserProvider(
    array(
        'admin' => array(
            // El password es "foo"
            'password' => '5FZ2Z8QIkA7UTZ4BYkoC+GsReLf569mSKDsfods6LYQ8t+a8EW9oaircfMpmaLbPBh4FOBiiFyLfuZmTSUwzZg==',
            'roles'    => array('ROLE_ADMIN'),
        ),
    )
);

// Para algunas comprobaciones extra: está la cuenta activada, bloqueada, expirada, etc.?
$userChecker = new UserChecker();

// Array de password encoders
$encoderFactory = new EncoderFactory(...);

$provider = new DaoAuthenticationProvider(
    $userProvider,
    $userChecker,
    'secured_area',
    $encoderFactory
);

$provider->authenticate($unauthenticatedToken);
```

El ejemplo anterior muestra el uso de un user provider "_in-memory_", pero puedes emplear otro user provider, siempre que implemente [UserProviderInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/User/UserProviderInterface.html). También es posible emplear múltiples user providers para encontrar los datos del usuario con [ChainUserProvider](http://api.symfony.com/3.0/Symfony/Component/Security/Core/User/ChainUserProvider.html).

#### Password Encoder Factory

El **DaoAuthenticationProvider** utiliza un **encoder factory** para crear un **password encoder** para un tipo de usuario dado. Esto te permite utilizar diferentes estrategias de encoding para diferentes tipos de usuarios. El **EncoderFactory** por defecto recibe un array de encoders:

```
use Symfony\Component\Security\Core\Encoder\EncoderFactory;
use Symfony\Component\Security\Core\Encoder\MessageDigestPasswordEncoder;

$defaultEncoder = new MessageDigestPasswordEncoder('sha512', true, 5000);
$weakEncoder = new MessageDigestPasswordEncoder('md5', true, 1);

$encoders = array(
    'Symfony\\Component\\Security\\Core\\User\\User' => $defaultEncoder,
    'Acme\\Entity\\LegacyUser'                       => $weakEncoder,

    // ...
);

$encoderFactory = new EncoderFactory($encoders);
```

Cada encoder ha de implementar **PasswordEncoderInterface** o ser un array con las keys _class_ y _arguments_, lo que le permite al encoder factory construir el encoder sólo cuando se le necesite.

Crear un Password Encoder personalizado

Hay muchos encoders ya incorporados, pero si necesitar crear el tuyo propio, simplemente necesita seguir las siguientes reglas:

1.  La clase debe implementar **PasswordEncoderInterface**.
2.  Las implementaciones de _encodePassword()_ y _isPasswordValid()_ deben asegurar que el password no sea demasiado largo por razones de seguridad. Puedes emplear el método _isPasswordTooLong()_ para comprobarlo:

```
use Symfony\Component\Security\Core\Encoder\BasePasswordEncoder;
use Symfony\Component\Security\Core\Exception\BadCredentialsException;

class FoobarEncoder extends BasePasswordEncoder
{
    public function encodePassword($raw, $salt)
    {
        if ($this->isPasswordTooLong($raw)) {
            throw new BadCredentialsException('Password inválido.');
        }

        // ...
    }

    public function isPasswordValid($encoded, $raw, $salt)
    {
        if ($this->isPasswordTooLong($raw)) {
            return false;
        }

        // ...
}
```

Utilizar Password Encoders

Cuando el método _getEncoder()_ del **password encoder factory** es llamado con el objeto user como primer argumento, devolverá un encoder del tipo **PasswordEncoderInterface** que debería usarse para codificar el password de este usuario:

```
// una instancia de Acme\Entity\LegacyUser
$user = ...;

// el password enviado, por ejemplo al registrarse registering
$plainPassword = ...;

$encoder = $encoderFactory->getEncoder($user);

// devolverá $weakEncoder
$encodedPassword = $encoder->encodePassword($plainPassword, $user->getSalt());

$user->setPassword($encodedPassword);

// ... guardar el usuario
```

Ahora cuando quieras comprobar si el password enviado es correcto, puedes utilizar:

```
// obtener el Acme\Entity\LegacyUser
$user = ...;

// El password enviado, por ejemplo del formulario de login
$plainPassword = ...;

$validPassword = $encoder->isPasswordValid(
    $user->getPassword(), // El password codificado
    $plainPassword,       // El password enviado
    $user->getSalt()
);
```

### 3\. Authentication Events

El componente Security proporciona 4 eventos relacionados con la autenticación:

| | | |
| -------- | -------- |
| Nombre | Constante Event | Argumento pasado al listener |
| security.authentication.success | AuthenticationEvents::AUTHENTICATION_SUCCESS | AuthenticationEvent |
| security.authentication.failure | AuthenticationEvents::AUTHENTICATION_FAILURE | AuthenticationFailureEvent |
| security.interactive_login | SecurityEvents::INTERACTIVE_LOGIN | InteractiveLoginEvent |
| security.switch_user | SecurityEvents::SWITCH_USER | SwitchUserEvent |

#### Eventos de éxito y fracaso de autenticación

Cuando un provider autentica al usuario, un evento _security.authentication.success_ es lanzado. Pero este evento se disparará, por ejemplo, en cada request si tienes **autenticación session-based**. Si quieres hacer algo cuando el usuario entre simplemente, emplea _security.interactive_login_. 

Cuando un usuario intenta autenticarse pero falla (se lanza una **AuthenticationException**), se lanza un evento _security.authentication.failure_. Puedes atender a este evento, por ejemplo, para logear intentos fállidos de entrada al sistema.

#### Eventos de seguridad

El evento _security.interactive_login_ se lanza después de que el usuario se haya logeado **activamente** en el sitio. Es importante distinguir esta acción de los **métodos de autenticación no interactivos**, como:

*   Autenticación basada en una coockie "Recordarme".
*   Autenticación basada en tu session.
*   Autenticación con los headers HTTP basic o HTTP digest.

Puedes atender al evento security.interactive_login, por ejemplo, si quieres dar una bienvenida al usuario con un mensaje flash cada vez que entra.

El evento _security.switch_user_ se lanza cada vez que activas el _switch_user_ firewall listener.