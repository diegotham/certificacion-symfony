Cada parte de un **formulario** que se renderiza puede customizarse. Puedes cambiar como se renderiza cada _row_, cambiar el marcado empleado para renderizar **errores** o personalizar cómo mostrar la etiqueta _textarea_. 

Symfony emplea templates para renderizar cada parte de un formulario, como etiquetas _label_, _input_, mensajes de error y cualquier otra cosa.

En Twig, cada "fragmento" se representa por un **bloque Twig**. Para personalizar cualquier parte un formulario, necesitas sobreescribir el bloque apropiado.

Vamos a customizar el fragmento _form_row_ y añadir un atributo **class** al elemento _div_ que envuelve a cada _row_. Para hacerlo, crea un nuevo archivo template que guarde el nuevo atributo:

```
{# app/Resources/views/form/fields.html.twig #}
{% block form_row %}
{% spaceless %}
    <div class="form_row">
        {{ form_label(form) }}
        {{ form_errors(form) }}
        {{ form_widget(form) }}
    </div>
{% endspaceless %}
{% endblock form_row %}
```

El fragmento _form_row_ se emplea cuando se renderiza la mayoría de campos a través de la función _form_row_. Para decirle al componente **Form** que utiliza el nuevo fragmento _form_row_ definido antes, añade lo siguiente en la parte de arriba de la template que renderiza el formulario:

```
{# app/Resources/views/default/new.html.twig #}
{% form_theme form 'form/fields.html.twig' %}

{# or if you want to use multiple themes #}
{% form_theme form 'form/fields.html.twig' 'form/fields2.html.twig' %}

{# ... render the form #}
```

La etiqueta _form_theme_ en **Twig** importa los fragmentos definidos en la template dada y los utiliza cuando renderiza el **formulario**. En otras palabras, cuando se llama a la función _form_row_ en esta template, empleará el bloque _form_row_ de la theme personalizada (en lugar el bloque por defecto _form_row_ que viene con **Symfony**).

La theme personalizada no tiene por qué sobreescribir todos los bloques. Cuando se renderiza un bloque que no está sobreescrito, el motor de themes mostrará la theme global (definida a nivel de bundle).

Si se proporcionan varias **themes** se buscarán en el orden listado antes de acceder al theme global.

Para **personalizar cualquier porción de un formulario** sólo hay que sobreescribir el fragmento apropiado, y para ello debemos saber cómo funciona el **nombrado de los fragmentos de un formulario**.

### Nombrado de los fragmentos de un formulario

En **Symfony**, cada parte de un formulario que se renderiza (**elementos HTML**, **errores**, **etiquetas**, etc) se define en una **theme base**, que es una colección de **bloques en Twig**.

Cada bloque que se requiere en una template (por ejemplo, _form_div_layout.html.twig_) se encuentra en el [Twig Bridge](https://github.com/symfony/symfony/tree/master/src/Symfony/Bridge/Twig). Dentro del archivo se puede ver cada **bloque** y cada **field type**.

Cada **nombre de fragmento** sigue el **mismo patrón básico** y está dividido en dos piezas, separadas por una barra baja:

*   _form_row_. Empleada por _form_row_ para renderizar la mayoría de los campos.
*   _textarea_widget_. Empleada por _form_widget_ para renderizar el field type textarea.
*   _form_errors_. Empleada por _form_errors_ para renderizar errores de un campo.

Cada fragmento sigue el mismo patrón básico: _type_part_. La porción type corresponde al _field type_ que se renderiza (**textarea**, **checkbox**, **date**, etc) mientras que la porción _part_ corresponde a qué se está renderizando. Por defecto hay 4 posibles _parts_ de un formulario que pueden renderizarse:

| | | |
| -------- | -------- |
| **Nombre** | **Ejemplo** | **Renderiza** |
| label | form_label | La etiqueta del campo |
| widget | form_widget | La representación HTML del campo |
| errors | form_errors | Los errores del campo |
| row | form_row | El row entero del campo (label, widget y errors) |

Realmente hay dos _parts_ más: _rows_ y _rest_, pero no se suelen sobreescribir.

Sabiendo el field type (por ejemplo _textarea_) y que parte quieres personalizar (por ejemplo, _widget_), puedes construir el nombre del fragmento que se ha de sobreescribir (por ejemplo: _textarea_widget_).

### Herencia de fragmentos de templates

En algunos casos, el fragmento que quieres personalizar no existe. Por ejemplo, no hay un fragmento _textarea_errors_ en los themes por defecto de Symfony. ¿Cómo se renderizan los errores para un campo _textfield_?

A través del fragmento _form_errors_. Cuando **Symfony** renderiza los errores para un textarea type, primero busca un fragmento _textarea_errors_ antes de llegar al fragmento _form_errors_. Cada **field type** tiene un **parent type** (el parent type de _textarea_ es _text_, y su parent es _form_) y Symfony emplea el fragmento del parent type si el actual no existe.

Así que para sobreescribir los errores para sólo los campos de **textarea**, copia el fragmento _form_errors_, renómbralo a _textarea_errors_ y customízalo. Para sobreescribir el renderizado de errores por defecto para todos los campos, copia y personaliza el fragmento _form_errors_ directamente.

El **parent type** de cada campo está disponible en la [referencia](http://symfony.com/doc/current/reference/forms/types.html).

### Global form theming

En el ejemplo anterior hemos empleado el helper _form_theme_ para importar los fragmentos personalizados en sólo ese formulario. Puedes también decirle a **Symfony** que importe las customizaciones de formularios en el proyecto entero.

Para incluir automáticamente los bloques customizados del template _fields.html.twig_ creado en todas las templates, modificamos el archivo de configuración:

```
# app/config/config.yml
twig:
    form_themes:
        - 'form/fields.html.twig'
    # ...
```

Cualquier bloque dentro del template _fields.html.twig_ se usa ahora globalmente para definir el output del formulario.

También es posible customizar el bloque del formulario en una template donde se necesita esa misma customización:

```
{% extends 'base.html.twig' %}

{# import "_self" as the form theme #}
{% form_theme form _self %}

{# personalizar el fragmento del formulario #}
{% block form_row %}
    {# output customizado del row field #}
{% endblock form_row %}

{% block content %}
    {# ... #}

    {{ form_row(form.task) }}
{% endblock %}
```

La etiqueta _{% form_theme form _self %}_ permite a los **bloques de formularios** customizarse directamente dentro de la template que usará esas customizaciones. Utiliza este método para hacer customizaciones rápidas que sólo sean necesarias en una sóla template. Esta funcionalidad sólo funciona si la template extiende a otra. Si la template no extiende, debes apuntar _form_theme_ a otra template.