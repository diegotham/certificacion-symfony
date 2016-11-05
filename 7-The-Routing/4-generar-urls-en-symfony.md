El **sistema de routing** puede usarse también para **generar URLs**. En realidad, routing es un sistema **bidireccional**: mapear la URL a un controller -> parámetros y route -> parámetros de vuelta a una URL. Los métodos _match()_ y _generate()_ forman este sistema bidireccional. 

```
$params = $this->get('router')->match('/blog/my-blog-post');
/* array(
     'slug'        => 'my-blog-post',
     '_controller' => 'AppBundle:Blog:show',
 )
*/
$uri = $this->get('router')->generate('blog_show', array(
    'slug' => 'my-blog-post'
));
// /blog/my-blog-post
```

Para **generar una URL** hay que especificar el nombre de la route (por ejemplo, **blog_show**) y cualquier _wildcard_ (slug = _my-blog-post_) utilizado en la dirección para esa route. Con esta información se puede generar cualquier URL.

```
class MainController extends Controller
{
    public function showAction($slug)
    {
        // ...

        $url = $this->generateUrl(
            'blog_show',
            array('slug' => 'my-blog-post')
        );
    }
}
```

El método _generateUrl()_ definido en la clase **base Controller** es un shortcut del siguiente código:

```
$url = $this->container->get('router')->generate(
    'blog_show',
    array('slug' => 'my-blog-post')
);
```

### Generar URLs en JavaScript

Si tu aplicación front-end utiliza **Ajax requests**, podrías querer **generar URLs en JavaScript** basándose en tu configuración routing. Puedes hacerlo con el [FOSJsRoutingBundle](https://github.com/FriendsOfSymfony/FOSJsRoutingBundle):

```
var url = Routing.generate(
    'blog_show',
    {"slug": 'my-blog-post'}
);
```

### Generar URLs con Query Strings

El método generate toma un array de valores **wildcard** para generar la **URI**. Si pasas valores extra se añadirán a la URI como **query string**:

```
$this->get('router')->generate('blog', array(
    'page' => 2,
    'category' => 'Symfony'
));
// /blog/2?category=Symfony
```

### Generar URLs desde una template

El sitio más común donde generar una URL es desde dentro de una template para enlazar entre páginas de tu **aplicación**. Esto se hace como antes pero empleando una **función helper**:

```
<a href="{{ path('blog_show', {'slug': 'my-blog-post'}) }}">
  Read this blog post.
</a>
```

### Generar URLs absolutas

Por defecto el router generará **URLs relativas** (_/blog_). Desde un controller puedes generar **URLs absolutas** (_http://www.ejemplo.com/blog_) simplemente añadiendo un tercer parámetro al método _generateUrl()_.

```
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;

$this->generateUrl('blog_show', array('slug' => 'my-blog-post'), UrlGeneratorInterface::ABSOLUTE_URL);
// http://www.example.com/blog/my-blog-post
```

Desde una template simplemente usa la función _url()_ (que genera una URL absoluta) antes que la función _path()_ (que genera una URL relativa).

```
<a href="{{ url('blog_show', {'slug': 'my-blog-post'}) }}">
  Read this blog post.
</a>
```

El **host** que se utiliza cuando se genera una URL se detecta automáticamente usando el **objeto Request** actual. Cuando se generen URLs absolutas desde fuera de un contexto web (por ejemplo en un comando de **consola**) esto no funciona. Puedes leer como hacer que funcione [aquí](http://symfony.com/doc/current/cookbook/console/sending_emails.html).