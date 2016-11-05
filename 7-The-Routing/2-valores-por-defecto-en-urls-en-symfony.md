Suponemos que tenemos una route que muestra una lista con todos los posts de una **aplicación blog**:

```
// src/AppBundle/Controller/BlogController.php

// ...
class BlogController extends Controller
{
    // ...

    /**
     * @Route("/blog")
     */
    public function indexAction()
    {
        // ...
    }
}
```

Esta route es lo más sencillo posible, no tiene placeholders y sólo coincide con la URL exacta _/blog_. Pero si ahora por ejemplo queremos que pueda soportar paginación, tendremos que añadir un placeholder _{page}_ para que puedan coincidir routes como _/blog/2_:

```
// src/AppBundle/Controller/BlogController.php

// ...

/**
 * @Route("/blog/{page}")
 */
public function indexAction($page)
{
    // ...
}
```

El valor de _{page}_ estará disponible en el controller. Su valor puede emplearse para determinar el conjunto de entradas de blog a mostrar en la página dada. Pero los placeholders son requeridos por defecto, esta route no coincidirá ya con _/blog_. Para ver la página 1 tendríamos que utilizar la URL /blog/1.

Para solucionar este podemos hacer el parámetro _{page}_ opcional, lo que se consigue con _defaults_:

```
// src/AppBundle/Controller/BlogController.php

// ...

/**
 * @Route("/blog/{page}", defaults={"page" = 1})
 */
public function indexAction($page)
{
    // ...
}
```

Añadiendo _page_ a la key _defaults_ el placeholder _{page}_ ya no es **requerido**. La URL /blog coincidirá con esta route y el valor del parámetro _page_ será 1\. La URL _/blog/2_ también coincidirá, mostrando la segunda página de resultados.

Se pueden tener **más de un placeholder opcional** (_/blog/{slug}/{page}_), pero todo lo de después de un placeholder opcional ha de ser opcional. Por ejemplo _/{page}/blog_ es una dirección válida, pero page siempre será requerido (_/blog_ no coincidirá con esta route).

Las routes con parámetros opcionales al final no coincidirán en requests con **trailing slash (/)**: /blog/ no coincidirá, en cambio /blog sí.