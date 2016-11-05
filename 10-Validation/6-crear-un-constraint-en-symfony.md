Puedes crear un constraint personalizado estendiendo la clase base **Constraint**. Como ejemplo vamos a crear un validator simple que compruebe que un string sólo contiene caracteres alfanuméricos:

### Crear la clase Constraint

Primero necesitamos crear una clase Constraint y extender Constraint:

```
// src/AppBundle/Validator/Constraints/ContainsAlphanumeric.php
namespace AppBundle\Validator\Constraints;

use Symfony\Component\Validator\Constraint;

/**
 * @Annotation
 */
class ContainsAlphanumeric extends Constraint
{
    public $message = 'El string "%string%" contiene un carácter no permitido: sólo puede contener letras o números.';
}
```

La anotación **@Annotation** es necesaria para este nuevo constraint para que esté disponible para usar en clases mediante anotaciones. Las opciones para el constraint se representan como propiedades _public_ en la clase constraint.

### Crear el Validator

La validación se produce en otra clase "_constraint validator_". La clase constraint validator se especifica por el método _validatedBy_, que incluye alguna lógica por defecto:

```
// en la clase base Symfony\Component\Validator\Constraint
public function validatedBy()
{
    return get_class($this).'Validator';
}
```

Si creas una clase propia Constraint (como MyConstraint), Symfony buscará automáticamente otra clase, MyConstraintValidator, cuando realice la validación.

La clase validator es simple, y sólo tiene un método requerido, _validate_:

```
// src/AppBundle/Validator/Constraints/ContainsAlphanumericValidator.php
namespace AppBundle\Validator\Constraints;

use Symfony\Component\Validator\Constraint;
use Symfony\Component\Validator\ConstraintValidator;

class ContainsAlphanumericValidator extends ConstraintValidator
{
    public function validate($value, Constraint $constraint)
    {
        if (!preg_match('/^[a-zA-Z0-9]+$/', $value, $matches)) {
            $this->context->buildViolation($constraint->message)
                ->setParameter('%string%', $value)
                ->addViolation();
        }
    }
}
```

Dentro de validate no necesitas un valor de retorno. En su lugar, añades violations a la propiedad del validator _context_, y un valor se considerará válido si no provoca violations. El método _buildViolation_ toma el mensaje de error como argumento y devuelve una instancia de **ConstraintViolationBuilderInterface**. La llamada al método _addViolation_ finalmente añade la violation a context.

### Usar el Validator

Usar validaciones personalizadas es igual que cuando se emplean las que vienen por defecto en Symfony:

```
// src/AppBundle/Entity/AcmeEntity.php
use Symfony\Component\Validator\Constraints as Assert;
use AppBundle\Validator\Constraints as AcmeAssert;

class AcmeEntity
{
    // ...

    /**
     * @Assert\NotBlank
     * @AcmeAssert\ContainsAlphanumeric
     */
    protected $name;

    // ...
}
```

Si tu constraint contiene opciones, han de ser propiedades _public_ en la clase personalizada Constraint que se ha creado antes. Estas opciones pueden ser configuradas como opciones de los constraints incorporados en Symfony.

#### Constraints validators con dependencias

Si tu constraint validator tiene dependencias, como conexión a una base de datos, necesitará ser configurada como un _service_ en el **Dependency Injection Container**. Este _service_ debe incluir la etiqueta _validator.constraint_validator_ y un atributo _alias_:

```
# app/config/services.yml
services:
    validator.unique.your_validator_name:
        class: Fully\Qualified\Validator\Class\Name
        tags:
            - { name: validator.constraint_validator, alias: alias_name }
```

Tu constraint class debe ahora usar este alias para referenciar al validator apropiado:

```
public function validatedBy()
{
    return 'alias_name';
}
```

Symfony buscará automáticamente un nombre de clase seguido de la palabra **Validator**. Si tu constraint validator está definido como _service_, es importante que sobreescribas el método _validatedBy_ para devolver el alias empleado al definir el _service_, sino Symfony no empleará el _constraint validator service_, e instanciará la clase sin dependencias inyectadas.

#### Clase Constraint Validator

Además de validar una propiedad de clase, un constraint puede tener un _class scope_ proporcionando un target en su clase **Constraint**:

```
public function getTargets()
{
    return self::CLASS_CONSTRAINT;
}
```

Con esto, el método _validate_ obtiene un objeto como su primer argumento:

```
class ProtocolClassValidator extends ConstraintValidator
{
    public function validate($protocol, Constraint $constraint)
    {
        if ($protocol->getFoo() != $protocol->getBar()) {
            $this->context->buildViolation($constraint->message)
                ->atPath('foo')
                ->addViolation();
        }
    }
}
```

Observa cómo el _class constraint validator_ se aplica a la propia clase, y no a la propiedad:

```
/**
 * @AcmeAssert\ContainsAlphanumeric
 */
class AcmeEntity
{
    // ...
}
```