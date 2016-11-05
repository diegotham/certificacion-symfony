Las **gateway caches** son una muy buena forma de hacer que una aplicación tenga mejor rendimiento. Pero tienen una limitación: sólo pueden cachear páginas enteras. Si no puedes cachear páginas enteras o si una parte de una página tiene una zona más dinámica, no sirven. 

**Symfony** proporciona una solución para estos casos, basándose en la **tecnología ESI**, [Edge Side Includes](http://www.w3.org/TR/esi-lang).

La especificación ESI describe **tags** que se pueden inscrutar en páginas para comunicarse con la gateway cache. En Symfony sólo se implementa una tag, _include_:

```
<!DOCTYPE html>
<html>
    <body>
        <!-- ... some content -->

        <!-- Embed the content of another page here -->
        <esi:include src="http://..." />

        <!-- ... more content -->
    </body>
</html>
```

Cada **ESI tag** tiene una **URL fully qualified**. Representa un fragmento de página que puede obtenerse a través de la URL.

Cuando se maneja un _request_, la gateway cache obtiene la página entera desde su cache o la solicita de la aplicación. Si la respuesta contiene una o más ESI tags, se procesan de la misma forma. La **gateway cache o devuelve el fragmento de página cacheado o solicita el fragmento a la aplicación**. Cuando se han resuelto todas las ESI tags, la gateway cache las une en la página principal y envía el contenido final al cliente.

Todo esto ocurre de forma transparente en el nivel de la cache gateway (fuera de la aplicación). Si aprovechas las ESI tags, **Symfony** realiza el proceso de incluirlas casi sin esfuerzo.

### Utilizar ESI en Symfony

Para emplear ESI primero ha de estar activada en la configuración:

```
# app/config/config.yml
framework:
    # ...
    esi: { enabled: true }
```

Ahora suponemos que tenemos una página que es relativamente estática, excepto por un bloque de últimas noticias al final del contenido. Con **ESI** podemos cachear el bloque independientemente de la página:

```
// src/AppBundle/Controller/DefaultController.php

// ...
class DefaultController extends Controller
{
    public function aboutAction()
    {
        $response = $this->render('static/about.html.twig');
        // establece el max age - y también marca la respuesta como public
        $response->setSharedMaxAge(600);

        return $response;
    }
}
```

En este ejemplo la página cacheada tiene una vida limitada de diez minutos. Lo siguiente es incluir el bloque de noticias en la template incrustándolas. Esto se hace mediente el helper _render_.

Ya que el contenido incrustado viene de otra página (aquí controller), Symfony emplea el helper _render_ para configurar ESI tags:

```
{# app/Resources/views/static/about.html.twig #}

{# puedes usar una controller reference #}
{{ render_esi(controller('AppBundle:News:latest', { 'maxPerPage': 5 })) }}

{# ... o una URL #}
{{ render_esi(url('latest_news', { 'maxPerPage': 5 })) }}
```

Utilizando el ESI renderer (a través de la función Twig _render_esi_), le dices a Symfony que la acción debe renderizarse como una **ESI tag**. La razón por la que se usa el helper es para que la aplicación funcione incluso si no hay gateway cache instalada.

La variable _maxPerPage_ está disponible como argumento en el controller (_$maxPerPage_). Las variables que se pasan a través de _render_esi_ también se convierten en parte de la cache key de forma que tienes caches únicas para cada combinación de variables y valores.

Cuando se usa la función _render_ por defecto (o se establece el renderer a _inline_), **Symfony** une el contenido de la página en el principal antes de enviar la respuesta al cliente. Pero si usas un **ESI renderer** (llamada a _render_esi_) y Symfony detecta que está comunicándose con una **gateway cache que soporta ESI**, genera una **ESI include tag**. Pero si no hay gateway cache o si no soporta ESI, Symfony simplemente unirá el contenido de la página incluída dentro del contenido de la principal como si hubiera usado _render_.

La acción incrustada puede especificar ahora sus propias normas de caching, totalmente independientes de la página principal:

```
// src/AppBundle/Controller/NewsController.php
namespace AppBundle\Controller;

// ...
class NewsController extends Controller
{
    public function latestAction($maxPerPage)
    {
        // ...
        $response->setSharedMaxAge(60);

        return $response;
    }
}
```

Con **ESI**, la cache de la página entera será válida por 600 segundos, pero la cache del componente news sólo durará 60 segundos.

Cuando se usa una _controller reference_, la **ESI tag** debería referenciar a la acción incrustada como una URL accesible de forma que la gateway cache pueda unirla independientemente del resto de la página. Symfony se encarga de generar una URL única para cualquier _controller reference_ y puede enrutarlas bien gracias al [FragmentListener](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/EventListener/FragmentListener.html) que debe activarse en la configuración:

```
# app/config/config.yml
framework:
    # ...
    fragments: { path: /_fragment }
```

Una gran ventaja del **ESI renderer** es que se puede hacer a la aplicación tan dinámica como se necesite y al mismo tiempo cargar la aplicación lo menos posible.

Cuando utilices **ESI** recuerda utilizar la directiva _s-maxage_ en lugar de _max-age_. El navegador sólo recibe el **resource agregado**, por lo que no es consciente de los subcomponentes, y se atendrá a la directiva _max-age_ y cacheará la página entera.

El helper _render_esi_ soporta también otras dos opciones:

*   **alt**. Se emplea como el atributo _alt_ de una ESI tag, que permite **especificar una URL alternativa** a usarse si no se puede encontrar la _src_.
*   **ignore_errors**. Si se establece a true, un atributo _onerror_ se añadirá al ESI con un valor de _continue_ indicando que, en cada error, la gateway cache simplemente removerá la ESI tag silenciosamente.