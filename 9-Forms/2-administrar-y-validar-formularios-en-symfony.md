La segunda tarea en un **formulario** es trasladar el contenido enviado por el usuario a las **propiedades de un objeto**. Para hacerlo, los datos del usuario deben escribirse en el objeto **Form**.

```
// src/AppBundle/Controller/DefaultController.php
use Symfony\Component\HttpFoundation\Request;

public function newAction(Request $request)
{
    // Creamos un nuevo objeto $task
    $task = new Task();

    $form = $this->createFormBuilder($task)
        ->add('task', TextType::class)
        ->add('dueDate', DateType::class)
        ->add('save', SubmitType::class, array('label' => 'Create Task'))
        ->getForm();

    $form->handleRequest($request);

    if ($form->isSubmitted() && $form->isValid()) {
        // ... Llevar a cabo alguna acción como guardar los datos en la base de datos

        return $this->redirectToRoute('task_success');
    }

    return $this->render('default/new.html.twig', array(
        'form' => $form->createView(),
    ));
}
```

El método _createView_ se debe llamar después de que se haya llamado a _handleRequest_. Sino, los cambios efectuados en los eventos ***_SUBMIT** no se aplican a la view (como **errores de validación**).

El **controller** sigue un patrón común para manejar los formularios, y tiene tres posibilidades:

1.  Cuando se carga la página en el navegador, el formulario se crea y renderiza. El método _handleRequest_ detecta que el formulario no se envió y no hace nada. _isValid_ devuelve false si el formulario no se envió.
2.  Cuando el usuario envía el formulario, _handleRequest_ lo detecta e inmediatamente escribe los datos en las propiedades **task** y **dueDate** del objeto **$task**. Es entonces cuando este objeto es validado. Si es inválido, _isValid_ devuelve false de nuevo, y el formulario se muestra de nuevo mostrando los **errores de validación**. Puedes también emplear el método _isSubmitted_ para comprobar que el formulario se ha enviado, independientemente de si los datos enviados son válidos o no.
3.  Cuando el usuario envía el formulario con datos válidos, los datos se escriben de nuevo en el formulario, pero esta vez _isValid_ devuelve **true**. Ahora puedes realizar acciones sobre el objeto _$task_ (como persistirlo a la base de datos) antes de redirigir al usuario a otra página. Redireccionar al usuario después de un envío de formulario satisfactorio previene que el usuario pueda darle a refrescar en el navegador y se envíen de nuevo los datos. 
    Si necesitas más control acerca de **cuando se envía el formulario** o **qué datos se pasan**, puedes emplear [submit](http://symfony.com/doc/current/cookbook/form/direct_submit.html#cookbook-form-call-submit-directly). 

### Enviar formularios con múltiples botones

Cuando el **formulario** contiene más de un **botón de enviar**, querrás comprobar qué botón fue clickado para adaptar el flujo de la aplicación en el controller. Para hacerlo, añadimos un segundo botón al formulario:

```
$form = $this->createFormBuilder($task)
    ->add('task', TextType::class)
    ->add('dueDate', DateType::class)
    ->add('save', SubmitType::class, array('label' => 'Crear Task'))
    ->add('saveAndAdd', SubmitType::class, array('label' => 'Guardar y añadir'))
    ->getForm();
```

En el controller, utilizamos el botón _isClicked_ para consultar si el botón "_Guardar y añadir_" fue clickado:

```
if ($form->isValid()) {
    // ... Acción, como guardar los datos en base de datos

    $nextAction = $form->get('saveAndAdd')->isClicked()
        ? 'task_new'
        : 'task_success';

    return $this->redirectToRoute($nextAction);
}
```

### Validación de formularios

En **Symfony**, la validación se hace en el objeto en cuestión (en este caso **Task**). Más que si el formulario es válido, la comprobación sería si el objeto _$task_ es válido después de que el formulario haya puesto los datos en el mismo. Llamar a _$form->isValid_ es un shortcut que pregunta al objeto _$task_ si tiene datos válidos.

La validación se hace añadiendo restricciones (llamadas _constraints_) a una clase. Añadimos la restricción de que task no puede estar vacío y que dueDate no puede estar vacío y debe ser un objeto válido DateTime:

```
// src/AppBundle/Entity/Task.php
namespace AppBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class Task
{
    /**
     * @Assert\NotBlank()
     */
    public $task;

    /**
     * @Assert\NotBlank()
     * @Assert\Type("\DateTime")
     */
    protected $dueDate;
}
```

#### Validación HTML5

Desde **HTML5** muchos navegadores pueden forzar ciertas restricciones de validación en el lado del cliente. La validación más común se activa poniendo un atributo como _required_ en campos que son requeridos, esto hace que se muestre un mensaje del navegador si el campo no se ha rellenado.

Los formularios generados aprovechan esta funcionalidad añadiendo **atributos HTML** que activan la validación. La validación por el lado del cliente también puede desactivarse con el atributo _novalidate_ en la etiqueta **form** o con _formnovalidate_ en la etiqueta **submit**. Esto es especialmente útil cuando quieres comprobar las restricciones por el lado del servidor. 

```
{# app/Resources/views/default/new.html.twig #}
{{ form(form, {'attr': {'novalidate': 'novalidate'}}) }}
```

#### Grupos de validación

Si tu objeto emplea **grupos de validación**, necesitarás especificar que grupos de validación emplear:

```
$form = $this->createFormBuilder($users, array(
    'validation_groups' => array('registration'),
))->add(...);
```

Si en cambio empleas clases para los **formularios** (que es lo recomendable), tendrás que añadir el método _configureOptions_:

```
use Symfony\Component\OptionsResolver\OptionsResolver;

public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => array('registration'),
    ));
}
```

En ambos casos, solo el grupo de **validación** _registration_ se usará para validar el objeto.

#### Desactivar la validación

A veces puede resultar útil **desactivar la validación** de un formulario entero. Para estos casos se puede establecer la opción **validation_groups** a **false**:

```
use Symfony\Component\OptionsResolver\OptionsResolver;

public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => false,
    ));
}
```

Cuando haces esto, el formulario seguirá haciendo comprobaciones rutinarias, por ejemplo si un archivo subido es demasiado grande o si se envían campos no existentes. Si quieres **eliminar la validación**, puedes emplear el evento [POST_SUBMIT](http://symfony.com/doc/current/cookbook/form/dynamic_form_modification.html#cookbook-dynamic-form-modification-suppressing-form-validation).

#### Grupos basados en los datos enviados

Si necesitas alguna lógica avanzada para determinar los **grupos de validación** (basados en datos enviados), puedes establecer la opción _validation_groups_ a un **array callback**:

```
use Symfony\Component\OptionsResolver\OptionsResolver;

// ...
public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => array(
            'AppBundle\Entity\Client',
            'determineValidationGroups',
        ),
    ));
}
```

Esto llamará al método estático _determineValidationGroups_ en la clase **Client** después de que se haya enviado el **formulario**, pero antes de ejecutar la **validación**. El **objeto Form** se pasa como argumento a ese método. Puedes también definir la lógica con un **Closure**:

```
use AppBundle\Entity\Client;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

// ...
public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => function (FormInterface $form) {
            $data = $form->getData();

            if (Client::TYPE_PERSON == $data->getType()) {
                return array('person');
            }

            return array('company');
        },
    ));
}
```

La opción _validation_groups_ sobreescribe el **grupo de validación por defecto** que se está empleando. Si quieres validar las restricciones por defecto de la entidad también tienes que ajustar la opción añadiendo el grupo en el array devuelto:

```
use AppBundle\Entity\Client;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

// ...
public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'validation_groups' => function (FormInterface $form) {
            $data = $form->getData();

            if (Client::TYPE_PERSON == $data->getType()) {
                return array('Default', 'person');
            }

            return array('Default', 'company');
        },
    ));
}
```

#### Grupos basados en el botón clickado

Cuando el formulario contiene **múltiples botones de envío**, puedes cambiar el grupo de validación dependiendo en qué boton se emplea para enviar el formulario. Por ejemplo, considera un formulario que te permite avanzar al siguiente paso o volver al paso anterior. Cuando se vuelva al paso anterior, los datos guardados deberán ser guardados, pero no validados.

Primero, necesitamos añadir los dos botones al formulario:

```
$form = $this->createFormBuilder($task)
    // ...
    ->add('nextStep', SubmitType::class)
    ->add('previousStep', SubmitType::class)
    ->getForm();
```

Entonces configuramos el botón para volver al paso anterior y ejecutar **grupos específicos de validación**. En este ejemplo, queremos eliminar la validación, por lo que establecemos la opción de _validation_groups_ a false.

```
$form = $this->createFormBuilder($task)
    // ...
    ->add('previousStep', SubmitType::class, array(
        'validation_groups' => false,
    ))
    ->getForm();
```

Ahora el formulario se saltará las restricciones de  validación. De todas formas seguirá validando restricciones básicas de integridad, como comprobar si el archivo subido era demasiado grande o si has intentado enviar texto en un campo numérico.