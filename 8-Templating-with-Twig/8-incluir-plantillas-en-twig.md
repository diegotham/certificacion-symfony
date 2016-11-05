Por defecto, las **plantillas** pueden encontrarse en dos localizaciones:

*   _app/Resources/views_. El directorio views puede contener templates de tu aplicación así como templates que sobreescriben templates de bundles de terceros.
*   _path/to/bundle/Resources/views_. Cada bundle de terceros guarda sus templates en el directorio _Resources/views/_ (y subdirectorios). Cuando llevas intención compartir tu bundle es mejor poner las templates en el bundle en lugar de en el directorio _app/_. 

La mayoría de **templates** que emplees estarán en el directorio _app/Resources/views/_. El directorio que vayas a emplear para hacer referencia a las mismas será en referencia a este directorio. Por ejemplo para renderizar o extender _app/Resources/views/base.html.twig_, usarás el directorio _base.html.twig_ y para _app/Resources/views/blog/index.html.twig_ emplearás _blog/index.html.twig_. 

### Referenciar plantillas en un Bundle

**Symfony** emplea una sintaxis **bundle:directory:filename** para templates que están dentro de un bundle. Esto permite diferentes tipos de templates, cada uno en una localización diferente:

*   **AcmeBlogBundle:Blog:index.html.twig**: esta sintaxis se emplea para especificar una template para una página específica. Las tres partes del string, cada una separada por dos puntos, indican lo siguiente:
    *   AcmeBlogBundle. (bundle), src/Acme/BlogBundle.
    *   Blog. (directory), Resources/views.
    *   index.html.twig. (filename), el nombre del archivo.
*   **AcmeBlogBundle::layout.html.twig**: esta sintaxis se refiere a la template base específica del bundle **AcmeBlogBundle**. Ya que no se incluye la parte del medio (el directory), la template está en _Resources/views/layout.html.twig_. 

### Sufijos de las plantillas

Cada nombre de template tiene dos extensiones que especifican el _format_ y _engine_ de la **template**:

*   _blog/index.html.twig_. Format: **HTML**. Engine: **Twig**
*   _blog/index.html.php_. Format: **HTML**. Engine: **PHP**
*   _blog/index.css.twig_. Format: **CSS**. Engine: **Twig**

Por defecto, una template en Symfony puede estar escrita en PHP o en Twig, y la última parte de la extensión indica el motor (_engine_) a emplear. La primera parte de la extensión (.html, .css, .xml, etc) es el formato final en el que la plantilla se generará.

### Incluir plantillas

A menudo querrás incluir la misma **template** o **fragmento de código** en varias páginas. Por ejemplo en una aplicación de noticias, el código de la template mostrando una noticia podría incluir los detalles de la noticias, además de los artículos más populares, o una lista de últimos artículos.

Cuando necesitas **reutilizar código PHP**, normalmente mueves el código a una nueva clase o **función PHP**. Lo mismo ocurre en las templates. Moviendo el código reusable de la template en su propia template, hace que pueda incluirse después en cualquier otra. Primero crea la template que necesitas reutilizar:

```
{# app/Resources/views/article/article_details.html.twig #}
<h2>{{ article.title }}</h2>
<h3 class="byline">by {{ article.authorName }}</h3>

<p>
    {{ article.body }}
</p>
```

Incluir esta template en cualquier otra es muy simple:

```
{# app/Resources/views/article/list.html.twig #}
{% extends 'layout.html.twig' %}

{% block body %}
    <h1>Recent Articles<h1>

    {% for article in articles %}
        {{ include('article/article_details.html.twig', { 'article': article }) }}
    {% endfor %}
{% endblock %}
```

La template se incluye con la [función include](http://diego.com.es/funciones-en-twig#include). Nótese que el nombre de la template sigue la misma convención. Se ha incluído un argumento adicional que incluye la variable _article_ en _article_details.html.twig_, lo que no es necesario ya que todas las variables disponibles en _list.html.twig_ estarán disponibles también en _article_details.html.twig_ (a no ser que establezcas _with_context_ en false).