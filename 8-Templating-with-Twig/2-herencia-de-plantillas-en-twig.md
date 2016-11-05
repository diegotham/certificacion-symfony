De forma muy frecuente las **templates de un proyecto** comparten elementos comunes, como el _header_, _footer_, _sidebar_, etc. Con **Symfony** este problema se solventa de forma que **una template puede decorarse con otra**. Esto funciona de la misma forma que las **clases PHP**: la herencia de templates permite crear una capa base que contiene todos los elementos comunes definidos en bloques (como si fueran métodos base de clases PHP). Una template hija puede extender la base y sobreescribir cualquiera de sus bloques (como cuando una **subclase de PHP** sobreescribe métodos de su clase madre).

**Ejemplo**:

```
{# app/Resources/views/base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Test Application{% endblock %}</title>
    </head>
    <body>
        <div id="sidebar">
            {% block sidebar %}
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/blog">Blog</a></li>
                </ul>
            {% endblock %}
        </div>

        <div id="content">
            {% block body %}{% endblock %}
        </div>
    </body>
</html>
```

Esta template define el **esqueleto base HTML** de una página simple de dos columnas. En este ejemplo se han creado tres áreas _{% block %}_: _title_, _sidebar_ y _body_. Cada bloque puede ser sobreescrito por una template hija o se puede dejar con su implementación por defecto. También puede renderizarse directamente, lo que mostraría los valores por defecto de los tres bloques.

Una **template hija** puede ser como la siguiente:

```
{# app/Resources/views/blog/index.html.twig #}
{% extends 'base.html.twig' %}

{% block title %}My cool blog posts{% endblock %}

{% block body %}
    {% for entry in blog_entries %}
        <h2>{{ entry.title }}</h2>
        <p>{{ entry.body }}</p>
    {% endfor %}
{% endblock %}
```

La template madre se identifica con sintaxis de string (_'base.html.twig'_), y hace referencia al directorio _app/Resources/views_ del proyecto. 

La key para la herencia de templates es la etiqueta _{% extends %}_. Esta le dice al **motor de templates** que primero evalúe la **template base**, que establece la primera capa y define varios bloques. La template hija entonces se renderiza, y los bloques _title_ y _body_ de la madre se sobreescriben con los de la hija. Dependiendo del valor de **blog_entries**, el output podría resultar así:

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>My cool blog posts</title>
    </head>
    <body>
        <div id="sidebar">
            <ul>
                <li><a href="/">Home</a></li>
                <li><a href="/blog">Blog</a></li>
            </ul>
        </div>

        <div id="content">
            <h2>My first post</h2>
            <p>The body of the first post.</p>

            <h2>Another post</h2>
            <p>The body of the second post.</p>
        </div>
    </body>
</html>
```

Ya que la template hija no ha definido ningún bloque _sidebar_, se utiliza el valor de la madre. El contenido de una etiqueta _{% block %}_ en una template madre siempre se utiliza por defecto. 

Algunos tips a tener en cuenta en la herencia de templates:

*   Si empleas la etiqueta _{% extends %}_, ésta ha de ser la primera que se define.
*   Cuantas más etiquetas _{% block %}_ tengas mejor. Las templates hija no tienen que definir todos los bloques de la madre, por lo que puedes crear tantos bloques como quieras en las templates base para poder personalizarlas mejor. Cuantos más bloques más flexible será la template.
*   Si ves que estás **duplicando contenido en varios templates**, probablemente significa que debes mover ese contenido a un _{% block %}_ en una template madre. En ocasiones puede resultar mejor mover el contenido en una template nueva e incluirlo con _include_.
*   Si necesitas **obtener el contenido de un bloque de una template madre**, puedes utilizar la función _{{ parent() }}_. Esto es útil si quieres añadir los contenidos de un bloque madre en lugar de sobreescribirlo por completo.

```
{% block sidebar %}
    <h3>Table of Contents</h3>

    {# ... #}

    {{ parent() }}
{% endblock %}
```

Puedes emplear tantos **niveles de herencia** como necesites.

### Herencia de templates a tres niveles

La **herencia de templates** a tres niveles es una forma bastante común de organizar las templates de un proyecto. 

1.  Se crea una template _base app/Resources/views/base.html.twig_ que contiene el principal layout para la aplicación. Internamente esta template se llama _base.html.twig_.
2.  Crea una template para cada sección de tu sitio. Por ejemplo la funcionalidad _blog_ tendría una template llamada _blog/layout.html.twig_ que contiene sólo elementos de la sección _blog_.

   
```
    {# app/Resources/views/blog/layout.html.twig #}
    {% extends 'base.html.twig' %}

    {% block body %}
        <h1>Blog Application</h1>

        {% block content %}{% endblock %}
    {% endblock %}
   
```

3.  Crea templates individuales para cada página y haz que cada una extienda la sección a la que pertenece. Por ejemplo, la página _index_ podría llamarse _blog/index.html.twig_ y mostraría una lista de los posts del blog.

   
```

    {# app/Resources/views/blog/index.html.twig #}
    {% extends 'blog/layout.html.twig' %}

    {% block content %}
        {% for entry in blog_entries %}
            <h2>{{ entry.title }}</h2>
            <p>{{ entry.body }}</p>
        {% endfor %}
    {% endblock %}
   
```

De esta forma se crean los tres niveles: la template de entradas extiende a la **template base de la sección** (_blog/layout.html.twig_) y ésta extiende a la **template base de la aplicación** (_base.html.twig_).

### Sobreescribir templates de bundles

Cuando se usa un bundle de terceros es probable que tengas que sobreescribir y customizar alguna de sus plantillas.

Si por ejemplo hemos instalado el bundle **AcmeBlogBundle** y queremos sobreescribir la página de la lista de blogs para personalizarla, podemos ver en el Blog controller:

```
public function indexAction()
{
    // Lógica para devolver los blogs
    $blogs = ...;

    $this->render(
        'AcmeBlogBundle:Blog:index.html.twig',
        array('blogs' => $blogs)
    );
}
```

Cuando se renderiza _AcmeBlogBundle:Blog:index.html.twig_, **Symfony** busca en dos directorios la **template**:

1.  _app/Resources/AcmeBlogBundle/views/Blog/index.html.twig_
2.  _src/Acme/BlogBundle/Resources/views/Blog/index.html.twig_

Para sobreescribir la template del bundle, copia la template _index.html.twig_ del bundle a _app/Resources/AcmeBlogBundle/views/Blog/index.html.twig_, si el directorio _app/Resources/AcmeBlogBundle/_ no existe, tendrás que crearlo. Ahora ya puedes customizar la template. 

Si añades la template en un nuevo directorio, es probable que tengas que **limpiar la cache** (_php bin/console cache:clear_) incluso en **debug mode**.

Esta lógica también se aplica a las **templates base de los bundles**. Si cada template de **AcmeBlogBundle** hereda de una template base llamada _AcmeBlogBundle::layout.html.twig_, para sobreescribirla habrá que hacer lo mismo que antes, Symfony buscará en:

1.  _app/Resources/AcmeBlogBundle/views/layout.html.twig_
2.  _src/Acme/BlogBundle/Resources/views/layout.html.twig_

Para sobreescribirla, copiala del bundle a _app/Resources/AcmeBlogBundle/views/layout.html.twig_ y customízala.

### Sobreescribir templates Core

Ya que el **Symfony Framework** en sí mismo es un bundle, las templates core pueden sobreescribirse de la misma forma. Por ejemplo, **TwigBundle** contiene varios templates "exception" y "error" que pueden sobreescribirse copiando desde cada directorio, _Resources/views/Exception_ del TwigBundle al directorio _app/Resources/TwigBundle/views/Exception_.