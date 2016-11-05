Trabajar con **formularios en HTML** es una tarea muy común e importante en el **desarrollo web**. **Symfony** integra un componente de formularios que hace que sea más fácil.

**Indice de contenido**

1.  Fomularios básicos
2.  Formularios en clases
3.  Formularios en services
4.  Formularios incrustados
5.  Formularios sin clases

### 1. Formularios básicos

El siguiente ejemplo es de una **aplicación** de una **lista de tareas** (_todo list_) que mostrará _tasks_. Ya que los usuarios necesitarás editar y crear tareas, necesitarás construir un formulario. Antes de comenzar, primero creamos la clase Taks que representa y guarda los datos para una tarea:

```
// src/AppBundle/Entity/Task.php
namespace AppBundle\Entity;

class Task
{
    protected $task;
    protected $dueDate;

    public function getTask()
    {
        return $this->task;
    }

    public function setTask($task)
    {
        $this->task = $task;
    }

    public function getDueDate()
    {
        return $this->dueDate;
    }

    public function setDueDate(\DateTime $dueDate = null)
    {
        $this->dueDate = $dueDate;
    }
}
```

Esta clase es un **simple objeto PHP** (no tiene nada de **Symfony**) que simplemente soluciona un problema dentro de la aplicación (la necesidad de representar una tarea en la aplicación).

#### Crear el formulario

Una vez creada la clase **Task**, el siguiente paso es **crear y renderizar el formulario HTML**. En Symfony esto se hace construyendo un **objeto form** y renderizarlo en una template. Esto se puede hacer desde el controller (aunque lo recomendable es crear una clase para ello, y así el formulario puede reusarse):

```
// src/AppBundle/Controller/DefaultController.php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use AppBundle\Entity\Task;
use Symfony\Component\Form\Extension\Core\Type\TexType;
use Symfony\Component\Form\Extension\Core\Type\DateType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class DefaultController extends Controller
{
    /**
     * @Route("/newtask", name="newtask")
     */
    public function newAction(Request $request)
    {
        // Creamos una tarea y le ponemos datos dummy
        $task = new Task();
        $task->setTask('Ir a comprar');
        $task->setDueDate(new \DateTime('tomorrow'));

        $form = $this->createFormBuilder($task)
            ->add('task', TextType::class)
            ->add('duedate', DateType::class)
            ->add('save', SubmitType::class, array('label' => 'Crear Task'))
            ->getForm();

        return $this->render('default/new.html.twig', array(
            'form' => $form->createView(),
        ));
    }
}
```

Crear un formulario requiere poco código ya que los objetos form de **Symfony** están construidos con un **form builder**. El objetivo del form builder es permitirte crear _recipes_, y dejar que el builder haga el trabajo pesado.

En este ejemplo hemos añadido dos campos al formulario, _task_ y _dueDate_, que corresponden a las propiedades **task** y **dueDate** de la clase **Task**. También hemos añadido a cada uno un type, **TextType** y **DateType**, representados por su nombre _fully qualified_. Entre otras cosas, determina que **etiquetas HTML** se renderizarán para ese campo. 

Finalmente hemos añadido un botón de enviar con una etiqueta customizada.

#### Renderizar el formulario

Ahora que se ha creado el formulario, el próximo paso es **renderizarlo**. Esto se hace pasando un objeto _form view_ a la template (la parte de _$form->createView()_) y empleando un conjunto de funciones helper:

```
{# app/Resources/views/default/new.html.twig #}
{{ form_start(form) }}
{{ form_widget(form) }}
{{ form_end(form) }}
```

En este ejemplo enviamos el formulario en un POST request y en la misma URL en la que es mostrado.

Para renderizarlo sólo se han necesitado tres líneas:

*   _form_start(form)_. Renderiza la etiqueta de inicio del **formulario**, incluyendo el atributo enctype para la subida de archivos.
*   _form_widget(form)_. Renderiza todos los campos, lo que incluye el propio campo, la etiqueta y cualquier mensajes de error de validación para el campo.
*   _form_end(form)_. Renderiza la etiqueta final y cualquier ampo que no haya sido renderizado todavía, en caso de que hayas renderizado los campos tú mismo uno a uno. Esto es útil para renderizar hidden fields y aprovechar la protección automática contra [ataques CSRF](http://diego.com.es/ataques-csrf-cross-site-request-forgery-en-php). 

Este formulario no es muy flexible, normalmente se renderizan los campos uno a uno para poder personalizarlos, pero este ejemplo es el básico para la explicación de **cómo funcionan los formularios en Symfony**. 

El campo task tiene el valor de la propiedad task el objeto $task (en este caso "Ir a comprar"). Esto es lo primero que hace un formulario: obtiene datos de un objeto y lo traduce en un formato adecuado para ser renderizado en un formulario HTML.

El sistema de formularios puede acceder al valor de la propiedad **protected** task a través del método _getTask_ y _setTask_ en la clase Task. A no ser que una propiedad sea **public**, debe tener un _getter_ o _setter_ para que el **componente Form** pueda obtener y poner datos en la propiedad. Para una propiedad booleana, puedes emplear **isser** o **hasser** (_isPublished()_ o _hasReminder()_) en lugar de **getters** y **setters**.

### 2. Formularios en clases

En el ejemplo anterior se ha creado un formulario directamente en el controller, pero es más conveniente hacerlo en una clase aparte, así puede reusarse en cualquier otro lado de la aplicación. Creamos una nueva clase:

```
// src/AppBundle/Form/Type/TaskType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->add('save', SubmitType::class)
        ;
    }
}
```

Esta nueva clase contiene todo lo necesario para crear una clase de formulario **task**. Puede usarse para construir un **objeto form** rápidamente en el controller:

```
// src/AppBundle/Controller/DefaultController.php
use AppBundle\Form\Type\TaskType;

public function newAction()
{
    $task = ...;
    $form = $this->createForm(TaskType::class, $task);

    // ...
}
```

Cada formulario tiene  que saber el nombre de la clase que almacena los datos (en el ejemplo _AppBundle\Entity\Task_). Normalmente la aplicación conoce la clase debido al segundo argumento pasado a createForm (_$task_). Después, si empiezas a **incrustar formularios**, esto ya no será sufiente, es por ello que se recomienda indicar explícitamente la opción **data_class** añadiéndola en el form type (_TaskType.php_):

```
use Symfony\Component\OptionsResolver\OptionsResolver;

public function configureOptions(OptionsResolver $resolver)
{
    $resolver->setDefaults(array(
        'data_class' => 'AppBundle\Entity\Task',
    ));
}
```

Cuando se mapean formularios a objetos, todos los campos son mapeados. Cualquier campo en el formulario que no exista en el objeto mapeado causará una excepción.

En casos donde necesitas campos extra en el formulario (como la casilla de "aceptar los términos y condiciones") que no sean mapeados por el objeto, necesitas establecer la opción _mapped_ como false:

```
use Symfony\Component\Form\FormBuilderInterface;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder
        ->add('task')
        ->add('dueDate', null, array('mapped' => false))
        ->add('save', SubmitType::class)
    ;
}
```

Además, si hay campos en el formulario donde no se han incluído datos para enviar, se establecerán como _null_.

El campo de datos puede accederse desde el controller:

```
$form->get('dueDate')->getData();
```

El campo de un campo no mapeado puede también modificarse directamente:

```
$form->get('dueDate')->setData(new \DateTime());
```

### 3. Formularios en services

Los **form types** pueden tener algunas dependencias externas. Puedes definir tu form type como un _service_, e inyectar las **dependencias** que necesites.

Para usar un service definido como _app.my_service_ en el form type, creamos un constructor en el form type para recibir el _service_:

```
// src/AppBundle/Form/Type/TaskType.php
namespace AppBundle\Form\Type;

use App\Utility\MyService;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;

class TaskType extends AbstractType
{
    private $myService;

    public function __construct(MyService $myService)
    {
        $this->myService = $myService;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        // You can now use myService.
        $builder
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->add('save', SubmitType::class)
        ;
    }
}
```

Y definimos el form type como _service_:

```
# src/AppBundle/Resources/config/services.yml
services:
    app.form.type.task:
        class: AppBundle\Form\Type\TaskType
        arguments: ["@app.my_service"]
        tags:
            - { name: form.type }
```

### 4. Formularios incrustados

En ocasiones es necesario construir un formulario que incluya campos de objetos diferentes. Por ejemplo, un formulario de registro puede contener datos que pertenecen a un objeto **User** además de objetos **Address**.

Ahora cada **Task** pertenece a un sólo objeto **Category** (primero creamos el objeto Category):

```
// src/AppBundle/Entity/Category.php
namespace AppBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class Category
{
    /**
     * @Assert\NotBlank()
     */
    public $name;
}
```

Después, añadimos una nueva propiedad _category_ en la clase **Task**:

```
// ...

class Task
{
    // ...

    /**
     * @Assert\Type(type="AppBundle\Entity\Category")
     * @Assert\Valid()
     */
    protected $category;

    // ...

    public function getCategory()
    {
        return $this->category;
    }

    public function setCategory(Category $category = null)
    {
        $this->category = $category;
    }
}
```

El constraint **Valid** se ha añadido a la propiedad _category_. Esto hace cascada en la validación de la entidad correspondiente. Si omites este constraint una entidad hija no sería validada. 

Ahora que la aplicación se ha actualizado para reflejar los nuevos requisitos, creamos una clase form para que el objeto Category pueda ser modificado por el usuario:

```
// src/AppBundle/Form/Type/CategoryType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class CategoryType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Category',
        ));
    }
}
```

El objetivo es permitir que la categoría de una tarea pueda ser modificada dentro de la misma tarea. Para conseguirlo, añadimos un campo **category** en el objeto **TaskType** cuyo tipo es una instancia de la nueva clase **CategoryType**:

```
use Symfony\Component\Form\FormBuilderInterface;
use AppBundle\Form\Type\CategoryType;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    // ...

    $builder->add('category', CategoryType::class);
}
```

Los campos de **CategoryType** pueden ahora renderizarse junto a los campos de la clase **TaskType**.

Renderizamos los campos de Category en la misma forma que los campos originales Task:

```
{# ... #}

<h3>Category</h3>
<div class="category">
    {{ form_row(form.category.name) }}
</div>

{# ... #}
```

Cuando el usuario envía el formulario, los datos enviados para los campos **Category** se usan para construir una instancia de Category, la cual es establecida en el campo _category_ de la instancia **Task**.

La instancia Category es accesible a través de _$task->getCategory_ y puede ser persistida a la base de datos o usar su valor como se quiera.

También es posible incrustar una colección de formularios en un formulario (un formulario **Category** con varios formularios **Product**). Esto se puede hacer con el campo collection. Más información en [este enlace](http://symfony.com/doc/current/cookbook/form/form_collections.html).

### 5\. Formularios sin clases

En la mayoría de los casos los formularios van ligados a un objeto, y los campos del formulario obtienen y guardan los datos en las propiedades del objeto.

Pero si en alguna ocasión prefieres **utilizar el formulario sin una clase** y **obtener los datos en un array**, es fácil:

```
// make sure you've imported the Request namespace above the class
use Symfony\Component\HttpFoundation\Request;
// ...

public function contactAction(Request $request)
{
    $defaultData = array('message' => 'Escribe un mensaje aquí');
    $form = $this->createFormBuilder($defaultData)
        ->add('name', TextType::class)
        ->add('email', EmailType::class)
        ->add('message', TextareaType::class)
        ->add('send', SubmitType::class)
        ->getForm();

    $form->handleRequest($request);

    if ($form->isValid()) {
        // Los datos están en un array con los keys "name", "email", y "message"
        $data = $form->getData();
    }

    // ... renderizar el formulario
}
```

Por defecto, un formulario asume que quieres trabajar con **arrays de datos**, en lugar de con un objeto. Hay dos formas de cambiar este comportamiento y **emplear objetos**:

*   Pasar un objeto cuando se crea el formulario (como primer argumento en _createFormBuilder_ o el segundo argumento de _createForm_).
*   Declarar la opción _data_class_ en el formulario.

Si no se hace ninguna de estas dos acciones, el formulario devolverá los datos como un array. En este ejemplo, _$defaultData_ no es un objeto (y no se ha establecido una opción), _$form->getData()_ devuelve un array.

Puedes acceder a valores POST directamente a través del objeto request:

```
$request->request->get('name');
```

De todas formas en la mayoría de los casos la opción _getData()_ es una mejor opción, ya que devuelve los datos (normalmente un objeto) después de haber sido transformados por el **componente Form**.

#### Añadir validación

Cuando se llama a _$form->isValid()_, el objeto se valida a partir de los _constraints_ que hayas aplicado a la clase. Si tu formulario está mapeado con un objeto, esta es casi siempre la forma a emplear. 

Pero si el formulario no está mapeado a un objeto y prefieres extraer un array simple de los datos, puedes establecer los constraints directamente y enlazarlos a sus campos respectivos. 

```
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;
use Symfony\Component\Form\Extension\Core\Type\TextType;

$builder
   ->add('firstName', TextType::class, array(
       'constraints' => new Length(array('min' => 3)),
   ))
   ->add('lastName', TextType::class, array(
       'constraints' => array(
           new NotBlank(),
           new Length(array('min' => 3)),
       ),
   ))
;
```

Si estás usando grupos de validación tienes que referenciar al grupo Default cuando crees el formulario, o establecer el grupo correcto en el constraint que añadas:

```
new NotBlank(array('groups' => array('create', 'update'))
```