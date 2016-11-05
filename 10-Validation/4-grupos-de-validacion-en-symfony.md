En ocasiones sólo necesitamos validar un objeto frente a algunos constraints en una clase. Para esto se organiza cada constraint en uno o más **grupos de validación**, y entonces se aplica la validación con sólo un grupo de constraints determinado.

Si por ejemplo tenemos una clase **User**, que se usa tanto cuando el usuario se registra como cuando actualiza su información de contacto después:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Validator\Constraints as Assert;

class User implements UserInterface
{
    /**
     * @Assert\Email(groups={"registration"})
     */
    private $email;

    /**
     * @Assert\NotBlank(groups={"registration"})
     * @Assert\Length(min=7, groups={"registration"})
     */
    private $password;

    /**
     * @Assert\Length(min=2)
     */
    private $city;
}
```

Con esta configuración hay tres grupos de validación:

*   **Default**. Contiene los constraints en la clase actual y todas las clases referenciadas que no pertenecen a otro grupo.
*   **User**. Equivalente a todos los constrainst del objeto User en el grupo Default. Este es siempre el nombre de la clase.
*   **registration**. Contiene los constraints en los campos _email_ y _password_. 

Los constraints de una clase en el grupo Default son los que o no tienen un grupo configurado o que están configurados en un grupo igual al nombre de la clase o al string Default.

Cuando se valida el objeto **User**, no hay diferencia entre el grupo **Default** y el grupo **User**. Pero hay diferencia si User tiene **objetos incrustados**. Por ejemplo si User tiene una propiedad _address_ que contiene un objeto **Address** en el que has añadido el constraint **Valid** en la propiedad de forma que sea validado cuando se valide el objeto User.

Si validas User con el grupo Default, cualquier constraint en la clase Address que están en el grupo Default se validarán. Pero si validas User con el grupo de validación User, los únicos constraints en la clase Address que se validarán serán los de la clase User.

En otras palabras: el grupo Default y el grupo con el nombre de la clase (User) son idénticos, salvo cuando la clase está incrustada en otro objeto que es el que realmente se va a validar.

Si hay herencia, (_User extends BaseUser_) y validas con el nombre de la subclase (User), todos los constraints en User y BaseUser se validarán. Sin embargo, si validas la clase base (BaseUser), sólo se validarán los constraints de esta clase.

Para decirle al validator que use un grupo específico, pasa uno o más nombres de grupo como el tercer argumento del método validate():

```
$errors = $validator->validate($author, null, array('registration'));
```

Si no se especifica ningún grupo, se validarán todos los constraints que pertenezcan al grupo Default.