La naturaleza de las **aplicaciones web** es que son **dinámicas**. No importa lo eficiente que sean, cada request siempre contendrá más trabajo para el servidor. Para muchas aplicaciones esto no llega a generar problemas, **Symfony** devolverá las peticiones rápidamente. Pero cuando las aplicaciones se hacen más grandes y el trabajo que se pide al servidor es mayor, se crea la necesidad de emplear **caches**.

La forma más efectiva de **mejorar el rendimiento de una aplicación** es cachear el _output_ de una página y evitar que tenga que pasar por completo por la aplicación en cada request subsecuente.

### Tipos de caches

Los **headers HTTP cache** enviados por una aplicación pueden ser interpretados por tres tipos de caches:

*   **Browser caches**. Cada navegador viene con su propia cache local que es especialmente útil cuando se vuelve a la página anterior o para imágenes u otros _assets_. La cache del navegador es _private_ ya que los resources cacheados no se comparten con nadie.
*   **Proxy caches**. Las proxy caches son compartidas. Las instalan normalmente grandes corporaciones e ISPs para reducir el tráfico de red.
*   **Gateway caches**. Como las proxy, también son caches compartidas pero en el lado del servidor. Hacen que las aplicaciones sean más escalables y eficientes. Se llaman también **reverse proxy caches**, **surrogate caches** o **HTTP accelerators**.

Cada response de una aplicación pasará probablemente por los dos primeros tipos de cache. Estas caches están fuera de nuestro control pero siguen las indicaciones de HTTP cache en sus respuestas. El sistema de cache de Symfony se basa en la simplicidad de la [cache HTTP](http://diego.com.es/cache-http). 

### Gateway cache

Una **gateway cache** (o **reverse proxy**), es una capa independiente frente a una aplicación. El reverse proxy cachea _responses_ devueltas por la aplicación y responde a _requests_ con respuestas cacheadas antes de que lleguen a la aplicación. Symfony proporciona su propio reverse proxy, pero puede utilizarse cualquier otro.

La tarea de la cache es recibir _requests_ del cliente y pasarlos a la aplicación, así como recibir _responses_ de la aplicación y reenviarlas al cliente. La cache es simplemente un intermediario entre los mensajes HTTP. Una de las gateway caches más populares es [Varnish](https://www.varnish-cache.org/).

La **Symfony Reverse Proxy** está escrita en **PHP**, y si está activada cachea las respuestas de la aplicación sin ninguna configuración adicional. Cada aplicación Symfony viene con un un **caching kernel** preconfigurado (_AppCache_) que envuelve el kernel por defecto (_AppKernel_). El caching kernel es el reverse proxy.

Para activar la cache simplemente modificamos el código del **front controller** para utilizar el caching kernel:

```
// web/app.php
use Symfony\Component\HttpFoundation\Request;

// ...
$kernel = new AppKernel('prod', false);
$kernel->loadClassCache();
// wrap the default AppKernel with the AppCache one
$kernel = new AppCache($kernel);

$request = Request::createFromGlobals();

$response = $kernel->handle($request);
$response->send();

$kernel->terminate($request, $response);
```

El caching kernel actuará inmediatamente como un reverse proxy, cacheando respuestas de la aplicación y devolviéndolas al cliente.

El cache kernel tiene un método especial _getLog()_ que devuelve una representación en un string de lo que ha ocurrido en la cache. En en **entorno de desarrollo**, puedes emplearlo para **debugging** y validar la **estrategia de cache**:

```
error_log($kernel->getLog());
```

El objeto **AppCache** tiene configuración por defecto, pero puede configurarse sobreescribiendo el método _getOptions()_:

```
// app/AppCache.php
use Symfony\Bundle\FrameworkBundle\HttpCache\HttpCache;

class AppCache extends HttpCache
{
    protected function getOptions()
    {
        return array(
            'debug'                  => false,
            'default_ttl'            => 0,
            'private_headers'        => array('Authorization', 'Cookie'),
            'allow_reload'           => false,
            'allow_revalidate'       => false,
            'stale_while_revalidate' => 2,
            'stale_if_error'         => 60,
        );
    }
}
```

Las opciones principales son las siguientes:

*   **default_ttl**. El número de segundos que una entrada de cache debería considerarse fresh cuando no hay información explícita de la **freshness** en la _response_. Los headers _Cache-Control_ o _Expires_ sobreescriben este valor- Por defecto es 0.
*   **private_headers**. Conjunto de headers request que lanzan un Cache-Control _private_ en _responses_ que no establecen si la response es _public_ o _private_ a través de una directiva Cache-Control. Por defecto es _Authorization_ y _Cookie._
*   **allow_reload**. Especifica si el cliente puede forzar a recargar la cache incluyendo la directiva Cache-Control _no-cache_ en el request. Por defecto es false.
*   **allow_revalidate**. Especifica si el cliente puede forzar a revalidar una cache incluyendo la directiva _max-age=0_ en el request. Por defecto es false.
*   **stale_while_revalidate**. Especifica el número de segundos por defecto durante los cuales la cache puede inmediatamente revolver una response stale mientras la revalida internamente. Por defecto es 2\. Esta configuración es sobreescrita por la extensión HTTP Cache-Control _stale-while-revalidate_.
*   **stale_if_error**. Especifica el número de segundos por defecto durante los cuales la cache puede servir una stale response cuando se encuentra con un error. Por defecto es 60\. Esta configuración es sobreescrita por la extensión HTTP Cache-Control _stale-if-error_.

Si _debug_ es true, Symfony añade automáticamente el header _X-Symfony-Cache_ a la _response_ conteniendo información acerca de los **hits** y **misses** de la cache.

#### Cambiar de un reverse proxy a otro

La **Symfony Reverse Proxy** es una herramienta muy buena cuando no se puede instalar algo en un servidor más allá de código PHP. Pero al estar escrita en PHP, no puede ser tan rápida como una proxy escrita en **C**. Es por ello por lo que **Symfony recomienda emplear Varnish o Squid** en entornos de producción si es posible. Lo bueno es que cambiar de un servidor proxy a otro es fácil y transparente ya que no hace falta ningún cambio en la aplicación. Lo mejor es comenzar con la proxy de Symfony y después pasarse a Varnish cuando el tráfico incremente.

### HTTP Cache

Los **headers HTTP cache** se usan para comunicarse con la **gateway cache** y cualquier otras caches entre la aplicación y el cliente. Symfony proporciona una potente interfaz para interactuar con los headers. Para aprovechar las capas de cache, una aplicación debe poder comunicarse con las responses cacheables y con las reglas de cómo y cuando la cache debe ser _stale_. Esto se hace estableciendo headers HTTP cache en la response.

HTTP no es más que un lenguaje de texto que los clientes y servidores utilizan para comunicarse entre ellos. HTTP caching es la parte del lenguaje que permite a los clientes y servidores intercambiarse información relacionada con el cacheo. HTTP especifica cuatro cache headers de respuesta: Cache-Control, Expires, ETag y Last-Modified. 

El más importante y versátil es el header **Cache-Control**, que es una colección de varias informaciones de la cache. Cada información va separada por una coma:

```
Cache-Control: private, max-age=0, must-revalidate

Cache-Control: max-age=3600, must-revalidate
```

Symfony proporciona una abstracción alrededor del header Cache-Control para hacer que sea más manejable su creación:

```
// ...

use Symfony\Component\HttpFoundation\Response;

$response = new Response();

// marcar la response como public o private
$response->setPublic();
$response->setPrivate();

// establecer la max age private o shared
$response->setMaxAge(600);
$response->setSharedMaxAge(600);

// establecer una directiva Cache-Control personalizada
$response->headers->addCacheControlDirective('must-revalidate', true);
```

Si necesitas establecer **cache headers** para diferentes acciones de controllers, puedes emplear el [FOSHttpCacheBundle](https://github.com/FriendsOfSymfony/FOSHttpCacheBundle). Proporciona una forma de definir cache headers basándose en el patrón URL y otras propiedades del request.

#### Public vs Private responses

Las caches **gateway** y **proxy** se consideran **compartidas** ya que el contenido de la cache se comparte con más de un usuario. Si una respuesta específica para un usuario es guardada en la cache de forma errónea, puede devolverse después a otros usuarios. Esto puede ser peligroso con información personal.

Para manejar esta situación, cada response debe establecerse como _public_ o _private_:

*   **public**. Indica que la respuesta puede cachearse por caches privadas y públicas.
*   **private**. Indica que todo o parte del contenido de la respuesta es sólo para un usuario y no debe cachearse en cache compartida. 

Por defecto cada response de Symfony es private. Para aprovechar caches compartidas (como la Symfony reverse proxy), la response debe establecerse como public.

#### Safe Methods

**HTTP caching** sólo funciona para **métodos HTTP seguros** (como **GET** y **HEAD**). Seguros significa que nunca se puede cambiar el estado de la aplicación en el servidor cuando se sirve un request (si que puedes hacer cosas como logear información, datos de cache, etc). Esto tiene dos consecuencias:

*   **Nunca** se ha de cambiar el **estado de la aplicación cuando se responde a GET o HEAD requests**. Incluso si no se utiliza una gateway cache, la presencia de proxy caches significa que cualquier request HEAD o GET podría o no llegar realmente a tu servdor.
*   **No** esperes **que se cacheen métodos PUT, POST o DELETE**. Se supone que estos métodos se emplean para modificar el estado de la aplicación (como eliminar un post de un blog). Cachearlos podría evitar que ciertos requests alcanzaran la aplicación.

#### Caching Rules y Defaults

**HTTP 1.1** permite cachear cualquier cosa por defecto a no ser que haya un header **Cache-Control**. En la práctica, la mayoría de caches no hacen nada cuando los requests tienen una **cookie**, un header **authorization**, usan un **método no seguro** (como POST, PUT o DELETE) o cuando las responses tienen un **código de estado de redirección**.

Symfony automáticamente establece un header Cache-Control cuando el desarrollador no establece ninguno, siguiendo las siguientes normas:

*   Si no hay definido un cache header (Cache-Control, Expires, ETag o Last-Modified), Cache-Control se establece a _no-cache_, lo que significa que la respuesta no será cacheada.
*   Si Cache-Control está vacío (pero está presente uno de los otros cache headers), su valor se establece a _private_, _must-revalidate_.
*   Pero si al menos una directiva Cache-Control está establecida, y no se han añadido directivar _public_ o _private_, Symfony añade la directiva _private_ automáticamente (excepto cuando está establecido _s-maxage_).