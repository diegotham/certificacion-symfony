El **componente de seguridad de Symfony** proporciona un sistema de seguridad en dos capas: primero **autentica** al usuario y luego lo **autoriza** para ver si tiene los permisos necesarios para acceder a un resource. La clase que decide los permisos es **AccessDecisionManager**, la cual dispone de varios voters (que puedes crear también). En cada request se pregunta a cada voter si el usuario autenticado tiene el role correcto para la URL actual.

### 1\. Configurar roles estáticos

En el archivo de configuración _security.yml_ puedes añadir **reglas de control de acceso** para definir los roles requeridos:

```
security:
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
```

En el mismo archivo puedes añadir una jerarquía de roles. La siguiente configuración indica que un usuario que tiene el role ROLE_ADMIN puede también acceder a cualquier página del role ROLE_USER:

```
security:
    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: [ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]
```

Cada role es un string, de forma que puedes crear los roles que quieras (siempre empezando por **ROLE_**). Pero si queremos que un role dependa de alguna forma en una propiedad del usuario autenticado, tendremos que crear un **role object**, que implementa **RoleInterface**. Realmente cada role de los anteriores definido como string es convertido a un role object por Symfony.

### 2\. Crear roles dinámicos

**AccessDecisionManager** colecciona el set actual de roles mediante el método _getRoles()_ en el usuario autenticado. Este método se define en **UserInterface**, que cada objeto user debe implementar. Para crear un role dinámico modificamos en la clase User el método _getRoles()_:

```
namespace AppBundle\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use AppBundle\User\UserDependentRole;

class User implements UserInterface
{
    public function getRoles()
    {
        return array(
            'ROLE_USER',
            // ...
            new UserDependentRole($this) // ROLE dinámico
        );
    }
}
```

Ahora creamos la clase que proporciona el role de usuario dinámico: **UserDependentRole**, que ha de implementar **RoleInterface**:

```
namespace AppBundle\Entity;

use Symfony\Component\Security\Core\Role\RoleInterface;
use Symfony\Component\Security\Core\User\UserInterface;

class UserDependentRole implements RoleInterface
{
    private $user;

    public function __construct(UserInterface $user)
    {
        $this->user = $user;
    }

    public function getRole()
    {
        return 'ROLE_' . strtoupper($this->user->getUsername());
    }
}
```

El único requisito de **RoleInterface** es que implementes el método _getRole()_, que debe devolver un string que identifique de forma única al role actual.

Si ahora un usuario se autentica con el username _admin_, tendrá el role ROLE_ADMIN, si lo hace con el username george, tendrá el role ROLE_GEORGE.

Esto permite que sólo usuarios con un nombre de usuario específico puedan entrar a ciertas partes del sitio. Esta forma de hacerlo no es recomendable, pero permite ver las posibilidades de crear roles personalizados.