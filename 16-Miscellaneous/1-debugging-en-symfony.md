El [componente Debug](http://symfony.com/doc/current/components/debug/introduction.html) proporciona herramientas para facilitar la **depuración de código en PHP**.

Cuando trabajas con un **proyecto Symfony** en tu **entorno local**, deberías utilizar el entorno dev (el front controller _app_dev.php_). Esta configuración del entorno es la seleccionada por dos razones principales:

*   Ofrecer al desarrollador feedback preciso siempre que hay algo que va mal (bara web debug, buenas páginas de excepción, profiler...)
*   Ser lo más similar posible al **entorno de producción** para evitar problemas cuando se lance el proyecto.

Para hacer el entorno de producción lo más rápido posible, Symfony crea grandes archivos de cache conteniendo la suma de clases PHP que tu aplicación necesita en cada request. Sin embargo, este comportamiento puede confundir a tu debugger, porque la misma clase puede estar localizada en dos sitios diferentes: el archivo de la clase original y el archivo que agrupa todas las clases.

El siguiente es uno de los métodos posibles para **mejorar la velocidad de carga del entorno de producción en Symfony**:

```
// app_dev.php

$loader = require __DIR__.'/../app/autoload.php';
Debug::enable();

$kernel = new AppKernel('dev', true);
$kernel->loadClassCache();
$request = Request::createFromGlobals();
// ...
```

Para tener al debugger más contento, desactiva la carga de todas las caches de clases PHP removiendo la llamada a _loadClassCache()_:

```
// ...

$loader = require_once __DIR__.'/../app/autoload.php';
Debug::enable();

$kernel = new AppKernel('dev', true);
// $kernel->loadClassCache();
$request = Request::createFromGlobals();
```

Si desactivas las caches PHP, no te olvides de revertirlas después de tu sesión de debugging.

A algunos IDEs no les gusta la idea de que algunas clases estén localizadas en diferentes sitios. Para evitar problemas, ajusta tu IDE para ignorar los archivos o carpetas de caches PHP.

En ocasiones la medida anterior puede no ser suficiente. En [stackoverflow](http://stackoverflow.com/questions/12905404/symfony2-slow-initialization-time) nombraron una serie de medidas que a mi me han ido muy bien ya que a veces el entorno de desarrollo era demasiado lento en mi computadora. Los cambios de directrices son en el [php.ini](https://diego.com.es/configuracion-en-php):

*   Establece `_realpath_cache_size_` = 4096k
*   Desactiva **XDebug** por completo
*   Establece `_realpath_cache_ttl_` = 7200
*   Activa y establece **APC** correctamente
*   Resetea **Apache** para que se carguen los cambios