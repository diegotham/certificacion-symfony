En ocasiones se tienen que hacer más cosas aparte de incluir una simple template. Por ejemplo si tenemos una sidebar en el _layout_ que contiene los tres artículos más recientes, obtenerlos puede incluir consultar la base de datos u otra lógica pesada que no puede hacerse dentro de la template.

La solución es simplemente incrustar el resultado de un controller desde la template. Primero, creamos un controller que renderiza un número concreto de **artículos recientes**.

```
// src/AppBundle/Controller/ArticleController.php
namespace AppBundle\Controller;

// ...

class ArticleController extends Controller
{
    public function recentArticlesAction($max = 3)
    {
        // hacer una llamada a la base de datos u otra lógica
        // obtener un número "$max" de artículos recientes
        $articles = ...;

        return $this->render(
            'article/recent_list.html.twig',
            array('articles' => $articles)
        );
    }
}
```

La template _recent_list_ es como sigue:

```
{# app/Resources/views/article/recent_list.html.twig #}
{% for article in articles %}
    <a href="/article/{{ article.slug }}">
        {{ article.title }}
    </a>
{% endfor %}
```

Hay que tener en cuenta que esta forma de enlazar no sería la más correcta, sería mejor utilizar la [función path](http://diego.com.es/generar-urls-en-plantillas-con-symfony).

Para incluir el controller, tendrás que hacer referencia al mismo empleando la sintaxis estándar de strings para controllers (_bundle:controller:action_):

```
{# app/Resources/views/base.html.twig #}

{# ... #}
<div id="sidebar">
    {{ render(controller(
        'AppBundle:Article:recentArticles',
        { 'max': 3 }
    )) }}
</div>
```

Cuando veas que necesitas una variable o una pieza de información a la que no tienes acceso en una template, considera **renderizar el controller**. Los controllers son rápidos de ejecutar y proporcionan una buena **organización de código y reutilización**. Como todos los controllers, deben ser lo más ligeros posible, dejando las partes pesadas en _services_ reusables. 

### Contenido asíncrono con hinclude.js

Los controllers pueden incrustarse de forma asíncrona con la librería JavaScript [hinclude.js](http://mnot.github.io/hinclude/). Ya que el contenido incrustado viene de otra página (o controller en este caso), Symfony emplea una versión de la función estándar _render_ para configurar etiquetas **hinclude**:

```
{{ render_hinclude(controller('...')) }}
{{ render_hinclude(url('...')) }}
```

Para que esto funcione ha de incluirse _hinclude.js_ en la template.

Cuando se utiliza un controller en lugar de una URL se ha de activar el _fragments_ de la configuración de Symfony:

```
# app/config/config.yml
framework:
    # ...
    fragments: { path: /_fragment }
```

**Contenido por defecto** (durante la carga o si **JavaScript** está desactivado) puede establecerse globalmente en la configuración de la aplicación:

```
# app/config/config.yml
framework:
    # ...
    templating:
        hinclude_default_template: hinclude.html.twig
```

Puedes definir templates por defecto por cada función _render_ (la cual sobreescribirá cualquier template global por defecto que se defina):

```
{{ render_hinclude(controller('...'),  {
    'default': 'default/content.html.twig'
}) }}
```

O también puedes especificar un _string_ para mostrar como contenido por defecto:

```
{{ render_hinclude(controller('...'), {'default': 'Loading...'}) }}
```