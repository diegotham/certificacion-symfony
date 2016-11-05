Cuando se genera HTML desde una template, hay siempre un riesgo de que una variable de una template muestre **contenido HTML** no intencionado o **código peligroso por parte de terceros**. El resultado es que el contenido dinámico podría romper el HTML de la página o permitir a usuarios realizar [ataques XSS (Cross Site Scripting)](http://diego.com.es/ataques-xss-cross-site-scripting-en-php).

En las **plantillas Twig** el **output escaping** está activado por defecto, asumiendo que el contenido que se va a escapar es para mostrar **HTML**. El siguiente ejemplo muestra el contenido de un artículo:

```
{{ article.body }}
```

En algunos casos puede ser necesario **desactivar output escaping** para renderizar una variable en la que se confía y contiene lenguaje de marcado que no ha de escaparse. Para esto simplemente se añade la etiqueta _raw_:

```
{{ article.body|raw }}
```

Podemos **desactivar el autoescaping de forma global** desactivando en el archivo de configuración (_app/config/config.yml_) el _key_ **autoescape**:

```
twig:
    // ...
    autoescape: false
```

Si tenemos el autoescaping desactivado, podemos escapar manualmente de la siguiente forma:

```
{{ article.body|e }}
```

Por defecto escapará los contenidos para HTML, pero puedes especificar otros formatos:

```
{{ article.body|e('js') }}
{{ article.body|e('css') }}
{{ article.body|e('url') }}
{{ article.body|e('html_attr') }}
```

Independientemente de si el escapado automático está activado o no, se puede marcar una sección de una template como escaped con la tag **autoescape**:

```
{% autoescape %}
    Aquí todo será escapado (en HTML)
    {{ foo }} {# La variable foo se escapará #}
    {{ var|raw }} {# La variable var no se escapará #}
{% endautoescape %}
```

Y también podemos especificar un formato concreto:

```
{% autoescape 'js' %}
    Aquí todo será escapado (en JS)
{% endautoescape %}
```

Si ya se ha escapado algo, Twig no lo escapará otra vez aunque se les aplique las etiquetas _e_ o _escape_.

Tampoco escapa las expresiones _static_:

```
{% set hola = "<strong>Hola</strong>" %}
{{ hola }}
{{ "<strong>world</strong>" }}
```

En el ejemplo anterior no escapará _Hola_.