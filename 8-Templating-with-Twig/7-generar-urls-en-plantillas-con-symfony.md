**Crear enlaces a otras páginas** es una de las tareas más comunes en una template de una **aplicación**. En lugar de escribir las URLs directamente, puedes emplear la función _path_ de **Twig** para generar las URLs basándote en la **configuración del routing**. Después, si quieres modificar la URL de una página en particular, todo lo que tienes que hacer es cambiar la configuración del routing, las templates generarán automáticamente la nueva URL.

Primero, creamos un nombre a la página principal, "__welcome_", que será accesible con la siguiente configuración del routing:

```
// src/AppBundle/Controller/WelcomeController.php

// ...
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class WelcomeController extends Controller
{
    /**
     * @Route("/", name="_welcome")
     */
    public function indexAction()
    {
        // ...
    }
}
```

Para enlazar a la página, simplemente utiliza la función Twig _path_ y haz referencia a la route:

```
<a href="{{ path('_welcome') }}">Home</a>
```

En una route más complicada que acepta parámetros:

```
// src/AppBundle/Controller/ArticleController.php

// ...
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class ArticleController extends Controller
{
    /**
     * @Route("/article/{slug}", name="article_show")
     */
    public function showAction($slug)
    {
        // ...
    }
}
```

Hay que especificar, además del nombre de la route, un valor para el parámetro {_slug_}.

```
{# app/Resources/views/article/recent_list.html.twig #}
{% for article in articles %}
    <a href="{{ path('article_show', {'slug': article.slug}) }}">
        {{ article.title }}
    </a>
{% endfor %}
```

También puedes generar una URL absoluta con la función url:

```
<a href="{{ url('_welcome') }}">Home</a>
```