La **validación** es una tarea muy común en **aplicaciones web**. Los datos introducidos en **formularios** han de ser validados, además de validarse antes de introducirse en una **base de datos** o pasarse a un [web service](http://diego.com.es/introduccion-a-los-web-services).

Symfony viene con un **componente de validación** que hace esta tarea más sencilla. Este componente se basa en la [JSR303 Bean Validation specification](http://jcp.org/en/jsr/detail?id=303).

1.  Lo básico
2.  Validación en formularios
3.  Validar valores y arrays

### 1\. Lo básico

Tenemos un **objeto en PHP** puro que usaremos después en la **aplicación**:

```
// src/AppBundle/Entity/Author.php
namespace AppBundle\Entity;

class Author
{
    public $name;
}
```

Ahora sólo es una clase normal que tiene un objetivo en la aplicación. El objetivo de la validación es decirte si los datos de un objeto son válidos. Para ello, configuramos una lista de normas (llamadas _constraints_) que el objeto debe seguir para ser válido. Estas normas pueden especificarse en diferentes formatos (**YAML**, **XML**, **anotaciones** o **PHP**). 

```
// src/AppBundle/Entity/Author.php

// ...
use Symfony\Component\Validator\Constraints as Assert;

class Author
{
    /**
     * @Assert\NotBlank()
     */
    public $name;
}
```

Las propiedades _protected_ y _private_ también pueden ser validadas, al igual que los métodos _getter_. 

Ahora para validar realmente el objeto **Author**, usamos el método _validate_ en el _validator service_ (clase **Validator**). El validator lo que hace es lees las constraints (restricciones) de una clase y verificar si los datos del objeto satisfacen los constraints. Si la validación falla, se devuelve una lista no vacía de errores (clase **ConstraintViolationList**). Ejemplo desde dentro de un **controller**:

```
// ...
use Symfony\Component\HttpFoundation\Response;
use AppBundle\Entity\Author;

// ...
public function authorAction()
{
    $author = new Author();

    // ... hacer algo al objeto $author

    $validator = $this->get('validator');
    $errors = $validator->validate($author);

    if (count($errors) > 0) {
        /*
         * Utiliza un método __toString en la variable $errors el cual es un
         * objeto ConstraintViolationList. Esto nos da un string para hacer
         * debugging.
         */
        $errorsString = (string) $errors;

        return new Response($errorsString);
    }

    return new Response('El autor es válido!');
}
```

Si la propiedad _$name_ está vacía, verás el siguiente mensaje de error:

```
AppBundle\Author.name:
    This value should not be blank
```

Si insertas un valor en la propiedad _name_, aparecerá el mensaje de éxito.

La mayoría de las veces no tendremos que interactuar directamente con el validator service ni nos tendremos que preocupar sobre imprimir los errores. Se suele usar la validación indirectamente cuando se trata de datos de formularios.

También podríamos pasar una colección de errores a una template:

```
if (count($errors) > 0) {
    return $this->render('author/validation.html.twig', array(
        'errors' => $errors,
    ));
}
```

Dentro de la template, mostraríamos los errores:

```
{# app/Resources/views/author/validation.html.twig #}
<h3>The author has the following errors</h3>
<ul>
{% for error in errors %}
    <li>{{ error.message }}</li>
{% endfor %}
</ul>
```

Cada **error de validación**, llamado "_constraint violation_", se representa por un objeto **ConstraintViolation**.

### 2\. Validación en formularios

El _validator service_ se puede utilizar cada vez que se quiera validar un objeto, pero normalmente se trabaja indirectamente. La librería de formularios de Symfony emplea el _validator service_ internamente para validar el objeto una vez que los valores se han enviado. Las constraint violations en el objeto se convierten en objetos **FormError** que pueden mostrarse fácilmente con tu formulario. El típico formulario en Symfony es algo así:

```
// ...
use AppBundle\Entity\Author;
use AppBundle\Form\AuthorType;
use Symfony\Component\HttpFoundation\Request;

// ...
public function updateAction(Request $request)
{
    $author = new Author();
    $form = $this->createForm(AuthorType::class, $author);

    $form->handleRequest($request);

    if ($form->isValid()) {
        // La validación ha pasado, hacer algo con el objeto $author

        return $this->redirectToRoute(...);
    }

    return $this->render('author/form.html.twig', array(
        'form' => $form->createView(),
    ));
}
```

El ejemplo emplea una clase de formulario **AuthorType**, que no se muestra aquí.

El **Symfony validator** está disponible por defecto, pero se ha de activar explícitamente las anotaciones si empleas este método para validar:

```
# app/config/config.yml
framework:
    validation: { enable_annotations: true }
```

### 3\. Validación de valores y arrays

Además de validar objetos también se pueden validar simplemente **valores**, como que un string sea realmente una dirección de email.

```
// ...
use Symfony\Component\Validator\Constraints as Assert;

// ...
public function addEmailAction($email)
{
    $emailConstraint = new Assert\Email();
    // las opciones del constraint pueden establecerse de esta forma
    $emailConstraint->message = 'Invalid email address';

    // usa el validator para validar el valor
    $errorList = $this->get('validator')->validate(
        $email,
        $emailConstraint
    );

    if (0 === count($errorList)) {
        // ... es una dirección de email válida
    } else {
        // no es una dirección de email válida
        $errorMessage = $errorList[0]->getMessage();

        // ... hacer algo con el error
    }

    // ...
}
```

Llamando a _validate_ en el **validator** podemos pasar un valor y el objeto constraint con el que quieres validarlo. El método validate devuelve un objeto **ConstraintViolationList**, que actúa como un array de errores. Cada error en la colección es un objeto **ConstraintViolation**, que guarda el mensaje de error en su método _getMessage_.