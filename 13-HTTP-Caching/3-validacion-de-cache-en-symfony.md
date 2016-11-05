Cuando un resource necesita actualizarse tan pronto como se realiza un cambio, el **modelo de expiración** no sirve. Con el modelo de expiración, la aplicación no responderá a una petición hasta que la cache sea _stale_. 

El **modelo de validación** trata esta situación. Con este modelo, la cache continúa guardando _responses_. La diferencia es que para cada _request_, la cache pregunta a la aplicación si la _response_ cacheada es todavía válida o si necesita ser regenerada. Si la cache es todavía válida, la aplicación debería devolver un [código de estado 304](http://diego.com.es/codigos-de-estado-http) sin contenido. Esto le dice a la cache que puede devolver la respuesta cacheada.

Bajo este modelo sólo ahorras **CPU** si puedes determinar que la respuesta cacheada es todavía válida haciendo menos trabajo que generando la página entera otra vez.

### Validación con el header ETag

El header **ETag** es un string (llamado el "_entity-tag_") que identifica de forma única una representación del resource objetivo. Lo genera y establece la aplicación de forma que pueda comprobar, por ejemplo, si el resource _/about_ guardado en la cache está al día con lo que la aplicación devolvería. Una **ETag** es una identificación utilizada para comparar si dos versiones diferentes de un resource son equivalentes. Cada identificación debe ser única entre las representaciones del mismo resource.

Una simple implementación:

```
// src/AppBundle/Controller/DefaultController.php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Request;

class DefaultController extends Controller
{
    public function homepageAction(Request $request)
    {
        $response = $this->render('static/homepage.html.twig');
        $response->setETag(md5($response->getContent()));
        $response->setPublic(); // aseguramos que la respuesta es cacheable públicamente
        $response->isNotModified($request);

        return $response;
    }
}
```

El método _isNotModified()_ compara el _If-None-Match_ enviado en el request con el header **ETag** establecido en la respuesta. Si los dos coinciden, el método automáticamente establece el código de respuesta a **304**.

La cache establece el header _If-None-Match_ en el request en el **ETag** de la respuesta original cacheada antes de enviar el _request_ a la aplicación. Así es como la cache y el servidor se comunican y deciden si el resource se ha actualizado desde que fue cacheado.

El algoritmo es simple y genérico, pero necesitas crear la respuesta entera antes de establecer el ETag, lo que hace que se ahorre en ancho de banda pero no en ciclos CPU. Después veremos cómo puede emplearse mejor la validación.

### Validación con el header Last-Modified

El header _Last-Modified_ es la segunda forma de validación. Según la **especificación HTTP**, "El header _Last-Modified_ indica la fecha y hora en la cual el servidor de origen cree que la representación se ha modificado". En otras palabras, la aplicación decide si el contenido cacheado se ha actualizado basándose en si se ha modificado desde que la respuesta fuera cacheada.

Por ejemplo, puedes usar la fecha de última modificación para todos los objetos que necesiten establecer la representación del resource como el valor del header _Last-Modified_:

```
// src/AppBundle/Controller/ArticleController.php
namespace AppBundle\Controller;

// ...
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use AppBundle\Entity\Article;

class ArticleController extends Controller
{
    public function showAction(Article $article, Request $request)
    {
        $author = $article->getAuthor();

        $articleDate = new \DateTime($article->getUpdatedAt());
        $authorDate = new \DateTime($author->getUpdatedAt());

        $date = $authorDate > $articleDate ? $authorDate : $articleDate;

        $response = new Response();
        $response->setLastModified($date);
        // Establecer la respuesta como public. Sino será private por defecto.
        $response->setPublic();

        if ($response->isNotModified($request)) {
            return $response;
        }

        // ... hacer más tareas para rellenar la response con el contenido entero

        return $response;
    }
}
```

El método _isNotModified()_ compara el header _If-Modified-Since_ enviado por el request con el header _Last-Modified_ establecido en la _response_. Si son equivalentes, la respuesta se establecerá al **código de estado 304**. 

La cache establece el header _If-Modified-Since_ en el request para el _Last-Modified_ de la respuesta cacheada antes de enviar el request a la aplicación. Así es como se comunican la cache y el servidor y deciden si el resource se ha actualizado desde que fuera cacheado.

### Optimizar el código con validación

El principal objetivo de cualquier estrategia de cacheo es **aliviar la carga de la aplicación**. Cuanto menos hagamos en la aplicación creando una **respuesta 304 desde la aplicación**, mejor. El método _Response::isNotModified()_ hace exactamente eso con un patrón simple y eficiente:

```
// src/AppBundle/Controller/ArticleController.php
namespace AppBundle\Controller;

// ...
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;

class ArticleController extends Controller
{
    public function showAction($articleSlug, Request $request)
    {
        // Obtener la mínima información para establecer
        // el valor ETag o Last-Modified
        // (basado en el Request, los datos se extraen de
        // una base de datos o en un store key-value, por ejemplo)
        $article = ...;

        // crear una Response con una ETag y/o un header Last-Modified
        $response = new Response();
        $response->setETag($article->computeETag());
        $response->setLastModified($article->getPublishedAt());

        // Establecer la respuesta como public. Sino será private por defecto.
        $response->setPublic();

        // Comprobar que la Response no está modificada en el actual Request
        if ($response->isNotModified($request)) {
            // devolver una 304 Response inmediatamente
            return $response;
        }

        // hacer más cosas aquí, como devolver más datos
        $comments = ...;

        // o renderizar una template con la $response que ya hemos comenzado
        return $this->render('article/show.html.twig', array(
            'article' => $article,
            'comments' => $comments
        ), $response);
    }
}
```

Cuando la **Response** no se ha modificado, el _isNotModified()_ automáticamente establece el código de estado de respuesta a 304, y remueve el contenido y algunos headers que no deben estar presentes en respuestas 304.

### Variar la respuesta

Hasta ahora hemos asumido que cada **URI** tiene exactamente una representación del resource objetivo. Por defecto, **HTTP caching** utiliza el URI del resource como la key de la cache. Si dos personas solicitan la misma URI de un resource cacheable, la segunda persona recibirá la versión cacheada.

A veces esto no es suficiente y diferentes versiones de la misma URI necesitan ser cacheadas basándose en uno o más valores de **header requests**. Por ejemplo, si comprimes páginas cuando el cliente lo soporta, cualquier URI tiene dos representaciones: una cuando el cliente soporta compresión y otra cuando no. Esta determinación se hace con el valor del **request header** _Accept-Encoding_.

En este caso necesitamos guardar la versión comprimida y la no comprimida de la respuesta del URI en particular y devolverlos basándonos en el valor del header _Accept-Encoding_. Esto se hace empleando el **response header** _Vary_, que es una lista de headers separados por coma cuyos valores lanzan una representación diferente del resource solicitado:

```
Vary: Accept-Encoding, User-Agent
```

El header _Vary_ cacheará diferentes versiones de cada resource basándose en el URI y en el valor del header request _Accept-Encoding_ y _User-Agent_.

El **objeto Response** ofrece una interfaz limpia para manejar el header Vary:

```
// establecer un header vary
$response->setVary('Accept-Encoding');

// establecer varios headers vary
$response->setVary(array('Accept-Encoding', 'User-Agent'));
```

El método setVary() toma el nombre de un header o un array de headers para los cuales la response varía.

### Expiración y validación

Podemos emplear tanto el **modelo de expiración** como el **modelo de validación** en el mismo **Response**. Expiración gana frente a validación, pero puedes beneficiarte fácilmente de los dos modelos. Usando ambos puedes mostrar el contenido cacheado a la vez que compruebas su validez.

Además de los métodos que ya hemos visto, **Symfony** proporciona otros métodos para manipular la cache:

```
// Marca la respuesta como stale
$response->expire();

// Fuerza la respuesta a devolver un código 304 sin contenido
$response->setNotModified();
```

Además, la mayoría de headers HTTP relacionados con la cache pueden establecerse a través del método _setCache()_:

```
// Establece la configuración de cache de vez
$response->setCache(array(
    'etag'          => $etag,
    'last_modified' => $date,
    'max_age'       => 10,
    's_maxage'      => 10,
    'public'        => true,
    // 'private'    => true,
));
```

### Invalidación de la cache

Una vez que una URL es cacheada por una **cache gateway**, la cache ya no solicitará a la aplicación ese contenido. Esto permite a la cache proporcionar respuestas rápidas y reduce la carga de la aplicación. Sin embargo, te arriesgas a devolver contenido caducado. Una forma de solucionar esto es utilizar largos periodos de vida de cache, pero notificar activamente a la gateway cache cuando el contenido cambia. Las **reverse proxies** normalmente proporcionan un canal para recibir esas notificaciones, típicamente a través de **requests HTTP**.

De todas formas **la invalidación de cache se ha de evitar siempre que sea posible**. Si cometes un **error** al invalidar algo, las caches caducadas se servirán durante bastante tiempo. En su lugar, **emplea cortos periodos de vida de cache** o utiliza el **modelo de validación**, y ajusta los controllers para realizar comprobaciones de validación eficientes. Además, debido a que la invalidación es un tema específico de cada reverse proxy, utilizar este concepto te ataría a un reverse proxy específico o tendrías que ingeniártelas para soportar diferentes proxies.

Sim embargo a veces puede que se necesite ese extra de rendimiento que se consigue con la **invalidación**. Para la invalidación, la aplicación tiene que detectar cuando cambia el contenido y decirle a la cache que elimine las URLs que contienen esos datos de la cache. 

Para la **cache invalidation** puedes emplear el bundle [FOSHttpCacheBundle](https://github.com/FriendsOfSymfony/FOSHttpCacheBundle). Proporciona services para ayudar con varios conceptos de cache invalidation y también documenta la configuración para varias caching proxies.

Si el contenido corresponde a una URL, el modelo **PURGE** funciona bien. Envías un request a la cache proxy con el método **HTTP PURGE** (utilizar la palabra "PURGE" es una convenxión, ténicamente puede ser cualquier string) en lugar de GET y haces que la cache proxy lo detecte y remueva los datos de la cache en lugar de ir a la aplicación para obtener una _response_.

Así se puede configurar la **Symfony reverse proxy** para soportar el método HTTP PURGE:

```
// app/AppCache.php

use Symfony\Bundle\FrameworkBundle\HttpCache\HttpCache;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
// ...

class AppCache extends HttpCache
{
    protected function invalidate(Request $request, $catch = false)
    {
        if ('PURGE' !== $request->getMethod()) {
            return parent::invalidate($request, $catch);
        }

        if ('127.0.0.1' !== $request->getClientIp()) {
            return new Response(
                'Invalid HTTP method',
                Response::HTTP_BAD_REQUEST
            );
        }

        $response = new Response();
        if ($this->getStore()->purge($request->getUri())) {
            $response->setStatusCode(200, 'Purged');
        } else {
            $response->setStatusCode(200, 'Not found');
        }

        return $response;
    }
}
```

Se ha de proteger el método HTTP PURGE de alguna forma para evitar que se pueda purgar el contenido cacheado desde cualquier lado.

**Purge** le dice a la cache que libere un resource en todas sus variantes (según el header _Vary_). Una alternativa a esto es refrescar el contenido. **Refrescar** es que se le diga a la **caching proxy** que descarte su cache local y obtenga el contenido de nuevo. De esta forma el nuevo contenido estará disponible en la cache. Lo malo de esto es que las variantes no son invalidadas.

En muchas aplicaciones el mismo contenido parcial se usa en varias páginas con diferentes URLs. Existen conceptos más flexibles en estos casos:

*   **Banning**. Invalida las responses que coinciden con **expresiones regulares** en una URL u otro criterio.
*   **Cache tagging**. Te permite añadir un **tag** para cada contenido usado en una _response_ de forma que puedas invalidar todas las URLs que contienen cierto contenido.