Los **grupos de validación en Symfony** se pueden validar por pasos. Para hacerlo, podemos emplear la funcionalidad **GroupSequence**, de forma que un objeto pueda definir una secuencia de grupos determinando el orden en el que se validen.

Si por ejemplo tenemos una clase User y queremos validar que el nombre de usuario y el password sean diferentes sólo si todas las otras validaciones pasan la comprobación (para evitar mensajes de error múltiples), podemos hacer lo siguiente:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Validator\Constraints as Assert;

/**
 * @Assert\GroupSequence({"User", "Strict"})
 */
class User implements UserInterface
{
    /**
     * @Assert\NotBlank
     */
    private $username;

    /**
     * @Assert\NotBlank
     */
    private $password;

    /**
     * @Assert\IsTrue(message="El password no puede ser igual al nombre de usuario", groups={"Strict"})
     */
    public function isPasswordLegal()
    {
        return ($this->username !== $this->password);
    }
}
```

En este ejemplo primero validamos todos los constraints del grupo **User** (que es el mismo que el grupo Default). Sólo si todos los constraints de ese grupo son válidos, se validará el segundo grupo, **Strict**. 

Realmente con **Group Sequences** el grupo **User** y **Default** ya no son idénticos, el grupo Default hace referencia a la secuencia de grupos, en lugar de a todos los constraints que no pertenecen a ningún grupo. Esto significa que tienes que usar el grupo del nombre de clase (como User) cuando se especifica una secuencia de grupos. Si en cambio usas el grupo Default se producirá un loop infinito.

### Group Sequence Providers

Tenemos una entidad User que puede ser un **usuario normal** o un **usuario premium**. Cuando sea premium user, se añadirán algunos constraints extra a la entidad (como detalles de la tarjeta de crédito). Para determinar dinámicamente qué grupos deben activarse, puedes crear un **Group Sequence Provider**. Primero creamos la entidad y añadimos un grupo constraint llamado Premium:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class User
{
    /**
     * @Assert\NotBlank()
     */
    private $name;

    /**
     * @Assert\CardScheme(
     *     schemes={"VISA"},
     *     groups={"Premium"},
     * )
     */
    private $creditCard;

    // ...
}
```

Ahora, cambiamos la clase **User** para implementar **GroupSequenceProviderInterface** y añadimos el método _getGroupSequence_, que debe devolver un array de grupos a usar:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

// ...
use Symfony\Component\Validator\GroupSequenceProviderInterface;

class User implements GroupSequenceProviderInterface
{
    // ...

    public function getGroupSequence()
    {
        $groups = array('User');

        if ($this->isPremium()) {
            $groups[] = 'Premium';
        }

        return $groups;
    }
}
```

Finalmente le decimos al componente Validator que la clase User proporciona una secuencia de grupos para ser validada:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

// ...

/**
 * @Assert\GroupSequenceProvider
 */
class User implements GroupSequenceProviderInterface
{
    // ...
}
```