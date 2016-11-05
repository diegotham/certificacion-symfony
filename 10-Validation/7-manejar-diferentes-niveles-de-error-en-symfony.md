En ciertas ocasiones puede ser provechoso **mostrar mensajes de errores de validación diferentes en función de ciertas normas**.

Por ejemplo, en un formulario de registro para nuevos usuarios donde introducen alguna información personal y eligen sus credenciales de autenticación. Tienen que proporcionar un nombre de usuario y un password, pero la información de cuenta bancaria es opcional. Sin embargo, quieres asegurarte de que estos campos opcionales, si se introducen, son válidos, pero muestran sus errores de forma diferente.

El proceso se lleva a cabo en **dos pasos**:

1.  Aplicar diferentes niveles de error en los constraints de validación.
2.  Customizar los mensajes de error dependiendo del nivel de error configurado.

### 1\. Asignar el nivel de error

Usamos la opción payload para **configurar el nivel de error de cada constraint**:

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class User
{
    /**
     * @Assert\NotBlank(payload = {"severity" = "error"})
     */
    protected $username;

    /**
     * @Assert\NotBlank(payload = {"severity" = "error"})
     */
    protected $password;

    /**
     * @Assert\Iban(payload = {"severity" = "warning"})
     */
    protected $bankAccountNumber;
}
```

### 2\. Customizar la template del mensaje de error

Cuando la validación del objeto **User** falla, puedes recuperar el constraint que causó un fallo en concreto con el método _getConstraint_. Cada constraint expone el **payload** como una propiedad _public_:

```
// fallo del constraint de validación, instancia de
// Symfony\Component\Validator\ConstraintViolation
$constraintViolation = ...;
$constraint = $constraintViolation->getConstraint();
$severity = isset($constraint->payload['severity']) ? $constraint->payload['severity'] : null
```

Ahora por ejemplo podemos customizar el bloque _form_errors_ de forma que la severidad se añade como una clase adicional HTML:

```
{%- block form_errors -%}
    {%- if errors|length > 0 -%}
    <ul>
        {%- for error in errors -%}
            {% if error.cause.constraint.payload.severity is defined %}
                {% set severity = error.cause.constraint.payload.severity %}
            {% endif %}
            <li{% if severity is defined %} class="{{ severity }}"{% endif %}>{{ error.message }}</li>
        {%- endfor -%}
    </ul>
    {%- endif -%}
{%- endblock form_errors -%}
```