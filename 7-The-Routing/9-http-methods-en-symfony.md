El **método HTTP de un request** es una de las cosas que Symfony comprueba cuando se evalúa el request para encontrar una route. Esto puede configurarse mediante cualquiera de las formas de configuración disponibles en Symfony (**anotaciones**, **YAML**, **XML** o **PHP**). Suponiendo que tenemos una aplicación blog con dos controladores: uno para mostrar un post (aceptando GET y HEAD request) y otro para procesar el formulario (POST request), la configuración de los métodos con anotaciones puede hacerse como sigue:

```
// src/AppBundle/Controller/MainController.php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Method;
// ...

class BlogApiController extends Controller
{
    /**
     * @Route("/api/posts/{id}")
     * @Method({"GET","HEAD"})
     */
    public function showAction($id)
    {
        // ... return a JSON response with the post
    }

    /**
     * @Route("/api/posts/{id}")
     * @Method("PUT")
     */
    public function editAction($id)
    {
        // ... edit a post
    }
}
```

A pesar del hecho de que estas dos routes tienen directorios idénticos (_/api/posts/{id}_), la primera route coincidirá sólo con **GET** o **HEAD** requests y la segunda route coincidirá sólo con **PUT** requests. Esto significa que puedes mostrar el post y editarlo a través de la misma **URL**, utilizando distintos controllers para las dos acciones. Si no se especifica ningún método, la route coincidirá con todos los métodos.

Se puede utilizar cualquier otro **verbo HTTP** de la misma forma. Si tenemos una entrada de blog, desde la misma URL podemos **obtenerla**, **modificarla** o **eliminarla**:

```
blog_show:
    path: /blog/{slug}
    defaults: {_controller:AppBundle:Blog:show}
    methods: [GET]
blog_update:
    path: /blog/{slug}
    defaults: {_controller: AppBundle:Blog:update}
    methods: [PUT]
blog_delete:
    path: /blog/{slug}
    defaults: {_controller: AppBundle:Blog:delete}
    methods: [DELETE]
```

La mayoría de navegadores no permiten enviar **PUT** y **DELETE** requests en un formulario **HTML**. Afortunadamente, Symfony proporciona una manera simple de superar esta limitación. Incluyendo el parámetro **_method** en el query string o en los parámetros de un HTTP request, Symfony podrá saber el método deseado cuando busque routes. Los formularios automáticamente incluyen un campo escondido para este parámetro si el método de sumisión no es GET o POST.