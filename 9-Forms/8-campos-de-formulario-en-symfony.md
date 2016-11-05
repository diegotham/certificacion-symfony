**Symfony** viene con un amplio número de _field types_ que cubren la mayoría de campos de formulario que se pueden dar, divididos en 8 grupos:

1\. **Base Fields** - FormType

2\. **Text Fields** - TextType, TextareaType, EmailType, IntegerType, MoneyType, NumberType, PasswordType, PercentType, SearchType, UrlType, RangeType

3\. **Choice Fields** - ChoiceType, EntityType, CountryType, LanguageType, LocaleType, TimezoneType, CurrencyType

4\. **Date and Time Fields** - DateType, DateTimeType, TimeType, BirthdayType

5\. **Other fields** - CheckboxType, FileType, RadioType

6\. **Field Groups** - CollectionType, RepeatedType

7\. **Hidden Fields** - HiddenType

8\. **Buttons** - ButtonType, ResetType, SubmitType

## 1\. Base Fields

### FormType

#### Class: _FormType_

**FormType** predefine las opciones disponibles para el resto de types, de los cuales FormType es el parent.

#### Opciones

*   **action**. Type: _string_. Default. _empty string_. Esta opción especifica dónde enviar los datos del formulario (si no se especifica se supone el mismo documento de referencia).
*   **allow_extra_fields**. Type: _boolean_. Default: _false_. Si envías campos extra que no están configurados en el formulario obtendrás un error de validación. Puedes silenciar este error de validación con esta opción en true.
*   **by_reference**. Type: _boolean_. Default: _true_. En la mayoría de los casos si hay un campo name, se espera que se llame a _setName()_ en el objeto. En algunos casos puede que no sea llamado, por lo que esta opción asegura que el setter se llama en todos los casos. Ejemplo de uso:

```
$builder = $this->createFormBuilder($article);
$builder
    ->add('title', TextType::class)
    ->add(
        $builder->create('author', FormType::class, array('by_reference' => ?))
            ->add('name', TextType::class)
            ->add('email', EmailType::class)
    )
```

Lo que hace Symfony cuando se especifica esta opción es lo siguiente:

```
$article->setTitle('...');
$article->getAuthor()->setName('...');
$article->getAuthor()->setEmail('...');
```

Observa que no se llama a _setAuthor()_. El author se modifica por referencia.

Si especificas _by_reference_ a **false**, el envío resulta así:

```
$article->setTitle('...');
$author = $article->getAuthor();
$author->setName('...');
$author->setEmail('...');
$article->setAuthor($author);
```

Por lo que fuerza al framework a llamar al setter del objeto padre.

Igualmente, si empleas un campo **CollectionType** donde la colección de datos es un objeto (como el **ArrayCollection** de **Doctrine**), entonces _by_reference_ debe ser false si necesitas llamar al _adder_ y al _remover_ (_addAuthor()_ y _removeAuthor()_).

*   **compound**. Type: _boolean_. Default: _true_. Esta opción especifica si el formulario es **compuesto**. Esto es independiente de si el formulario tiene hijos. Un formulario puede se compuesto pero no tener hijos (por ejemplo un collection form vacío). 
*   **constraints**. Type: _array_ or _Constraint_. Default: _null. _Permite añadir una o más constraints de validación a un campo específico. 
*   **data**. Type: _mixed_. Default: _Depende del campo del objeto_. Cuando creas un formulario, cada campo inicialmente muestra el valor de la propiedad correspondiente del objeto ligado al formulario. Si quieres sobreescribir el valor inicial del formulario, puedes establecer esta opción.

```
use Symfony\Component\Form\Extension\Core\Type\HiddenType;
// ...

$builder->add('token', HiddenType::class, array(
    'data' => 'abcdef',
));
```

*   **data_class**. Type: _string_. Esta opción se emplea para establecer el mapeador de datos a usar en el formulario.

```
use AppBundle\Form\MediaType;
// ...

$builder->add('media', MediaType::class, array(
    'data_class' => 'Acme\DemoBundle\Entity\Media',
));
```

*   **empty_data**. Type: _mixed_. Establece el valor que devolverá el campo cuando el valor esté vacío. Puede tomar varios valores en función de las demás opciones:
    *   Si está establecida _data_class_ y _required_ es **true**: _new $data_class()_.
    *   Si está establecida _data_class_ y _required_ es **false**: _null_.
    *   Si no está establecida _data_class_ y _compound_ es **true**: _array()_ (array vacío).
    *   Si no está establecida _data_class_ y _compound_ es **false**: _''_ (string vacío).

Pero puedes customizarlo con tus propias opciones. Si por ejemplo tienes un campo de elección de género que quieres establecer a _null_ si no se proporciona:

```
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
// ...

$builder->add('gender', ChoiceType::class, array(
    'choices' => array(
        'm' => 'Male',
        'f' => 'Female'
    ),
    'required'    => false,
    'placeholder' => 'Choose your gender',
    'empty_data'  => null
));
```

*   **error_bubbling**. Type: _boolean_. Default: _false_ a no ser que el fomulario sea compound. Si es true, cualquier error en este campo se pasarán al parent field o al parent form. Por ejemplo si es true en un campo normal, cualquier error en ese campo se pasará al formulario principal, no al campo específico. 
*   [**error_mapping**](http://symfony.com/doc/current/reference/forms/types/form.html#error-mapping). Type: _array_. Default: _array()_. Permite modificar el objetivo de un error de validación, de forma que por ejemplo puedan mostrarse errores de validación que comprueben las relaciones entre campos (por ejemplo con un método que compruebe que concuerda el código postal con la ciudad).
*   **extra_fields_message**. Type: _string_. Default: _This form should not contain extra fields_. Este es el mensaje de error de validación que se usa si los datos del formulario enviado contiene uno o más campos que no son parte de la definición del formulario.
*   **inherit_data**. Type: _boolean_. Default: _false_. Esta opción determina si el formulario heredará datos de su **parent form**. Esto puede resultar útil si tienes un conjunto de campos duplicados en varios formularios. Cuando un campo tiene la opción _inherit_data_ establecida, utiliza los datos del parent form tal cual, por lo que no se aplican **Data Transformers**.
*   **invalid_message**. Type: _string_. Default: _This value is not valid_. Este es el mensaje de error de validación que se usa si los datos introducidos en el campo no tienen sentido (por ejemplo que la validación falla). 
*   **invalid_message_parameters**. Type: _array_. Default: _array()_. Cuando se establece la opción invalid_message, podrías necesitar incluid variables en el string. Esto se puede hacer añadiendo placeholders a esa opción e incluyendo las variables en esta opción:

```
$builder->add('some_field', SomeFormType::class, array(
    // ...
    'invalid_message' => 'Valor inválido, debería incluir %num% letras',
    'invalid_message_parameters' => array('%num%' => 6),
));
```

*   **label_attr**. Type: _array_. Default: _array()_. Establece los **atributos HTML** para el elemento _<label>_, que se emplearán cuando se renderice la etiqueta para el campo. Es un array asociativo con atributo HTML como key. Estos atributos pueden establecerse directamente dentro de la template.

```
{{ form_label(form.name, 'Your name', {
       'label_attr': {'class': 'CUSTOM_LABEL_CLASS'}
}) }}
```

*   **mapped**. Type: _boolean_. Default: _true_. Si quieres que el campo sea ignorado cuando se lea o escriba en el objeto, puedes establecer esta opción a false.
*   **method**. Type: _string_. Default: _POST_. Especifica el [método HTTP](http://diego.com.es/metodos-http) empleado para enivar los datos. Su valor se renderiza como el atributo _method_ del elemento _form_ y se usa para decidir si procesar el envío del formulario en el método _handleRequest()_ después del envío. Valores posibles: POST, GET, PUT, DELETE, PATCH.
    Cuando el método es PUT, PATCH o DELETE **Symfony** incluye automáticamente un campo _hidden_ en el formulatio. Esto se hace para falsear estos métodos, ya que no están soportados por los navegadores.
    El método PATCH permite enviar datos parciales, por lo que si parte de los datos no se facilitan, se ignorarán y se mantendrán los que estaban antes. Con los otros métodos HTTP si no se proporcionan algunos campos se establecerán como _null_. 
*   **pos_max_size_message**. Type: _string_. Default: _The uploaded file was too large. Please try to upload a smaller file_. Este es el mensaje de error de validación que se usa si el envío de datos en POST excede de la directiva _php.ini_ _post_max_size_. El placeholder {{ max }} puede usarse para mostrar el tamaño permitido.
*   **property_path**. Type: _any_. Default: _el nombre del campo_. Los campos muestran un valor de propiedad del objeto del formulario al que está ligado. Cuando se envía el formulario, el valor enviado se guarda en el objeto. Si quieres sobreescribir la propiedad a la que ese campo está ligado, puedes establecer esta opción.
*   **required**. Type: _boolean_. Default: _true_. Si es true, se renderizará como un [campo requerido de HTML5](http://diveintohtml5.info/forms.html). 
*   **trim**. Type: _boolean_.  Default: _true_. Si es true, los espacios en blanco del string que se envíe se eliminarán con la [función trim de php](http://php.net/manual/en/function.trim.php). 

#### Opciones heredadas

Estas opciones se definen en la clase **BaseType**. Esta clase es la clase parent para **FormType** y **ButtonType**.

*   **attr**. Type: _array_. Default: _array()_. Si quieres **añadir atributos extra a un campo HTML** puedes emplear la opción _attr_. Es un array asociativo con atributos HTML como keys. Puede ser útil cuando se quiera establecer una clase personalizada para algún widget:

```
$builder->add('body', TextareaType::class, array(
    'attr' => array('class' => 'tinymce'),
));
```

*   **auto_initialize**. Type: _boolean_. Default: _true_. Establece si el formulario debe iniciarse automáticamente. Para todos los campos esta opción debería ser sólo true para formularios root.
*   **block_name**. Type: _string_. Default: _el nombre del formulario_. Permite sobreescribir el nombre del bloque utilizado para renderizar el form type. Útil por ejemplo si tienes múltiples instancias del mismo formulario y necesitas personalizar el renderizado de los formularios individualmente.
*   **disabled**. Type: _boolean_. Default: _false_. Si no quieres que un usuario modifique el valor de un campo, puedes establecer esta opción a true.
*   **label**. Type: _string_. Default: _El campo es "supuesto" del nombre del campo_. Establece la _label_ que se empleará cuando se renderice el campo. Si se establece en false, eliminará la _label_. Esta también puede establecerse en la template:

```
{{ form_label(form.name, 'Your name') }}
```

*   **translation_domain**. Type: _string_. Default: _messages_. Este es el dominio de traducción que se usará para las labels y opciones que se rendericen en este campo.

## 2\. Text Fields

### TextType

#### Class: _TextType_

#### Opciones heredadas

data, disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required, trim.

#### Opciones sobreescritas

*   **compound**. Type: _boolean_. Default: _false_.

### TextAreaType

#### Class: _TextAreaType_

#### Opciones heredadas

attr, data, disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required, trim

### EmailType

#### Class: _EmailType_

#### Opciones heredadas

data, disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required, trim

### IntegerType

#### Class: _IntegerType_

#### Opciones

*   **grouping**. Type: _integer_. Default: _false_. Este valor se usa internamente como el valor NumberFormatter::GROUPING_USED utilizado en la **clase de PHP NumberFormatter**. Si es true, los números se formarán dependiendo del locale: 12345.123 será 12,345.123.
*   **scale**. Type: _integer_. Default: _locale-specific_ (normalmente alrededor de 3). Especifica cuantos decimales se permiten hasta que se redondee el valor enviado. Por ejemplo de 20.12345 pasará a 20.12 si está establecido _scale_ en 2.
*   **rounding_mode**. Type: _integer_. Default: _IntegerToLocalizedStringTransformer::ROUND_DOWN_. Por defecto, si un usuario introduce un número no integer, se redondeará hacia abajo. Existen otros métodos posibles: ROUND_FLOOR, ROUND_UP, ROUND_CEILING, ROUND_HALF_DOWN, ROUND_HALF_EVEN, ROUND_HALF_UP.

#### Opciones sobreescritas

*   **compound**. Type: _boolean_. Default: _false_.

#### Opciones heredadas

data, disabled, empty_data, error_bubbling, error_mapping, invalid_message, invalid_message_parameters, label, label_attr, mapped, required

### MoneyType

#### Class: _MoneyType_

#### Opciones

*   **currency**. Type: _string_. Default: _EUR_. Especifica la moneda en que se especifique el dinero. Determina el símbolo que se muestre en la caja de texto.
*   **divisor**. Type: integer. Default: 1\. Si por alguna razón necesitas dividir el valor inicial por un número antes de renderizarlo puedes usar esta opción.

```
use Symfony\Component\Form\Extension\Core\Type\MoneyType;
// ...

$builder->add('price', MoneyType::class, array(
    'divisor' => 100,
));
```

En este caso si el campo _price_ se establece en 9900, el valor 99 se mostrará al usuario. Cuando el usuario envíe el valor 99, se multiplicará por 100 y 9900 se enviará al objeto.

*   **grouping**. Type: _integer_. Default: _false_. Mismo significado que en IntegerType.
*   **scale**. Type: _integer_. Default: _2_. Si necesitas escalar a un valor diferente a 2, puedes especificar esta opción.

#### Opciones sobreescritas

*   **compound**. Type: _boolean_. Default: _false_.

#### Opciones heredadas

data, disabled, empty_data, error_bubbling, error_mapping, invalid_message, invalid_message_parameters, label_label_attr, mapped, required

### NumberType

#### Class: _NumberType_

#### Opciones

*   **grouping**. Type: _integer_. Default: _false_. Igual que IntegerType.
*   **scale**. Type: _integer_. Default: _Locale-specific_ (normalmente alrededor de 3). Igual que IntegerType.
*   **rounding_mode**. Type: _integer_. Default: _NumberToLocalizedStringTransformer::ROUND_HALFUP_. Igual que IntegerType.

#### Opciones sobreescritas

*   **compound**. Type: _boolean_. Default: _false_.

#### Opciones heredadas

data, disabled, empty_data, error_bubbling, error_mapping, invalid_message, invalid_message_parameters, label, label_attr, mapped, required

### PasswordType

#### Class: _PasswordType_

#### Opciones

*   **always_empty**. Type: _boolean_. Default: _true_. Si es true el campo siempre se mostrará vacío, incluso si el valor correspondiente tiene un valor. Si es false el campo de password se renderizará con el atributo _value_ establecido con su valor.

#### Opciones sobreescritas

*   **trim**. Type: _boolean_. Default: _false_.

#### Opciones heredadas

disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required

### PercentType

#### Class: _PercentType_

#### Opciones

*   **scale**. Type: _integer_. Default: _0_. Por defecto los números se redondean. Para permitir decimales, cambia el valor.
*   **type**. Type: _string_. Default: _fractional_. Controla cómo los datos se guardan en tu objeto. Por ejemplo un porcentaje del 55% puede guardarse como .55 o 55 en tu objeto. Los dos casos posibles:
    *   fractional. Para guardar los datos como decimal (.55). El dato se multiplicará por 100 antes de mostrarse al usuario.
    *   integer. Para guardar los datos como decimal (55).

#### Opciones sobreescritas

*   **compound**. Type: _boolean_. Default: _false_.

#### Opciones heredadas

data, disabled, empty_data, error_bubbling, error_mapping, invalid_message, invalid_message_parameters, label, label_attr, mapped, required

### SearchType

#### Class: SearchType

#### Opciones heredadas

disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required, trim

### UrlType

#### Class: UrlType

#### Opciones

*   **default_protocol**. Type: _string_. Default: _http_. Si se inserta un valor que no comienza con ningún protocolo (http://, ftp://, etc) este protocolo se pondrá al principio del string cuando se envíen los datos.

#### Opciones heredadas

data, disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required, trim

### RangeType

#### Class: RageType

#### Opciones heredadas

attr, data, disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required, trim

#### Uso básico

```
use Symfony\Component\Form\Extension\Core\Type\RangeType;
// ...

$builder->add('name', RangeType::class, array(
    'attr' => array(
        'min' => 5,
        'max' => 50
    )
));
```

## 3\. Choice Fields

### ChoiceType

#### Class: ChoiceType

Un campo que permite al usuario elegir entre una o más opciones. Puede renderizarse como una etiqueta _select_, _radio buttons_ o _checkboxes._ Para emplear este campo hay que especificar obligatoriamente la opción _choices_ o _choice_loader_.

#### Ejemplo de uso

La forma más fácil de emplear este campo es especificar las choices directamente a través de la opción choices:

```
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
// ...

$builder->add('isAttending', ChoiceType::class, array(
    'choices'  => array(
        'Maybe' => null,
        'Yes' => true,
        'No' => false,
    ),
    // *esta línea es importante antes de Symfony 3*
    'choices_as_values' => true,
));
```

Esto creará un drop-down _select_.

Si el usuario selecciona No, el formulario devolverá false para este campo. Igualmente para las otras dos _keys_ con sus respectivos _values_. El _key_ es lo que se muestra al usuario y el _value_ lo que se quiere obtener/establecer en el código PHP.

La etiqueta _choice_as_values_ debe aplicarse siempre en versiones anteriores a Symfony 3 (por un tema de versiones del API de Symfony).

#### Ejemplo avanzado

Este campo tiene muchas opciones y la mayoría controlan cómo se mostrará el campo. En este ejemplo, los datos están en un objeto Category que tiene un método _getName()_. 

```
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
use AppBundle\Entity\Category;
// ...

$builder->add('category', ChoiceType::class, [
    'choices' => [
        new Category('Cat1'),
        new Category('Cat2'),
        new Category('Cat3'),
        new Category('Cat4'),
    ],
    'choices_as_values' => true,
    'choice_label' => function($category, $key, $index) {
        /** @var Category $category */
        return strtoupper($category->getName());
    },
    'choice_attr' => function($category, $key, $index) {
        return ['class' => 'category_'.strtolower($category->getName())];
    },
    'group_by' => function($category, $key, $index) {
        // asigna de forma aleatoria en dos grupos
        return rand(0, 1) == 1 ? 'Group A' : 'Group B'
    },
    'preferred_choices' => function($category, $key, $index) {
        return $category->getName() == 'Cat2' || $category->getName() == 'Cat3';
    },
]);
```

Puedes también establecer el _choice_name_ y _choice_value_ de cada choice para mayor customización HTML. También puedes cambiar cada texto de opción (label) con la opción _choice_label_. 

#### Etiqueta select, Checkboxes o Radio Buttons

Este campo puede renderizarse como diferentes **campos de HTML**, dependiendo de las opciones _expanded_ y _multiple_:

| | | |
| -------- | -------- |
| **Element Type** | **Expanded** | **Multiple** |
| select tag | false | false |
| select tag (con atributo _múltiple_) | false | true |
| radio buttons | true | false |
| checkboxes | true | true |

#### Opciones de agrupación

Puedes agrupar opciones pasando un array multitdimensional:

```
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
// ...

$builder->add('stockStatus', ChoiceType::class, [
    'choices' => [
        'Main Statuses' => [
            'Yes' => 'stock_yes',
            'No' => 'stock_no',
        ],
        'Out of Stock Statuses' => [
            'Backordered' => 'stock_backordered',
            'Discontinued' => 'stock_discontinued',
        ]
    ],
    'choices_as_values' => true,
);
```

Aunque también se puede hacer mejor con la opción _group_by_.

#### Opciones

*   [**choices**](http://symfony.com/doc/current/reference/forms/types/choice.html#choices). Type: _array_. Default:_ array()_. Forma más básica de especificación de choices.
*   [**choices_loader**](http://symfony.com/doc/current/reference/forms/types/choice.html#choice-loader). Type: _ChoiceLoaderInterface_. El choice_loader puede usarse para cargar las choices parcialmente en casos en que no hace falta cargar todas las opiones. Sólo en casos avanzados, y reemplazaría a choices.
*   [**choice_label**](http://symfony.com/doc/current/reference/forms/types/choice.html#choice-label). Type: _callable_ o _string_. Default: _null_. Normalmente el key de cada item de la opción choices se usa como texto a mostrar al usuario. Esta opción permite tomar más control.
*   [**choice_attr**](http://symfony.com/doc/current/reference/forms/types/choice.html#choice-attr). Type: _array_, _callable_ o _string_. Default: _array()_. Añade atributos adicionales a cada choice.
*   [**placeholder**](http://symfony.com/doc/current/reference/forms/types/choice.html#placeholder). Type: _string_ o _boolean_. Esta opción determina si aparece o no una opción especial vacía (como "Elige una opción"). Sólo es posible cuando la opción multiple es _false_.
*   [**choice_translation_domain**](http://symfony.com/doc/current/reference/forms/types/choice.html#choice-translation-domain). Type: _string_, _boolean_ o _null_. Esta opción determina si los valores a elegir deben traducirse y en que dominio de traducción.
*   **[expanded](http://symfony.com/doc/current/reference/forms/types/choice.html#expanded)**. Type: _boolean_. Default: _false_. Si es true, se mostrarán buttons o checkboxes (dependiendo del valor _multiple_). Si es false se mostrará un elemento _select_.
*   [**multiple**](http://symfony.com/doc/current/reference/forms/types/choice.html#multiple). Type: _boolean_. Default: _false_. si es true, el usuario podrá seleccionar múltiples opciones (en lugar de poder elegir sólo una opción).
*   [**preferred_choices**](http://symfony.com/doc/current/reference/forms/types/choice.html#preferred-choices). type: _array_, _callable_, _string_. Default: _array()_. Permite mover ciertas choices arriba de la lista con un separador visual entre ellas y el resto de opciones (para opciones populares, por ejemplo).
*   [**group_by**](http://symfony.com/doc/current/reference/forms/types/choice.html#group-by). Type: _array_, _callable_, _string_. Default: _null_. Permite agrupar opciones en un select simplemente pasando un [array multidimensional](http://diego.com.es/arrays-en-php#ArraysMultidimensionales) de choices. 
*   [**choice_value**](http://symfony.com/doc/current/reference/forms/types/choice.html#choice-value). Type: _callable_ o _string_. Default: _null_. Devuelve el string "_value_" para cada choice. Se usa en el atributo _value_ en HTML y enviado en los requests POST/PUT. Puede ser de utilidad con API requests (ya que puedes configurar el _value_ que se enviará en el API request).
*   [**choice_name**](http://symfony.com/doc/current/reference/forms/types/choice.html#choice-name). Type: _callable_ o _string_. Default: _null_. Controla el valor interno del campo de choice. Normalmente no importa esta opción, salvo en casos avanzados.

Opciones sobreescritas

*   **compound**. Type: _boolean_. Default: mismo valor que la opción expanded.
*   [**empty_data**](http://symfony.com/doc/current/reference/forms/types/choice.html#empty-data). Type: _mixed_.
*   **error_bubbling**. Type: _boolean_. Default: _false_.

#### Opciones heredadas

by_reference, data, disabled, error_mapping, inherit_data, label, label_attr, mapped, required

### EntityType

Carga opciones desde una entidad Doctrine.

#### Class: EntityType

#### Opciones

*   [**class**](http://symfony.com/doc/current/reference/forms/types/entity.html#class). Type: _string_. **Required**. La clase de la entidad (AppBundle:Category o AppBundle\Entity\Category).
*   [**choice_label**](http://symfony.com/doc/current/reference/forms/types/entity.html#choice-label). Type: _string_ o _callable_. Propiedad que ha de usarse para mostrar las entidades como texto en el elemento HTML. Si se deja en blanco el objeto se mostrará como string por lo que debe de tener un método ___toString()_. 
*   [**query_builder**](http://symfony.com/doc/current/reference/forms/types/entity.html#query-builder). Type: Doctrine\ORM\QueryBuilder o un Closure. Utiliza una consulta para las opciones.
*   [**em**](http://symfony.com/doc/current/reference/forms/types/entity.html#em). Type: _string | Doctrine\Common\Persistence\ObjectManager_. Default: _el entity manager por defecto_. Este entity manager se usará para cargar las opciones en lugar del default manager por defecto.

#### Opciones sobreescritas

*   **choices**. Type: _array_ | _\Traversable_. Default: _null_. 

#### Opciones heredadas

De **ChoiceType**: placeholder, choice_translation_domain, expanded, multiple, preferred_choices, group_by.

De **FormType**: data, disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required.

### CountryType

Muestra todos los países del mundo, en el idioma elegido.

#### Class: CountryType

#### Opciones sobreescritas

*   **choices**. Default: _Symfony\Component\Intl\Intl::getRegionBundle()->getCountryNames()_. El valor por defecto de _choices_ es la lista entera de países. El _locale_ se emplea para traducir los nombres de los países.

#### Opciones heredadas

De **ChoiceType**: placeholder, error_bubbling, error_mapping, expanded, multiple, preferred_choices.

De **FormType**: data, disabled, empty_data, label, label_attr, mapped, required.

### LanguageType

Muestra una gran lista de idiomas, en el idioma elegido.

#### Class: LanguageType

#### Opciones sobreescritas

*   **choices**. Default: _Symfony\Component\Intl\Intl::getLanguageBundle()->getLanguageNames()_. El valor por defecto de _choices_ es todos los idiomas. El _locale_ se emplea para traducir los nombres de los idiomas.

#### Opciones heredadas

De **ChoiceType**: placeholder, error_bubbling, error_mapping, expanded, multiple, preferred_choices.

De **FormType**: data, disabled, empty_data, label, label_attr, mapped, required.

### LocaleType

Muestra una gran lista de locales, en el idioma elegido.

#### Opciones sobreescritas

*   **choices**. Default: Symfony\Component\Intl\Intl::getLocaleBundle()->getLocaleNames(). El valor por defecto de choices es todos los locales. Utiliza el _locale_ por defecto para especificar el lenguage.

#### Opciones heredadas

De **ChoiceType**: placeholder, error_bubbling, error_mapping, expanded, multiple, preferred_choices.

De **FormType**: data, disabled, empty_data, label, label_attr, mapped, required.

### TimeZone

Muestra una lista de todos los posibles timezones, mostrados completos (America/Chicago, Europe/Madrid).

#### Class: TimezoneType

#### Opciones sobreescritas

*   **choices**. Default: _array of timezones_. El Type son las choices de todos los timezones devueltos por DateTimeZone::listIdentifiers(), separados por continente.

#### Opciones heredadas

De **ChoiceType**: placeholder, expanded, multiple, preferred_choices.

De **FormType**: data, disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required.

### CurrencyType

Muestra una lista de monedas.

#### Class: TimezoneType

#### Opciones sobreescritas

*   **choices**. _Symfony\Component\Intl\Intl::getCurrencyBundle()->getCurrencyNames()_. La opción choices por defecto es la lista de monedas.

#### Opciones heredadas

De **ChoiceType**: placeholder, error_bubbling, expanded, multiple, preferred_choices.

De **FormType**: data, disabled, empty_data, label, label_attr, mapped, required.

## 4\. Date and Time Fields

### DateType

Permite modificar la información de fechas a través de diferentes elementos HTML.

#### Class: DateType

#### Uso básico

Este **field type** es muy configurable, pero fácil de usar. Las opciones más importantes son _input_ y _widget_. 

Si por ejemplo tenemos un campo _publishedAt_ cuya fecha es un objeto **DateTime**:

```
use Symfony\Component\Form\Extension\Core\Type\DateType;
// ...

$builder->add('publishedAt', DateType::class, array(
    'input'  => 'datetime',
    'widget' => 'choice',
));
```

La opción _input_ permite elegir el tipo de datos de fecha. Por ejemplo, también puede ser un **timestamp**.

```
use Symfony\Component\Form\Extension\Core\Type\DateType;
// ...

$builder->add('publishedAt', DateType::class, array(
    'input'  => 'timestamp',
    'widget' => 'choice',
));
```

El campo también soporta un _array_ y _string_ como _input option values_.

#### Opciones

*   [**days**](http://symfony.com/doc/current/reference/forms/types/date.html#days). Type: _array_. Default: _1 a 31_. Lista de días disponibles. Sólo relevante si la opción _widget_ es _choice_.
*   [**placeholder**](http://symfony.com/doc/current/reference/forms/types/date.html#placeholder). Type: _string_ o _array_. Si la opción _widget_ es _choice_, este campo se representa por series de cajas select. 
*   [**format**](http://symfony.com/doc/current/reference/forms/types/date.html#format). Type: _integer_ o _string_. Default: _IntlDateFormatter::MEDIUM (o yyy-MM-dd si widget es single_text)_. La opción pasada a la clase IntlDateFormatter se usa para transformar user input en el formato apropiado. Es importante cuando la opción _widget_ es _single_text_.
*   [**html5**](http://symfony.com/doc/current/reference/forms/types/date.html#html5). Type: boolean. Default: true. Si se establece a **true**, utiliza el type HTML5 (date, time o datetime) para renderizar el campo. Si es **false**, usará _text type_.
*   [**input**](http://symfony.com/doc/current/reference/forms/types/date.html#input). Type: _string_. Default: _datetime_. El formato de los datos intput (el formato en que la fecha se guarda en el objeto). Válidos: _string_ (2011-05-04), _datetime_ (DateTime object), _array_ (array('year'=>2011, 'month'=>02, 'day'=>05), _timestamp_ (12314123553)
*   [**model_timezone**](http://symfony.com/doc/current/reference/forms/types/date.html#model-timezone). Type: _string_. Default: _timezone por defecto del sistema_. Timezone en que los datos se guardan. El timezone debe ser [soportado por PHP](http://php.net/manual/en/timezones.php).
*   [**months**](http://symfony.com/doc/current/reference/forms/types/date.html#months). Type: _array_. Default: _1 a 12_. Lista de meses disponibles. Sólo relevante si la opción _widget_ es _choice._
*   [**view_timezone**](http://symfony.com/doc/current/reference/forms/types/date.html#view-timezone). Type: _string_. Default: _timezone por defecto del sistema_. El timezone de cómo los datos deberían mostrarse al usuario (por tanto los datos también que el usuario envía). El timezone debe ser soportado por PHP.
*   [**widget**](http://symfony.com/doc/current/reference/forms/types/date.html#widget). Type: _string_. Default: _choice_. La forma en que se renderizará este campo: _choice_ (3 inputs select), _text_ (3 text input), _single_text_ (single input de type date).
*   [**years**](http://symfony.com/doc/current/reference/forms/types/date.html#years). Type: array. Default: 5 años antes a 5 años después del año actual. Lista de años disponibles. Sólo relevante si la opción _widget_ es _choice_.

#### Opciones sobreescritas

*   **by_reference**. Default: _false_.
*   **compound**. Type: _boolean_. Default: _false_.
*   **data_class**. Type: _string_. Default: _null_.
*   **error_bubbling**. Default: _false_.

#### Opciones heredadas

data, disabled, error_mapping, inherit_data, invalid_message, invalid_message_parameters, mapped

### DateTimeType

Permite al usuario modificar los datos que representan una determinada fecha y hora.

#### Class: DateTimeType

#### Opciones

Iguales que **DateType**: days, html5, input, model_timezone, months, time_widget, view_timezone, widget, years

*   **date_format**. Type: _integer_ o _string_. Default: _IntlDateFormatter::MEDIUM_. Define el formato del campo.
*   **date_widget**. Type: _string_. Default: _choice_. Forma en la que renderizar el campo: _choice_, _text_ o _single_text_.
*   [**placeholder**](http://symfony.com/doc/current/reference/forms/types/datetime.html#placeholder). Type: string o boolean. Determina si incluir una opción vacía ("Elige una opción"). Sólo aplicable si la opción _multiple_ es false.
*   [**format**](http://symfony.com/doc/current/reference/forms/types/datetime.html#format). Type: _string_. Default: _Symfony\Component\Form\Extension\Core\Type\DateTimeType::HTML5_FORMAT_. Si la opción _widget_ es _single_text_, esta opción especifica el formato del input.
*   **hours**. Type: _array_. Default: _0 a 23_. Lista de horas disponibles. Sólo relevante si la opción _widget_ es _choice_.
*   **minutes**. Type: _array_. Default: _0 a 59_. Lista de minutos disponibles. Sólo relevante si la opción _widget_ es _choice_.
*   **seconds**. Type: _array_. Default: _0 a 59_. Lista de segundos disponibles. Sólo relevante si la opción _widget_ es _choice_.
*   **with minutes**. Type: _boolean_. Default: _true_. Si incluir o no los minutos en el input.
*   **with seconds**. Type: _boolean_. Default: _false_. Si incluir o no los segundos en el input.

#### Opciones sobreescritas

*   **by_reference**. Default: _false_.
*   **compound**. Type: _boolean_. Default: _false_.
*   **data_class**. Type: _string_. Default: _null_.
*   **error_bubbling**. Default: _false_.

#### Opciones heredadas

data, disabled, inherit_data, invalid_message, invalid_message_parameters, mapped

### TimeType

#### Class: TimeType

#### Opciones

Iguales que **DateTimeType**: placeholder, hours, html5, input, minutes, model_timezone, seconds, view_timezone, widget, with_minutes, with_seconds

#### Opciones sobreescritas

Iguales que **DateTimeType**.

#### Opciones heredadas

data, disabled, error_mapping, inherit_data, invalid_message, invalid_message_parameters, mapped

### BirthdayType

#### Class: BirthdayType

#### Opciones sobreescritas

*   **years**. Type: _array_. Default: _120 años atrás del año actual_.

#### Opciones heredadas

De **DateType**: days, placeholder, format, input, model_timezone, months, view_timezone, widget

De **FormType**: data, disabled, inherit_data, invalid_message, invalid_message_parameters, mapped

## 5\. Other Fields

### CheckboxType

#### Class: _CheckboxType_

#### Opciones

*   **value**. Type: mixed. Default: 1\. El value empleado como valor para el checkbox o radio button. No afecta al valor establecido en tu objeto.

#### Opciones sobreescritas

*   **compound**. Type: _string_. Default: _mixed_.
*   **empty_data**. Type: _string_. Default: _mixed_.

#### Opciones heredadas

data, disabled, error_bubbling, error_mapping, label, label_attr, mapped, required

### FileType

Explicado en una [sección anterior](http://diego.com.es/campos-filetype-en-symfony).

### RadioType

#### Class: _RadioType_

#### Opciones heredadas

De **CheckboxType**: value

De **FormType**: data, disabled, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required

## 6\. Field Groups

### CollectionType

#### Class: _CollectionType_

Empleado para renderizar una **colección de un campo o formulario**. En su forma más sencilla, puede ser un array de campos **TextType** que forman un array de valores _email_. En ejemplos más complejos puedes incrustar formularios enteros, lo que es útil cuando se crean formularios que muestran relaciones one-to-many.

#### Uso básico

Se emplea cuando quieres manejar una colección de elementos similares en un formulario. Por ejemplo si tenemos un campo _email_ que corresponde a un array de direcciones de email.

```
use Symfony\Component\Form\Extension\Core\Type\CollectionType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
// ...

$builder->add('emails', CollectionType::class, array(
    // each entry in the array will be an "email" field
    'entry_type'   => EmailType::class,
    // these options are passed to each "email" type
    'entry_options'  => array(
        'required'  => false,
        'attr'      => array('class' => 'email-box')
    ),
));
```

La forma más fácil de renderizar esto es todo de vez:

```
{{ form_row(form.emails) }}
```

Una forma más flexible:

```
{{ form_label(form.emails) }}
{{ form_errors(form.emails) }}

<ul>
{% for emailField in form.emails %}
    <li>
        {{ form_errors(emailField) }}
        {{ form_widget(emailField) }}
    </li>
{% endfor %}
</ul>
```

En ambos casos no se renderizan nuevos campos input a no ser que los datos de _emails_ ya contengan emails.

En el ejemplo anterior no podemos añadir o eliminar direcciones, es por ello que necesitamos las opciones _allow_add_ y _allow_delete_.

Si allow_add es true, si cualquier elemento no reconocido se envía, se añadirán al array. Está bien en la teoría pero lleva un poco más de esfuerzo en la práctica el tener el JavaScript correctamente.

Desde el ejemplo anterior, suponemos que empezamos con dos emails en el array de datos _emails_. En ese caso, dos campos input se renderizarán como estos:

```
<input type="email" id="form_emails_0" name="form[emails][0]" value="foo@foo.com" />
<input type="email" id="form_emails_1" name="form[emails][1]" value="bar@bar.com" />
```

Para permitir al usuario añadir otro email, simplemente hay que poner _allow_add_ en **true** y con **JavaScript** renderizar otro campo con el nombre form[emails][2] (y así continuamente para más campos).

Para hace esto más fácil, establecer la opción _prototype_ en true permite renderizar un campo **template**, el cual puedes usar después en **JavaScript** para crear dinámicamente estos campos. Un prototipo de campo renderizado:

```
<input type="email"
    id="form_emails___name__"
    name="form[emails][__name__]"
    value=""
/>
```

Reemplazando __name__ con algún valor único (como 2), puedes construir e insertar nuevos campos HTML en el formulario.

Un ejemplo con **jQuery** puede ser el siguiente. Si renderizas tu colección de campos todos de vez (_form_row(form.emails)_), resulta más sencillo ya que el atributo _data-prototype_ se renderiza automáticamente, y todo lo que se necesita es el **JavaScript**:

```
{{ form_start(form) }}
    {# ... #}

    {# guarda el prototipo en el atributo data-prototype #}
    <ul id="email-fields-list"
        data-prototype="{{ form_widget(form.emails.vars.prototype)|e }}">
    {% for emailField in form.emails %}
        <li>
            {{ form_errors(emailField) }}
            {{ form_widget(emailField) }}
        </li>
    {% endfor %}
    </ul>

    <a href="#" id="add-another-email">Añadir otro email</a>

    {# ... #}
{{ form_end(form) }}

<script type="text/javascript">
    // Llevar un segumiento de cuantos campos de email se han renderizado
    var emailCount = '{{ form.emails|length }}';

    jQuery(document).ready(function() {
        jQuery('#add-another-email').click(function(e) {
            e.preventDefault();

            var emailList = jQuery('#email-fields-list');

            // obtener el prototipo template
            var newWidget = emailList.attr('data-prototype');
            // reemplazar el "__name__" utilizado en el id y nombre del prototipo
            // con un número único para los emails
            // el atributo name final queda así: name="contact[emails][2]"
            newWidget = newWidget.replace(/__name__/g, emailCount);
            emailCount++;

            // crear un nuevo elemento y añadirlo a la lista
            var newLi = jQuery('<li></li>').html(newWidget);
            newLi.appendTo(emailList);
        });
    })
</script>
```

Si se renderiza la colección entera de vez, el **prototype** está automáticamente disponible en el atributo _data-prototype_ del elemento (por ejemplo div o table) que envuelve a la colección. La única diferencia es que el _form row_ entero se renderiza, lo que significa que no puedes envolverlo en algún elemento contenedor.

#### Opciones

*   [**allow_add**](http://symfony.com/doc/current/reference/forms/types/collection.html#allow-add). Type: _boolean_. Default: _false_. Si es **true**, elementos no reconocidos se envían a la colección, se añadirán como nuevos elementos.
*   [**allow_delete**](http://symfony.com/doc/current/reference/forms/types/collection.html#allow-delete). Type: _boolean_. Default: _false_. Si es **true**, si un elemento existente no está contenido en los datos enviados, se eliminará del array final de elementos. Esto permite implementar un botón "eliminar" con JavaScript que elimina un elemento de formulario del **DOM**.
*   **delete_empty**. Type: _boolean_. Default: _false_. Si quieres eliminar explícitamente entradas vacías en colecciones del formulario.
*   [**entry_options**](http://symfony.com/doc/current/reference/forms/types/collection.html#entry-options). Type: _array_. Default: _array()_. Array que se pasa al **form type** especificado en la opción _entry_type_.
*   **entry_type**. Type: _string_ o _FormTypeInterface_ required. Este es el field type para cada elemento de la colección.
*   [**prototype**](http://symfony.com/doc/current/reference/forms/types/collection.html#prototype). Type: _boolean_. Default: _true_. Si es **true** y _allow_add_ es true, un atributo especial "_prototype_" estará disponible para poder renderizar un ejemplo de "template" en la página para ver un nuevo elemento.
*   **prototype_name**. Type: _string_. Default: ___name___. Si tienes varias colecciones en el formulario, o colecciones anidadas, podrías querer cambiar el placeholder para que los que no estén relacionados no se sobreescriban los valores.

#### Opciones heredadas

by_reference, empty_data, error_bubbling, error_mapping, label, label_attr, mapped, required

### RepeatedType

#### Class: _RepeatedType_

Este es un grupo especial de campor, crea dos campos idénticos cuyos valores deben coincidir (o se lanza un error de validación). El uso más común es cuando se quiere que el usuario repita el email o contraseña para verificar.

#### Ejemplo de uso

```
use Symfony\Component\Form\Extension\Core\Type\RepeatedType;
use Symfony\Component\Form\Extension\Core\Type\PasswordType;
// ...

$builder->add('password', RepeatedType::class, array(
    'type' => PasswordType::class,
    'invalid_message' => 'The password fields must match.',
    'options' => array('attr' => array('class' => 'password-field')),
    'required' => true,
    'first_options'  => array('label' => 'Password'),
    'second_options' => array('label' => 'Repeat Password'),
));
```

Si el envío es satisfactorio, el valor introducido en ambos campos _password_ se convierten en los datos de la key _password_. Aunque se rendericen dos campos, el resultado es uno.

La opción más importante es _type_, que puede ser cualquier **field type** y determina el type de los dos campos. La opción _options_ se pasa a cada uno de los campos individuales.

Los campos se pueden renderizar a la vez:

```
{{ form_row(form.password) }}
```

O de forma individual:

```
{# .first y .second pueden variar #}
{{ form_row(form.password.first) }}
{{ form_row(form.password.second) }}
```

Los nombres first y second son por defecto para los dos subcampos, pero pueden modificarse con las opciones _first_name_ y _second_name_. 

#### Opciones

*   **first_name**. Type: _string_. Default: _first_. El nombre del primer campo.
*   **first_options**. Type: _array_. Default: _array()_. Opciones adicionales sólo para el primer campo.
*   **options**. Type: _array_. Default: _array()_. Array de opciones que se pasará a los dos campos.
*   **second_name**. Type: _string_. Default: _second_. El nombre del segundo campo.
*   **second_options**. Type: _array_. Default: _array()_. Opciones adicionales sólo para el segundo campo.
*   **type**. Type: _string_. Default: _text_. El field type de los dos campos.

#### Opciones sobreescritas

*   **error_bubbling**. Default: _false_.

#### Opciones heredadas

data, error_mapping, invalid_message, invalid_message_paramentes, mapped

## 7\. Hidden Fields

### HiddenType

#### Class: _HiddenType_

Representa un campo _hidden_.

#### Opciones sobreescritas

*   **compound**. Type: _boolean_. Default: _false_.
*   **error_bubbling**. Default: _true_.
*   **required**. Default: _false_.

#### Opciones heredadas

data, error_mapping, mapped, property_path

## 8\. Buttons

### ButtonType

#### Class: _ButtonType_

Un simple botón no responsive.

#### Opciones heredadas

attr, disabled, label, translation_domain

### ResetType

#### Class: _ResetType_

#### Opciones heredadas

attr, disabled, label, label_attr, translation_domain

### SubmitType

#### Class: _SubmitType_

#### Opciones heredadas

attr, disabled, label, label_attr, translation_domain, validation_groups