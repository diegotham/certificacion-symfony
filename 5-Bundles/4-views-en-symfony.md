El controller es el responsable de administrar cada request que entra a una **aplicación Symfony**. En realidad, el controller delega la mayoría del trabajo pesado a otros módulos para que el código pueda testearse y reusarse. Cuando un controller necesita **generar HTML**, **CSS** o cualquier otro contenido, delega la tarea al **motor de templates**.

### Templates

Un template es simplemente un archivo de texto que puede generar cualquier formato basado en texto (**HTML**, **XML**, **CSV**, LaTeX, etc). El tipo más familiar de template es en PHP, un archivo de texto analizado por PHP que contiene una mezcla de texto y **código PHP**:

```
<!DOCTYPE html>
<html>
    <head>
        <title>Welcome to Symfony!</title>
    </head>
    <body>
        <h1><?php echo $page_title ?></h1>

        <ul id="navigation">
            <?php foreach ($navigation as $item): ?>
                <li>
                    <a href="<?php echo $item->getHref() ?>">
                        <?php echo $item->getCaption() ?>
                    </a>
                </li>
            <?php endforeach ?>
        </ul>
    </body>
</html>
```

Pero **Symfony** viene con un **lenguaje de templates** potente llamado [Twig](http://twig.sensiolabs.org/). **Twig** te permite escribir templates fácilmente legibles que son mejores para **diseñadores web** y, de alguna forma, más potentes que las **templates PHP**.

```
<!DOCTYPE html>
<html>
    <head>
        <title>Welcome to Symfony!</title>
    </head>
    <body>
        <h1>{{ page_title }}</h1>

        <ul id="navigation">
            {% for item in navigation %}
                <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
            {% endfor %}
        </ul>
    </body>
</html>
```

Twig define tres tipos de sintaxis:

*   **{{ ... }}** - Imprimir algo. Imprime una variable o el resultado de una expresión en la template. 
*   **{% ... %}** - Hacer algo. Controla la lógica de la template. Se usa para ejecutar sentencias como loops foreach, por ejemplo.
*   **{# ... #}** - Comentar algo. Es equivalente a los comentarios de PHP _/* comentario */_. Se pueden escribir comentarios de una línea o multilínea. El contenido de los comentarios no se incluye en las páginas renderizadas.

Twig también contiene filtros, que modifican el contenido antes de ser renderizado. El siguiente muestra el título en mayúsculas:

```
{{ title|upper }}
```

Twig viene con una larga lista de [tags](http://twig.sensiolabs.org/doc/tags/index.html) y [filtros](http://twig.sensiolabs.org/doc/filters/index.html) disponibles por defecto, y también puedes [añadir tus propias extensiones](http://twig.sensiolabs.org/doc/advanced.html#creating-an-extension). Añadir estensiones ya creadas se hace de la misma forma que añadir un nuevo service y etiquetarlo con la etiqueta [twig.extension](http://symfony.com/doc/current/reference/dic_tags.html#reference-dic-tags-twig-extension).

Twig también soporta funciones, y se pueden añadir las que quieras. Por ejemplo el siguiente ejemplo emplea una etiqueta _for_ y la función _cycle_ para imprimir 10 etiquetas div, alternando la class _odd_ y _even_:

```
{% for i in 0..10 %}
    <div class="{{ cycle(['odd', 'even'], i) }}">
      <!-- some HTML here -->
    </div>
{% endfor %}
```

Las templates Twig están hechas para ser simples y no procesar etiquetas PHP. El sistema de templates Twig está hecho para expresar presentación, no lógica de programación. Twig también puede hacer cosas que no puede hacer PHP como control de espacios en blanco, sandboxing, escapado automático de HTML, escapado manual contextual de HTML, o la inclusión de funciones y filtros customizados que sólo afectan a las templates. Twig contiene características que hace el escribir templates más fácil y más conciso. El siguiente ejemplo combina un **loop** con una sentencia _if_:

```
<ul>
    {% for user in users if user.active %}
        <li>{{ user.username }}</li>
    {% else %}
        <li>No users found</li>
    {% endfor %}
</ul>
```

Twig también es especialmente rápido. Cada **template Twig** se compila en una **clase PHP** que se renderiza en tiempo de ejecución. Las clases compiladas se encuentran en el directorio _var/cache/{environment}/twig_, donde {_environment_} es el **entorno**, como **dev** o **prod**, y en algunos casos puede ser útil al depurar. Si el modo **debug** está activado (como en el entorno **dev**), una template Twig se recompila cuando se hacen cambios en ella. Esto significa que durante el desarrollo puedes hacer cambios en una template Twig e instantáneamente ver los cambios sin tener que preocuparte en la cache. Cuando el modo **debug** está desactivado (como en el entorno **prod**), es necesario limpiar la cache del directorio Twig para que las templates Twig puedan regenerarse.