A veces un formulario no se puede crear de forma estática. En esta sección se explica cómo customizar un formulario en tres casos de uso:

1.  Formulario basado en datos subyacentes
2.  Generar dinámicamente formularios según datos de usuarios
3.  Generación dinámica de formularios enviados
4.  Eliminar la validación de formularios

### 1\. Formulario basado en datos subyacentes

**Ejemplo**: tenemos un formulario Product y queremos _modificar/añadir/eliminar_ un campo basándonos en los datos subyacentes del producto que se está editando.

Antes de ir a la generación dinámica del formulario, vamos a ver primero una clase de formulario:

```
// src/AppBundle/Form/Type/ProductType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('name');
        $builder->add('price');
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Product'
        ));
    }
}
```

Asumimos que este formulario emplea una clase **Product** que tiene sólo dos propiedades, _name_ y _price_. El formulario generado de esta clase se verá igual independientemente de si se **crea un nuevo producto** o si se está **editando** (producto obtenido de la base de datos).

Suponemos ahora que no queremos que el usuario pueda cambiar el valor _name_ una vez que el objeto se ha creado. Para hacer esto podemos emplear el [componente Event Dispatcher](http://diego.com.es/event-dispatcher-en-symfony) para analizar los datos en el objeto y modificar el formulario basándonos en los datos del objeto Product.

#### Añadir un Event Listener a una Form Class

En lugar de añadir directamente el widget _name_, la responsabilidad de crear ese campo en particular se delega a un **event listener**:

```
// src/AppBundle/Form/Type/ProductType.php
namespace AppBundle\Form\Type;

// ...
use Symfony\Component\Form\FormEvent;
use Symfony\Component\Form\FormEvents;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('price');

        $builder->addEventListener(FormEvents::PRE_SET_DATA, function (FormEvent $event) {
            // ... añadir el campo name si se necesita
        });
    }

    // ...
}
```

El objetivo es crear un campo _name_ sólo si el objeto Product es nuevo (no se ha persistido a la base de datos). Con esta condición el event listener puede ser como sigue:

```
// ...
public function buildForm(FormBuilderInterface $builder, array $options)
{
    // ...
    $builder->addEventListener(FormEvents::PRE_SET_DATA, function (FormEvent $event) {
        $product = $event->getData();
        $form = $event->getForm();

        // comprobar si el objeto Product es "nuevo"
        // Si no se pasan datos al formulario, los datos son "null".
        // Esto debería de considerar un nuevo "Product"
        if (!$product || null === $product->getId()) {
            $form->add('name', TextType::class);
        }
    });
}
```

La línea **FormEvents::PRE_SET_DATA** determina el string _form.pre_set_data_. Los **FormEvents** sirven para organizar y manipular los campos del formulario y las clases sobre las que se asientan. Se pueden ver todos los eventos de formulario en la clase [FormEvents](http://api.symfony.com/3.0/Symfony/Component/Form/FormEvents.html).

#### Añadir un Event Subscriber a una Form Class

Para mejor reusabilidad o si hay una lógica más compleja en el **event listener**, puedes crear la lógica para crear un campo name a un [event subscriber](http://diego.com.es/event-listeners-y-subscribers-en-symfony):

```
// src/AppBundle/Form/Type/ProductType.php
namespace AppBundle\Form\Type;

// ...
use AppBundle\Form\EventListener\AddNameFieldSubscriber;

class ProductType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('price');

        $builder->addEventSubscriber(new AddNameFieldSubscriber());
    }

    // ...
}
```

Ahora la lógica para crear el campo _name_ reside en su propia clase subscriber:

```
// src/AppBundle/Form/EventListener/AddNameFieldSubscriber.php
namespace AppBundle\Form\EventListener;

use Symfony\Component\Form\FormEvent;
use Symfony\Component\Form\FormEvents;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\Form\Extension\Core\Type\TextType;

class AddNameFieldSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        // Le dice al dispatcher que quieres llamar con el evento
        // form.pre_set_data y que debe llamarse al método preSetData.
        return array(FormEvents::PRE_SET_DATA => 'preSetData');
    }

    public function preSetData(FormEvent $event)
    {
        $product = $event->getData();
        $form = $event->getForm();

        if (!$product || null === $product->getId()) {
            $form->add('name', TextType::class);
        }
    }
}
```

### 2. Generar dinámicamente formularios según datos de usuarios

En ocasiones queremos crear un formulario dinámicamente basándonos no sólo en datos del formulario sino en algo más, como en datos que proporcione el usuario. Por ejemplo tenemos un website social donde un usuario sólo puede enviar mensajes a gente marcada como amigos. En este caso, se crea una "choice list" de sólo los amigos a los que poder enviar un mensaje.

#### Crear el Form Type

Con un event listener el formulario queda así:

```
// src/AppBundle/Form/Type/FriendMessageFormType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\FormEvents;
use Symfony\Component\Form\FormEvent;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;

class FriendMessageFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('subject', TextType::class)
            ->add('body', TextareaType::class)
        ;
        $builder->addEventListener(FormEvents::PRE_SET_DATA, function (FormEvent $event) {
            // ... añadir una choice list de amigos del usuario actual
        });
    }
}
```

El problema ahora es obtener el usuario actual y crear un choice field que contiene sólo los amigos de este usuario. Para ello inyectamos el service en el formulario mediante el constructor:

```
private $tokenStorage;

public function __construct(TokenStorageInterface $tokenStorage)
{
    $this->tokenStorage = $tokenStorage;
}
```

Teniendo acceso al usuario podríamos usarlo directamente en el **buildForm** y omitir el **event listener**, pero el método _buildForm_ modificaría el _form type_ entero y no sólo en esta instancia. Esto puede no ser un problema normalmente, pero a veces un form type puede usarse en un **request** para crear varios formularios o campos.

#### Customizar el Form Type

Ahora que sabemos lo básico podemos aprovechar el **TokenStorageInterface** y rellenar la lógica del listener:

```
// src/AppBundle/FormType/FriendMessageFormType.php

use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;
use Doctrine\ORM\EntityRepository;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;

// ...

class FriendMessageFormType extends AbstractType
{
    private $tokenStorage;

    public function __construct(TokenStorageInterface $tokenStorage)
    {
        $this->tokenStorage = $tokenStorage;
    }

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('subject', TextType::class)
            ->add('body', TextareaType::class)
        ;

        // cogemos el usuario y hacemos una comprobación de que existe
        $user = $this->tokenStorage->getToken()->getUser();
        if (!$user) {
            throw new \LogicException(
                'El FriendMessageFormType no puede usarse sin un usuario identificado!'
            );
        }

        $builder->addEventListener(
            FormEvents::PRE_SET_DATA,
            function (FormEvent $event) use ($user) {
                $form = $event->getForm();

                $formOptions = array(
                    'class' => 'AppBundle\Entity\User',
                    'property' => 'fullName',
                    'query_builder' => function (EntityRepository $er) use ($user) {
                        // consulta customizada
                        // devolver $er->createQueryBuilder('u')->addOrderBy('fullName', 'DESC');

                        // o llamar a un método en tu repositorio que devuelve el query builder
                        // $er es una instancia de tu UserRepository
                        // devuelve $er->createOrderByFullNameQueryBuilder();
                    },
                );

                // crear el campo, esto es similar a $builder->add()
                // nombre del campo, field type, datos, opciones
                $form->add('friend', EntityType::class, $formOptions);
            }
        );
    }

    // ...
}
```

Las opciones _multiple_ y _expanded_ son por defecto false porque el _type_ del campo _friend_ es **EntityType::class**.

#### Emplear el formulario

El formulario está ahora listo para usarse. Pero primero necesitamos resgitrarlo como _service_ y ponerle una _tag_ con _form.type_:

```
# app/config/config.yml
services:
    app.form.friend_message:
        class: AppBundle\Form\Type\FriendMessageFormType
        arguments: ['@security.token_storage']
        tags:
            - { name: form.type }
```

En un **controller** que extiende la clase **Controller**, puedes simplemente llamar:

```
use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class FriendMessageController extends Controller
{
    public function newAction(Request $request)
    {
        $form = $this->createForm(FriendMessageFormType::class);

        // ...
    }
}
```

También puedes fácimente incrustar el _form type_ en otro formulario:

```
// inside some other "form type" class
public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder->add('message', FriendMessageFormType::class);
}
```

### 3. Generación dinámica de formularios enviados

Otro caso que se puede dar es cuando quieres customizar un formulario a los datos especificados por el usuario. Por ejemplo, tenemos un formulario de registro para quedadas deportivas. Algunos eventos te permitirán especificar tu posición preferida en el campo. Esto sería un **choice field** por ejemplo. Sin embargo las choices posibles dependerán de cada deporte. El fútbol tendrá ataque, defensa, portero, etc. El béisbol tendrá un pitcher pero no tendrá portero. Necesitarás las opciones correctas para pasar la validación.

Las quedadas se pasan como un entity field al formulario:

```
// src/AppBundle/Form/Type/SportMeetupType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\FormEvent;
use Symfony\Component\Form\FormEvents;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
// ...

class SportMeetupType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('sport', EntityType::class, array(
                'class'       => 'AppBundle:Sport',
                'placeholder' => '',
            ))
        ;

        $builder->addEventListener(
            FormEvents::PRE_SET_DATA,
            function (FormEvent $event) {
                $form = $event->getForm();

                // esto sería tu entidad, por ejemplo SportMeetup
                $data = $event->getData();

                $sport = $data->getSport();
                $positions = null === $sport ? array() : $sport->getAvailablePositions();

                $form->add('position', EntityType::class, array(
                    'class'       => 'AppBundle:Position',
                    'placeholder' => '',
                    'choices'     => $positions,
                ));
            }
        );
    }

    // ...
}
```

Cuando construyes el formulario para mostrarlo al usuario por primera vez el ejemplo funciona bien.

Pero la cosa cambia cuando manejas el formulario enviado. El evento PRE_SET_DATA nos dice los datos con los que empiezas (por ejemplo, un objeto SportMeetup), no los datos enviados.

En un formulario, podemos normalmente atender a los siguientes eventos: PRE_SET_DATA, POST_SET_DATA, PRE_SUBMIT, SUBMIT, POST_SUBMIT. 

La clave está en **añadir un listener POST_SUBMIT** al campo del que depende tu nuevo campo. Si añades un listener POST_SUBMIT a un formulario hijo (por ejemplo, _sport_), y añades nuevos hijos al formulario padre, el componente Form detectará el nuevo campo automáticamente y lo mapeará a los datos proporcionados por el usuario.

El type quedaría así:

```
// src/AppBundle/Form/Type/SportMeetupType.php
namespace AppBundle\Form\Type;

// ...
use Symfony\Component\Form\FormInterface;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use AppBundle\Entity\Sport;

class SportMeetupType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('sport', EntityType::class, array(
                'class'       => 'AppBundle:Sport',
                'placeholder' => '',
            ));
        ;

        $formModifier = function (FormInterface $form, Sport $sport = null) {
            $positions = null === $sport ? array() : $sport->getAvailablePositions();

            $form->add('position', EntityType::class, array(
                'class'       => 'AppBundle:Position',
                'placeholder' => '',
                'choices'     => $positions,
            ));
        };

        $builder->addEventListener(
            FormEvents::PRE_SET_DATA,
            function (FormEvent $event) use ($formModifier) {
                // esto sería tu entidad, por ejemplo: SportMeetup
                $data = $event->getData();

                $formModifier($event->getForm(), $data->getSport());
            }
        );

        $builder->get('sport')->addEventListener(
            FormEvents::POST_SUBMIT,
            function (FormEvent $event) use ($formModifier) {
                // Es importante aquí obtener datos así $event->getForm()->getData(), ya que
                // $event->getData() obtendrá los datos del cliente (esto es, el ID)
                $sport = $event->getForm()->getData();

                // ya que hemos añadido el listener al hijo, tendremos que informar del
                // al padre a las funciones callback
                $formModifier($event->getForm()->getParent(), $sport);
            }
        );
    }

    // ...
}
```

Se puede ver que necesitas atender a los dos **eventos** y tener diferentes **callbacks** sólo porque en dos escenarios diferentes, los datos que puedes usar están disponibles en eventos diferentes. Además, los listeners siempre realizan las mismas cosas en un formulario concreto.

Una pieza que se nos escapa es la actualización por el lado del cliente del formulario después de que se haya seleccionado el deporte. Esto debería hacerse con una llamada a AJAX en la aplicación. Si tenemos el siguiente controller para crear las quedadas:

```
// src/AppBundle/Controller/MeetupController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use AppBundle\Entity\SportMeetup;
use AppBundle\Form\Type\SportMeetupType;
// ...

class MeetupController extends Controller
{
    public function createAction(Request $request)
    {
        $meetup = new SportMeetup();
        $form = $this->createForm(SportMeetupType::class, $meetup);
        $form->handleRequest($request);
        if ($form->isValid()) {
            // ... save the meetup, redirect etc.
        }

        return $this->render(
            'AppBundle:Meetup:create.html.twig',
            array('form' => $form->createView())
        );
    }

    //
```

La template asociada utiliza **JavaScript** para actualizar la _position_ del campo del formulario en función de la selección actual en el campo _sport_:

```
{# app/Resources/views/Meetup/create.html.twig #}
{{ form_start(form) }}
    {{ form_row(form.sport) }}    {# <select id="meetup_sport" ... #}
    {{ form_row(form.position) }} {# <select id="meetup_position" ... #}
    {# ... #}
{{ form_end(form) }}

<script>
var $sport = $('#meetup_sport');
// When sport gets selected ...
$sport.change(function() {
  // ... Obtener el formulario.
  var $form = $(this).closest('form');
  // Datos del formulario, pero sólo incluir el valor del deporte seleccionado.
  var data = {};
  data[$sport.attr('name')] = $sport.val();
  // Enviar los datos con AJAX al directorio de acción del formulario.
  $.ajax({
    url : $form.attr('action'),
    type: $form.attr('method'),
    data : data,
    success: function(html) {
      // Reemplazar la posición actual ...
      $('#meetup_position').replaceWith(
        // ... con la devuelta por la respuesta AJAX.
        $(html).find('#meetup_position')
      );
      // Position field ya muestra las posiciones correctas.
    }
  });
});
</script>
```

La principal diferencia entre enviar el formulario entero o sólo extraer el campo position actualizado es que no se necesita código adicional del lado del cliente. Todo el código generado puede reusarse.

### 4\. Eliminar la validación de formularios

Para **eliminar la validación de formularios** puedes usar el evento POST_SUBMIT y evitar llamar al **ValidationListener**.

La razón por la que se puede necesitar hacer esto es porque incluso si estableces los _validation_groups_ a _false_ todavía se ejecutan comprobaciones. Por ejemplo un archivo subido será comprobado igualmente para ver si es demasiado grande y el formulario todavía comprobará si se han enviado campos no existentes. Para desactivar todo, puedes emplear un listener:

```
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\FormEvents;
use Symfony\Component\Form\FormEvent;

public function buildForm(FormBuilderInterface $builder, array $options)
{
    $builder->addEventListener(FormEvents::POST_SUBMIT, function (FormEvent $event) {
        $event->stopPropagation();
    }, 900); // Siempre establece una prioridad mayor que ValidationListener

    // ...
}
```

Haciendo esto puedes accidentalmente desactivar algo más que la **validación de formularios**, ya que el **evento POST_SUBMIT** puede tener otros **listeners**.