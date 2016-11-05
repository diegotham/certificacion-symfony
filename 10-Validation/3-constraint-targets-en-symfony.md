Los constraints pueden aplicarse a una **propiedad** (por ejemplo _name_), a un **método getter public** (por ejemplo _getFullName_) o a una clase (**Author**). El primero es el más común y fácil de usar, pero el segundo te permite especificar **reglas de validación más complejas**.

### Propiedades

Validar las propiedades es la técnica de validación más simple. **Symfony** permite validar propiedades _public_, _protected_ y _private_. Una validación sencilla puede ser la siguiente:

```
// src/AppBundle/Entity/Author.php

// ...
use Symfony\Component\Validator\Constraints as Assert;

class Author
{
    /**
     * @Assert\NotBlank()
     * @Assert\Length(min=3)
     */
    private $firstName;
}
```

### Getters

Los constraints también pueden aplicarse al valor de retorno de un método. **Symfony** permite añadir un constraint a cualquier método _public_ cuyo nombre empiece por _get_, _is_ o _has_ (a veces se hace referencia a estos tres como "_getters_", aunque individualmente serían _getters_, _issers_ o _hassers_).

Esta forma permite **validar un objeto dinámicamente**. Por ejemplo, queremos asegurarnos de que en un campo _password_ no es igual al nombre del usuario (por razones de seguridad). Puedes hacer esto creando el método _isPasswordLegal_ con el constraint _isTrue_:

```
// src/AppBundle/Entity/Author.php

// ...
use Symfony\Component\Validator\Constraints as Assert;

class Author
{
    /**
     * @Assert\IsTrue(message = "El password no puede ser tu nombre")
     */
    public function isPasswordLegal()
    {
        return $this->firstName !== $this->password;
    }
}
```

### Classes

Algunos constraints se aplican a clases enteras, como [Callback](http://diego.com.es/constraints-en-symfony#Callback), que es un constraint genérico que se aplica a la propia clase. Cuando se valida la clase, los métodos especificados por el constraint simplemente se ejecutan de forma que cada uno pueda proporcionar también su propia validación.