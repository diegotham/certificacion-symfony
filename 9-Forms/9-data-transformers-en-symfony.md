Los **Data Transformers** se usan para traducir los datos de un campo en un formato que pueda mostrarse en un formulario y enviarse. Se usan internamente para varios _field types_. Por ejemplo, el field **DateType** puede renderizarse en formato **yyyy-MM-dd** en una caja de texto. Internamente, un Data Transformer convierte el valor inicial DateTime del campo en el string yyy-MM-dd para renderizar el formulario, y después de nuevo en un objeto DateTime al enviar el formulario.

Cuando un campo de formulario tiene la opción _inherit_data_ establecida, los Data Transformers no se aplicarán en ese campo.

### Ejemplo 1: Sanitizar HTML

Suponemos que tenermos un formulario **Task** con una descripción en un _textarea_:

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;

// ...
class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('description', TextareaType::class);
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Task',
        ));
    }

    // ...
}
```

Queremos: 

*   Que los usuarios puedan emplear un tipo de **etiquetas HTML** pero no otras. Habría que llamar a _strip_tags_ trans el envío del formulario.
*   Convertir **etiquetas </br>** en **saltos de línea \n** antes de renderizar el formulario.

Es un buen momento para incluir un data transformer en el campo _description_. La forma más fácil de hacerlo es con la clase [CallbackTransformer](http://api.symfony.com/3.0/Symfony/Component/Form/CallbackTransformer.html):

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\CallbackTransformer;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
// ...

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('description', TextareaType::class);

        $builder->get('description')
            ->addModelTransformer(new CallbackTransformer(
                // transforma <br/> en \n para que se lea mejor el textarea
                function ($originalDescription) {
                    return preg_replace('#<br\s*/?>#i', "\n", $originalDescription);
                },
                function ($submittedDescription) {
                    // elimina la mayoría de etiquetas HTML (pero no br,p)
                    $cleaned = strip_tags($submittedDescription, '<br><br/><p>');

                    // transforma cualquier \n en <br/>
                    return str_replace("\n", '<br/>', $cleaned);
                }
            ))
        ;
    }

    // ...
}
```

El **CallbackTransformer** toma dos funciones callback como argumentos. El primero transforma el valor original en un formato que se usará para renderizar el campo. El segundo hace lo contrario: transforma el valor enviado en el formato que se usará en el código.

El método _addModelTransformer_ acepta cualquier objeto que implemente DataTransformerInterface, por lo que puedes crear tus propias clases, en lugar de poner toda la lógica en el formulario.

También podemos añadir el **transformer** cuando añadimos el campo:

```
use Symfony\Component\Form\Extension\Core\Type\TextareaType;

$builder->add(
    $builder->create('description', TextareaType::class)
        ->addModelTransformer(...)
);
```

### Ejemplo 2: Transformar un string en una entidad

Por ejemplo tenemos una **relación many-to-one** de la entidad **Task** a la entidad **Issue** (cada Task tiene un **foreign key** opcional a su Issue). Añadir una _listbox_ con todos los posibles Issues puede resultar muy largo y generar mucho tiempo de carga. Decidimos añadir una _texbox_ donde el usuario pueda introducir el número de _issue_.

Establecemos el _text field_ normal:

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;

// ...
class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('description', TextareaType::class)
            ->add('issue', TextType::class)
        ;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Task'
        ));
    }

    // ...
}
```

Ahora queremos transformar lo que se reciba del campo de texto _issue_ en una entidad **Issue**. Podrías usar el **CallbackTransformer** como antes. Pero ya que esto es algo más complejo, crear una nueva clase transformer mantendrá la clase del formulario **TaskType** más simple.

Creamos una clase **IssueToNumberTransformer**, responsable de hacer las transformaciones correspondientes:

```
// src/AppBundle/Form/DataTransformer/IssueToNumberTransformer.php
namespace AppBundle\Form\DataTransformer;

use AppBundle\Entity\Issue;
use Doctrine\Common\Persistence\ObjectManager;
use Symfony\Component\Form\DataTransformerInterface;
use Symfony\Component\Form\Exception\TransformationFailedException;

class IssueToNumberTransformer implements DataTransformerInterface
{
    private $manager;

    public function __construct(ObjectManager $manager)
    {
        $this->manager = $manager;
    }

    /**
     * Transforms an object (issue) to a string (number).
     *
     * @param  Issue|null $issue
     * @return string
     */
    public function transform($issue)
    {
        if (null === $issue) {
            return '';
        }

        return $issue->getId();
    }

    /**
     * Transforms a string (number) to an object (issue).
     *
     * @param  string $issueNumber
     * @return Issue|null
     * @throws TransformationFailedException if object (issue) is not found.
     */
    public function reverseTransform($issueNumber)
    {
        // el issue number es opcional
        if (!$issueNumber) {
            return;
        }

        $issue = $this->manager
            ->getRepository('AppBundle:Issue')
            // consulta el issue con el id
            ->find($issueNumber)
        ;

        if (null === $issue) {
            // causa un error de validación
            // este mensaje no se muestra al usuario
            // mira la opción invalid_message
            throw new TransformationFailedException(sprintf(
                'Un issue con el número "%s" no existe!',
                $issueNumber
            ));
        }

        return $issue;
    }
}
```

Al igual que en el primer ejemplo, un **transformer tiene dos direcciones**. El método _transform_ convierte los datos de tu código en un formato que se puede mostrar en el formulario (por ejemplo el objeto Issue en su id, un string). El método _reverseTransform_ hace lo contrario convierte el valor enviado de vuelta al formato que desees (por ejemplo convertir el id de nuevo en un objeto Issue).

Para **provocar un error de validación**, lanza una **TransformationFailedException**. El mensaje que se pase a esta [excepción](http://diego.com.es/excepciones-en-php) no se mostrará al usuario. Establecemos el mensaje con la opción _invalid_message_.

Cuando se pasa _null_ al método _transform_, el transformer debería devolver un valor equivalente al tipo al que se quiere transformar: un string vacío, 0 para integers o 0.0 para floats.

Ahora ya podemos instanciar la clase **IssueToNumberTransformer** de **TaskType** y añadirlo al campo _issue_. Pero para hacerlo necesitamos una instancia del **entity manager** (porque IssueToNumberTransformer lo necesita). Para esto simplemente añadimos un constructor a **TaskType** y lo registramos como service:

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use AppBundle\Form\DataTransformer\IssueToNumberTransformer;
use Doctrine\Common\Persistence\ObjectManager;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;

// ...
class TaskType extends AbstractType
{
    private $manager;

    public function __construct(ObjectManager $manager)
    {
        $this->manager = $manager;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('description', TextareaType::class)
            ->add('issue', TextType::class, array(
                // validation message if the data transformer fails
                'invalid_message' => 'Ese no es un número de issue válido!',
            ));

        // ...

        $builder->get('issue')
            ->addModelTransformer(new IssueToNumberTransformer($this->manager));
    }

    // ...
}
```

Ahora definimos el _form type_ como _service_ en los **archivos de configuración**:

```
# src/AppBundle/Resources/config/services.yml
services:
    app.form.type.task:
        class: AppBundle\Form\Type\TaskType
        arguments: ["@doctrine.orm.entity_manager"]
        tags:
            - { name: form.type }
```

Ahora podemos usar el **TaskType**:

```
// por ejemplo en un controller
$form = $this->createForm(TaskType::class, $task);

// ...
```

Ahora un usuario podrá introducir un **número de issue** en el _text field_ y se transformará en un **objeto Issue**. Esto significa que, después de un envío satisfactorio, el **componente Form** parará un objeto Issue a _Task::setIssue()_ en lugar del número de issue. 

Si no se encuentra el issue, se creará un error de formulario para ese campo y su mensaje de error puede ser controlado con la opción _invalid_message_.

Hay que tener cuidado al emplear transformers. Por ejemplo el siguiente ejemplo está mal, ya que el transformer se aplicará a todo el formulario:

```
// ESTO ESTÁ MAL
$builder->add('issue', TextType::class)
    ->addModelTransformer($transformer);
```

### Crear un campo reusable issue_selector

En el ejemplo anterior hemos aplicado el transformer a un _text field_ normal. Pero si esta transformación se repite, podemos [crear un tipo de campo de formulario](http://diego.com.es/crear-un-tipo-de-campo-de-formulario-en-symfony) con la transformación. 

Primero creamos un field type customizado:

```
// src/AppBundle/Form/IssueSelectorType.php
namespace AppBundle\Form;

use AppBundle\Form\DataTransformer\IssueToNumberTransformer;
use Doctrine\Common\Persistence\ObjectManager;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class IssueSelectorType extends AbstractType
{
    private $manager;

    public function __construct(ObjectManager $manager)
    {
        $this->manager = $manager;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $transformer = new IssueToNumberTransformer($this->manager);
        $builder->addModelTransformer($transformer);
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'invalid_message' => 'The selected issue does not exist',
        ));
    }

    public function getParent()
    {
        return TextType::class;
    }
}
```

Esto actuará y se renderizará como un _text field_ (_getParent()_), pero tendrá automáticamente el data transformer y un valor por defecto para la opción _invalid_message_.

Ahora registramos el type como un service y lo etiquetamos con form.type de forma que se reconozca como un field type customizado:

```
# app/config/services.yml
services:
    app.type.issue_selector:
        class: AppBundle\Form\IssueSelectorType
        arguments: ['@doctrine.orm.entity_manager']
        tags:
            - { name: form.type }
```

Ahora cuando necesitemos usar un field type especial issue_selector, tan sólo tenemos que hacer lo siguiente:

```
// src/AppBundle/Form/TaskType.php
namespace AppBundle\Form\Type;

use AppBundle\Form\DataTransformer\IssueToNumberTransformer;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
// ...

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('description', TextareaType::class)
            ->add('issue', IssueSelectorType::class)
        ;
    }

    // ...
}
```

### Model y View Transformers

En el ejemplo anterior, el transformer se ha utilizado como un "model" transformer. Hay dos tipos de transformers y tres tipos de datos:

![Tipos de data transformers y datos en Symfony](http://symfony.com/doc/current/_images/DataTransformersTypes.png)

En cualquier formulario, los tres tipos de datos son:

*   **Model data**. Estos son los datos en el formato que se usa en la aplicación (por ejemplo un objeto Issue). Si llamas a _Form::getData()_ o _Form::setData()_, tratarás con el "model" data.
*   **Norm data**. Esta es una versión normalizada de los datos y es normalmente lo mismo que el "model" data (aunque no en el ejemplo de antes). No es frecuente usarlo directamente.
*   **View data**. Es el formato utilizado para rellenar los campos de formulario. También es el formato en el que el usuario enviará los datos. Cuando llamas a _Form::submit($data)_, el _$data_ es en el formato de datos del view.

Los dos tipos diferentes de transformers ayudan a convertir entre cada uno de estos tipos de datos.

**Model transformers:**

*   **transform**: model data => norm data
*   **reverseTransform**: norm data => model data

**View transformers:**

*   **transform**: norm data => view data
*   **reverseTransform**: view data => norm data

El transformer que se necesite depende de la situación.

En el ejemplo de antes, el campo es un _text field_, del cual se espera que siempre sea simple, en formato escalable en los formatos "norm" y "view". Por esta razón, **el transformer más apropiado** era el "model" transformer (que convierte del formato _norm_ -> _string issue number_ al formato _model_ -> _Issue object_).

La **diferencia entre transformers** es sutil y siempre hay que pensar como debería ser el "norm" de un campo. Por ejemplo el "norm" data para un _text fied_ es un _string_, pero es un objeto _DateTime_ para un _date field_.