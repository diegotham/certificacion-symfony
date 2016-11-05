Cuando cualquiera de los **authentication providers** haya verificado el todavía no autenticado token, se devolverá un token autenticado. El authentication listener debe establecer este token directamente en el [TokenStorageInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authentication/Token/Storage/TokenStorageInterface.html) utilizando el método _setToken()_. 

Ahora el usuario ya está autenticado (identificado). Ahora, otras partes de la aplicación pueden emplear el token para decidir si el usuario puede solicitar una **URI** concreta o **modificar un objeto**. Esta decisión se hace con una instancia de [AccessDecisionManagerInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authorization/AccessDecisionManagerInterface.html).

Una **authorization decision** siempre se basará en varias cosas:

*   El **token actual**. Por ejemplo el método _getRoles()_ del token puede usarse para devolver los roles del usuario actual (por ejemplo **ROLE_SUPER_ADMIN**), o una decisión puede basarse en la clase del token.
*   Un **conjunto de atributos**. Cada atributo representa un derecho que el usuario debería tener, por ejemplo **ROLE_ADMIN** asegura que el usuario es un administrador.
*   Un **objeto** (opcional). Cualquier objeto que haya de comprobarse con **control de acceso**, como un objeto **article** o **comment**. 

### Access Decision Manager

Decidir si un usuario está autorizado o no para realizar cierta acción puede ser un proceso complicado, por eso el **AccessDecisionManager** depende de múltiples voters, y hace un veredicto final asado en todos los votos (positivo, negativo o neutral) que haya recibido. Reconoce varias estrategias:

*   **affirmative** (por defecto). Permite el acceso tan pronto como haya un voter que garantice el acceso.
*   **consensus**. Permite el acceso si hay más voters garantizando el acceso de los que lo deniegan.
*   **unanimous**. Sólo permite el acceso si nonguno de los voters lo ha denegado.

```
use Symfony\Component\Security\Core\Authorization\AccessDecisionManager;

// instancias de Symfony\Component\Security\Core\Authorization\Voter\VoterInterface
$voters = array(...);

// una de "affirmative", "consensus", "unanimous"
$strategy = ...;

// si permitir acceso o no cuando todos los voters se abstienen
$allowIfAllAbstainDecisions = ...;

// si permitir el acceso cuando no hay mayoría (se aplica sólo a la estrategia "consensus")
$allowIfEqualGrantedDeniedDecisions = ...;

$accessDecisionManager = new AccessDecisionManager(
    $voters,
    $strategy,
    $allowIfAllAbstainDecisions,
    $allowIfEqualGrantedDeniedDecisions
);
```

Puedes cambiar la estrategia en la configuración.

### Voters

Los **voters** son instancias de [VoterInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authorization/Voter/VoterInterface.html), lo que significa que tienen que implementar unos pocos métodos para que pueda usarlos el **decision manager**:

```
int vote (TokenInterface $token, mixed $subject, array $attributes)
```

Este método se encargará del voto y devolverá un valor igual a una de las constantes de clase **VoterInterface**: VoterInterface::ACCESS_GRANTED, VoterInterface::ACCESS_DENIED o VoterInterface::ACCESS_ABSTAIN.

El **componente de seguridad** contiene algunos voters por defecto que cubren la mayoría de usos:

#### AuthenticatedVoter

El voter **AuthenticatedVoter** soporta los atributos **IS_AUTHENTICATED_FULLY**, **IS_AUTHENTICATED_REMEMBERED** y **IS_AUTHENTICATED_ANONYMOUSLY** y permite el acceso basándose en el **nivel actual de autenticación**, si el usuario está totalmente autenticado, o basado en una cookie "Recordarme", o autenticado de forma anónima, respectivamente.

```
use Symfony\Component\Security\Core\Authentication\AuthenticationTrustResolver;

$anonymousClass = 'Symfony\Component\Security\Core\Authentication\Token\AnonymousToken';
$rememberMeClass = 'Symfony\Component\Security\Core\Authentication\Token\RememberMeToken';

$trustResolver = new AuthenticationTrustResolver($anonymousClass, $rememberMeClass);

$authenticatedVoter = new AuthenticatedVoter($trustResolver);

// instancia de Symfony\Component\Security\Core\Authentication\Token\TokenInterface
$token = ...;

// cualquier objeto
$object = ...;

$vote = $authenticatedVoter->vote($token, $object, array('IS_AUTHENTICATED_FULLY');
```

#### RoleVoter

El RoleVoter soporta atributos que comienzan por **ROLE_** y garantiza el acceso al usuario cuando los atributos requeridos **ROLE_*** pueden encontrarse en el array de roles devueltos por el método _getRoles()_ del token:

```
use Symfony\Component\Security\Core\Authorization\Voter\RoleVoter;

$roleVoter = new RoleVoter('ROLE_');

$roleVoter->vote($token, $object, array('ROLE_ADMIN'));
```

#### RoleHierarchyVoter

El **RoleHierarchyVoter** extiende **RoleVoter** y proporciona alguna funcionalidad adicional: sabe como manejar una jerarquía de roles. Por ejemplo, un role ROLE_SUPER_ADMIN puede tener subroles ROLE_ADMIN y ROLE_USER, de forma que cuando un objeto requiera que el usuario tenga el role ROLE_ADMIN, permita el acceso a usuarios que tengan dicho role, pero también a usuarios que tengan el role ROLE_SUPER_ADMIN:

```
use Symfony\Component\Security\Core\Authorization\Voter\RoleHierarchyVoter;
use Symfony\Component\Security\Core\Role\RoleHierarchy;

$hierarchy = array(
    'ROLE_SUPER_ADMIN' => array('ROLE_ADMIN', 'ROLE_USER'),
);

$roleHierarchy = new RoleHierarchy($hierarchy);

$roleHierarchyVoter = new RoleHierarchyVoter($roleHierarchy);
```

### Roles

Los roles son objetos que dan sentido a ciertos derechos que tenga el usuario. El único requisito es que implementen **RoleInterface**, lo que significa que deberían también tener el método _getRole()_ que devuelve una representación en string del role. El **Role** por defecto simplemente devuelve su primer argumento del constructor:

```
use Symfony\Component\Security\Core\Role\Role;

$role = new Role('ROLE_ADMIN');

// will show 'ROLE_ADMIN'
var_dump($role->getRole());
```

La mayoría de **authentication tokens** extienden a **AbstractToken**, lo que significa que los roles proporcionados a su constructor serán convertidos automáticamente de strings a estos **objetos Role**.

### Usando el Decision Manager

#### El Access Listener

El **access decision manager** puede usarse en cualquier punto en un request para decidir si el usuario actual tiene acceso a un resource dado. Un método opcional para restringir el acceso basado en un patrón URL es el [AccessListener](http://api.symfony.com/3.0/Symfony/Component/Security/Http/Firewall/AccessListener.html), que es uno de los **firewall listeners** que se lanzan en cada request que coindiden con el **mapa firewall**. 

Utiliza un mapa de acceso (que ha de ser una instancia de [AccessMapInterface](http://api.symfony.com/3.0/Symfony/Component/Security/Http/AccessMapInterface.html)) que contiene **request matchers** y un **set de atributos** que son requeridos para el usuario actual para que tenga acceso a la aplicación:

```
use Symfony\Component\Security\Http\AccessMap;
use Symfony\Component\HttpFoundation\RequestMatcher;
use Symfony\Component\Security\Http\Firewall\AccessListener;

$accessMap = new AccessMap();
$requestMatcher = new RequestMatcher('^/admin');
$accessMap->add($requestMatcher, array('ROLE_ADMIN'));

$accessListener = new AccessListener(
    $securityContext,
    $accessDecisionManager,
    $accessMap,
    $authenticationManager
);
```

#### Authorization Checker

El **access decision manager** también está disponible para otras partes de la aplicación a través del método _isGranted()_ del [AuthorizationChecker](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Authorization/AuthorizationChecker.html). Una llamada a este método delegará la pregunta al access decision manager:

```
use Symfony\Component\Security\Core\Authorization\AuthorizationChecker;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;

$authorizationChecker = new AuthorizationChecker(
    $tokenStorage,
    $authenticationManager,
    $accessDecisionManager
);

if (!$authorizationChecker->isGranted('ROLE_ADMIN')) {
    throw new AccessDeniedException();
}
```