**Symfony** viene con campos de formulario incorporados para **crear formularios**, pero hay veces que puede ser necesario tener que crear un _custom form field type_. 

Para crear un _field type_ customizado primero hay que crear la clase que representa el campo. En este caso vamos a crear un campo género, cuya clase será **GenderType**, y el archivo se guardará en el directorio por defecto para campos de formulario: _<bundleName>\Form\Type_.

```
// src/AppBundle/Form/Type/GenderType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;

class GenderType extends AbstractType
{
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'choices' => array(
                'm' => 'Male',
                'f' => 'Female',
            )
        ));
    }

    public function getParent()
    {
        return ChoiceType::class;
    }
}
```

El valor de retorno del método _getParent_ indica que estás extendiendo al campo **ChoiceType**. Esto significa que por defecto heredas toda la lógica y renderización de ese field type. La lógica de la clase se puede ver desde [aquí](https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Form/Extension/Core/Type/ChoiceType.php). Hay tres métodos particularmente importantes:

*   _buildForm()_. Cada field type tiene un método _buildForm_, que es donde se configuran y construyen los campos. Es el mismo método que se emplea para configurar los formularios, y funciona de la misma forma.
*   _buildView()_. Este método se usa para establecer cualquier variable extra que necesites cuando renderices el campo en una template. Por ejemplo, en ChiceType, se establece una variable _multiple_ y se emplea en la template para establecer (o no) el atributo _multiple_ en el campo select. 
*   _configureOptions()_. Define opciones para tu form type que pueden usarse en _buildForm()_ y _buildView()_. Hay muchas [opciones comunes a todos los campos](http://symfony.com/doc/current/reference/forms/types/form.html), pero puedes crear otras que necesites en este método.

Si estás creando un campo que consiste en varios campos, asegúrate de establecer el _parent type_ en **form** o cualquiera que extienda a **form**. También, si necesitas modificar el _view_ de cualquiera de los _child types_ desde el _parent type_, puedes emplear el método _finishView()_. 

El objetivo de este campo en concreto es extender a choice type para permitir la selección de género. Esto se consigue modificando las _choices_ para listar los géneros.

### Crear una template para el campo

Cada field type es renderizado por un fragmento de template, que es determinado en parte por el nombre de clase del type.

La primera parte del prefijo (en este caso _gender_) viene del nombre de clase (_GenderType -> gender_). Esto puede modificarse sobreescribiendo _getBlockPrefix()_ en **GenderType**.

En este caso, ya que el parent field es **ChoiceType**, no necesitas hacer nada ya que el campo customizado será automáticamente renderizado como un ChoiceType. Pero en este caso cuando el campo se expanda (por ejemplo radio buttons o checkboxes, en lugar de un select field), queremos renderizarlo en un elemento _ul_. En la [form theme template](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-customization-form-themes), crea un bloque **gender_widget** para modificarlo:

```
{# app/Resources/views/Form/fields.html.twig #}
{% block gender_widget %}
    {% spaceless %}
        {% if expanded %}
            <ul {{ block('widget_container_attributes') }}>
            {% for child in form %}
                <li>
                    {{ form_widget(child) }}
                    {{ form_label(child) }}
                </li>
            {% endfor %}
            </ul>
        {% else %}
            {# dejar que el widget choice renderice la etiqueta select #}
            {{ block('choice_widget') }}
        {% endif %}
    {% endspaceless %}
{% endblock %}
```

Hay que asegurarse de que se emplea el **prefijo widget** correcto. En este ejemplo el nombre debería ser [gender_widget](http://symfony.com/doc/current/cookbook/form/form_customization.html#cookbook-form-customization-form-themes). Además el archivo de configuración principal debe apuntar al custom form type para que se emplee cuando se renderiza cualquier formulario.

Si se usa Twig se incluye lo siguiente:

```
# app/config/config.yml
twig:
    form_themes:
        - 'form/fields.html.twig'
```

### Utilizar el field type

Ahora ya podemos emplear el custom field type, simplemente creando una nueva instancia del type en uno de nuestros formularios:

```
// src/AppBundle/Form/Type/AuthorType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use AppBundle\Form\Type\GenderType;

class AuthorType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('gender_code', GenderType::class, array(
            'placeholder' => 'Choose a gender',
        ));
    }
}
```

Este ejemplo es realmente muy sencillo, pero si queremos obtener los **gender codes** desde un archivo de configuración o de una base de datos, tenemos que emplear _services_.

### Crear el field type como service

En el mismo ejemplo, vamos a a guardar los parámetros del género en la configuración:

```
# app/config/config.yml
parameters:
    genders:
        m: Male
        f: Female
```

Para emplear el parámetro, definimos el custom field type como un _service_, inyectando el valor del parámetro genders como primer argumento del método **__construct**:

```
# src/AppBundle/Resources/config/services.yml
services:
    app.form.type.gender:
        class: AppBundle\Form\Type\GenderType
        arguments:
            - '%genders%'
        tags:
            - { name: form.type }
```

Añadimos el **método constructor** a **GenterType**, que recibe la configuración gender:

```
// src/AppBundle/Form/Type/GenderType.php
namespace AppBundle\Form\Type;

use Symfony\Component\OptionsResolver\OptionsResolver;

// ...

// ...
class GenderType extends AbstractType
{
    private $genderChoices;

    public function __construct(array $genderChoices)
    {
        $this->genderChoices = $genderChoices;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'choices' => $this->genderChoices,
        ));
    }

    // ...
}
```

El **GenderType** recibe ahora los parámetros de configuración y se ha registrado como service. Ya que hemos usado el alias form.type en la configuración, se usará el service en lugar de crear un nuevo GenderType. En otras palabras, el controller no tiene por qué cambiar, sigue siendo así:

```
// src/AppBundle/Form/Type/AuthorType.php
namespace AppBundle\Form\Type;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use AppBundle\Form\Type\GenderType;

class AuthorType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder->add('gender_code', GenderType::class, array(
            'placeholder' => 'Choose a gender',
        ));
    }
}
```