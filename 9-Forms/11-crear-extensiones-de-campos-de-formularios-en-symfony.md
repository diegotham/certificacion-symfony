Los [campos de formulario customizados](http://diego.com.es/crear-un-tipo-de-campo-de-formulario-en-symfony) están muy bien cuando necesitamos _field types_ con un objetivo específico, pero en ocasiones no necesitamos tener que crear uno nuevo, sino que podemos **extender** los ya existentes.

Las **extensiones de campos de formularios** tienen 2 usos principales:

1.  Añadir una **funcionalidad especial** a un type (como añadir una funcionalidad "download" a un field type FileType).
2.  Añadir una **funciolidad genérica** a varios types (añadir texto de ayuda a cada type "input text").

Puede ser posible conseguir lo que quieres con los [_field types_ ya existentes](http://diego.com.es/campos-de-formulario-en-symfony), pero con las extensiones puede quedar **más limpio** (limitando la business logic en las templates) y **más flexible** (puedes añadir varias _type extensions_ a un form type).

Las **extensiones de form types** pueden hacer lo que la mayoría de field types customizados pueden hacer, pero en lugar de ser field types por sí mismos, se pegan a los ya existentes. 

Por ejemplo tenemos una entidad **Media **y cada media está asociada a un **archivo**. Tu formulario Media utiliza un _file type_, pero cuando se modifique la entidad, te gustaría ver su imagen automáticamente renderizada con el archivo. Esto se puede hacer customizándolo en la template, pero con las **extensiones field type** se puede hacer respetando el **DRY** (Don't Repeat Yourself).

### Definir el Form Type Extension

La primera tarea será crear la clase de la extensión del form type (en este caso será ImageTypeExtension). Por defecto, las extensiones de formulario normalmente están en el directorio _Form\Extension_ del los bundles.

Cuando creamos una extensión form type, podemos implementar la interface FormTypeExtensionInterface o extender la clase **AbstractTypeExtension**. En la mayoría de los casos, es más fácil extender la [clase abstract](http://diego.com.es/clases-abstractas-en-php):

```
// src/AppBundle/Form/Extension/ImageTypeExtension.php
namespace AppBundle\Form\Extension;

use Symfony\Component\Form\AbstractTypeExtension;
use Symfony\Component\Form\Extension\Core\Type\FileType;

class ImageTypeExtension extends AbstractTypeExtension
{
    /**
     * Returns the name of the type being extended.
     *
     * @return string The name of the type being extended
     */
    public function getExtendedType()
    {
        return FileType::class;
    }
}
```

El único método que debemos implementar es _getExtendedType_. Se emplea para indicar el nombre del form type que será extendido por tu extensión. El valor de retorno del método getExtendedType corresponde al nombre de clase fully qualified de la clase form type que quieres extender.

Además de la función _getExtendedType_, probablemente quieras sobreescribir uno de los siguientes métodos: _buildForm_, _buildview_, _configureOptions_, _finishView_.

### Registrar la extensión Form Type como un Service

El siguiente paso es notificar a **Symfony** sobre la nueva extensión. Todo lo que tenemos que hacer es declararla como un _service_ empleando la tag _form.type_extension_:

```
services:
    app.image_type_extension:
        class: AppBundle\Form\Extension\ImageTypeExtension
        tags:
            - { name: form.type_extension, extended_type: Symfony\Component\Form\Extension\Core\Type\FileType }
```

La key _extended_type_ de la tag es el tipo de campo al que esta extensión se aplicará. En nuestro caso queremos extender el field type _Symfony\Component\Form\Extension\Core\Type\FileType_.

### Añadir la extension Business Logic

El objetivo de la extensión es mostrar imágenes al lado de los _file inputs_. Para ese objetivo, queremos usar un enfoque similar al descrito en [Subida de archivos en Doctrine](http://diego.com.es/subida-de-archivos-en-doctrine): tenemos un model **Media** con una propiedad _file_ (correspondiente al campo _file_ en el formulario) y una propiedad _path_ (correspondiente al image path en la base de datos):

```
// src/AppBundle/Entity/Media.php
namespace AppBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;

class Media
{
    // ...

    /**
     * @var string The path - typically stored in the database
     */
    private $path;

    /**
     * @var \Symfony\Component\HttpFoundation\File\UploadedFile
     * @Assert\File(maxSize="2M")
     */
    public $file;

    // ...

    /**
     * Get the image URL
     *
     * @return null|string
     */
    public function getWebPath()
    {
        // ... $webPath es la URL completa de la imagen, para usar en las templates

        return $webPath;
    }
}
```

La clase form type extension necesitará hacer un par de cosas para extender el form type FileType::class:

1.  Sobreescribir el método configureOptions para añadir una opción image_path.
2.  Sobreescribir los métodos buildForm y buildView para pasar la URL de la imagen a la view.

La lógica es la siguiente: cuando se añade un campo de formulario **FileType::class**, podrás especificar una nueva opción: _image_path_. Esta opción le dirá al campo _file_ como obtener la **URL de la imagen** para mostrarla en la view:

```
// src/AppBundle/Form/Extension/ImageTypeExtension.php
namespace AppBundle\Form\Extension;

use Symfony\Component\Form\AbstractTypeExtension;
use Symfony\Component\Form\FormView;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\PropertyAccess\PropertyAccess;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\FileType;

class ImageTypeExtension extends AbstractTypeExtension
{
    /**
     * Returns the name of the type being extended.
     *
     * @return string The name of the type being extended
     */
    public function getExtendedType()
    {
        return FileType::class;
    }

    /**
     * Add the image_path option
     *
     * @param OptionsResolver $resolver
     */
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefined(array('image_path'));
    }

    /**
     * Pass the image URL to the view
     *
     * @param FormView $view
     * @param FormInterface $form
     * @param array $options
     */
    public function buildView(FormView $view, FormInterface $form, array $options)
    {
        if (array_key_exists('image_path', $options)) {
            $parentData = $form->getParent()->getData();

            if (null !== $parentData) {
                $accessor = PropertyAccess::createPropertyAccessor();
                $imageUrl = $accessor->getValue($parentData, $options['image_path']);
            } else {
                 $imageUrl = null;
            }

            // establecemos una variable "image_url" que estará disponible cuando se renderice el campo
            $view->vars['image_url'] = $imageUrl;
        }
    }

}
```

### Sobreescribir el File Widget Template Fragment

Cada _field type_ se renderiza con un fragmento de template. Esos fragmentos de template se pueden sobreescribir para customizar el renderizado de formularios ([Form Theming](http://diego.com.es/form-theming-en-symfony)). 

En la clase extension, hemos añadido una nueva variable _image_url_, pero todavía necesitas aprovechar esta nueva variable en las tempaltes. Debemos sobreescribir el bloque _file_widget_:

```
{# src/AppBundle/Resources/views/Form/fields.html.twig #}
{% extends 'form_div_layout.html.twig' %}

{% block file_widget %}
    {% spaceless %}

    {{ block('form_widget') }}
    {% if image_url is not null %}
        <img src="{{ asset(image_url) }}"/>
    {% endif %}

    {% endspaceless %}
{% endblock %}
```

Necesitarás cambiar tu **archivo de configuración** o especificar explícitamente como quieres que sea diseñado tu formulario para que **Symfony** emplee el bloque que sobreescribe (**Form theming**).

### Usar la extensión Form Type

Desde ahora, cuando añadamos un _field type_ del tipo **FileType::class** en el formulario, puedes especificar una opción _image_path_ que se usará para mostrar una imagen al lado del campo file. Por ejemplo:

```
// src/AppBundle/Form/Type/MediaType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\Extension\Core\Type\FileType;

class MediaType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('name', TextType::class)
            ->add('file', FileType::class, array('image_path' => 'webPath'));
    }
}
```

Cuando se muestre el formulario, si el modelo ya se ha asociado con una imagen, la verás mostrada junto al file input.

### Extensiones Form Type genéricas

Puedes modificar varios form types de vez especificando un parent común. Por ejemplo, algunos form types disponibles por defecto en **Symfony** heredan de **TextType** (como email, SearchType, etc). Una extensión form type bajo TextType (cuyo método _getExtendedType_ devuelve **TextType::class**) se aplica en todos estos form types.

De la misma forma, ya que la mayoría de form types de Symfony heredan de **FormType**, una extensión form type bajo FormType se aplica a todos. Una excepción son los ButtonType (no heredan de FormType). Tanto FormType como ButtonType también pueden extenderse.