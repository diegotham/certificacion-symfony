Tenemos dos controllers en la clase **BlogController**:

```
// src/AppBundle/Controller/BlogController.php

// ...
class BlogController extends Controller
{
    /**
     * @Route("/blog/{page}", defaults={"page" = 1})
     */
    public function indexAction($page)
    {
        // ...
    }

    /**
     * @Route("/blog/{slug}")
     */
    public function showAction($slug)
    {
        // ...
    }
}
```

Ambas routes tienen patrones que hacen coincidir URLs como _/blog/*_. El routing de Symfony siempre hará coincidir la primera route que encuentre. La route **blog_show** nunca coincidirá. Una URL como _/blog/my-blog-post_ coincidirá con la primera route y devolverá un valor _my-blog-post_ al parámetro _{page}_.

Para solucionar esto podemos añadir restricciones a las routes. Las routes en este ejemplo funcionarán perfectamente si el placeholder de _/blog/{page}_ sólo coincidiera con _integers_. Restricciones con [expresiones regulares](http://diego.com.es/expresiones-regulares-en-php) pueden añadirse en cada parámetro:

```
// src/AppBundle/Controller/BlogController.php

// ...

/**
 * @Route("/blog/{page}", defaults={"page": 1}, requirements={
 *     "page": "\d+"
 * })
 */
public function indexAction($page)
{
    // ...
}
```

El requisito _\d+_ es una **expresión regular** que indica que el valor del parámetro _{page}_ debe ser un número. Ahora una route como _my-blog-post_  sólo coincidirá con la segunda route _/blog/{slug}_.

El orden de las routes es muy importante. Si se cambiara el orden en que están definidas las routes anteriores, la URL _/blog/2_ coincidiría con _/blog/{slug}_ y nunca llegaría a _/blog/{page}_. Con un orden aduecuado y el uso de restricciones se puede conseguir lo que se quiera.

Ya que los requisitos están descritos con **expresiones regulares**, la complejidad y flexibilidad de cada requisitio depende de ti. Suponemos que la homepage de una aplicación está disponible en dos idiomas diferentes, basándose en la URL:

```
// src/AppBundle/Controller/MainController.php

// ...
class MainController extends Controller
{
    /**
     * @Route("/{_locale}", defaults={"_locale": "en"}, requirements={
     *     "_locale": "en|fr"
     * })
     */
    public function homepageAction($_locale)
    {
    }
}
```

La aplicación responderá con esta route si se trata de _en_ o _fr_. Si se intenta con _es_ no responderá esta route.