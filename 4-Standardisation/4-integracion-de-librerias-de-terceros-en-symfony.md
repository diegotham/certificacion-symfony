El **componente ClassLoader** proporciona herramientas para autocargar tus clases y cachear sus localizaciones para mejorar el rendimiento.

Cuando referencias una clase que no se ha requerido o incluído todavía, PHP utiliza el [mecanismo de autocarga](http://php.net/manual/en/language.oop5.autoload.php) para delegar la carga de un archivo que define la clase. **Symfony** proporciona tres autoloaders que pueden cargar tus clases:

*   El PSR-0 Class Loader. Carga las clases que siguen el estándar de nombres PSR-0.
*   El PSR-4 Class Loader. Carga las clases que siguen el estándar de nombres PSR-4.
*   MapClassLoader. Carga clases empleando un mapa estático desde el nombre de la clase al directorio del archivo.
*   ClassMapGenerator. Guarda todas las clases de un directorio para que después puedan ser cargadas.

Además, el componente ClassLoader viene con una clase **wrapper** que permite [cachear los resultados del class loader](#CachearUnClassLoader).

Si se emplea el [componente Debug](http://symfony.com/doc/current/components/debug/introduction.html), puedes también utilizar el [DebugClassLoader](http://symfony.com/doc/current/components/debug/class_loader.html) que facilite el debugging lanzando excepciones más concretas cuando una clase no se haya podido encontrar por el class loader.

### PSR-0 Class Loader

Si tus clases y librerías de terceros siguen el estándar PSR-0, puedes utilizar la clase ClassLoader para cargar todas las clases de tu proyecto.

Se pueden emplear tanto **ApcClassLoader** como **XcacheClassLoader** para cachear una instancia ClassLoader.

Registrar el autoloader ClassLoader es sencillo:

```
require_once '/path/to/src/Symfony/Component/ClassLoader/ClassLoader.php';

use Symfony\Component\ClassLoader\ClassLoader;

$loader = new ClassLoader();

// to enable searching the include path (eg. for PEAR packages)
$loader->setUseIncludePath(true);

// ... register namespaces and prefixes here - see below

$loader->register();
```

El autoloader se registra automáticamente en una aplicación Symfony (puede verse en app/autoload.php).

Para registrar tus clases puedes emplear [addPrefix()](http://api.symfony.com/3.0/Symfony/Component/ClassLoader/ClassLoader.html#method_addPrefix) o [addPrefixes()](http://api.symfony.com/3.0/Symfony/Component/ClassLoader/ClassLoader.html#method_addPrefixes):

```
// register a single namespaces
$loader->addPrefix('Symfony', __DIR__.'/vendor/symfony/symfony/src');

// register several namespaces at once
$loader->addPrefixes(array(
    'Symfony' => __DIR__.'/../vendor/symfony/symfony/src',
    'Monolog' => __DIR__.'/../vendor/monolog/monolog/src',
));

// register a prefix for a class following the PEAR naming conventions
$loader->addPrefix('Twig_', __DIR__.'/vendor/twig/twig/lib');

$loader->addPrefixes(array(
    'Swift_' => __DIR__.'/vendor/swiftmailer/swiftmailer/lib/classes',
    'Twig_'  => __DIR__.'/vendor/twig/twig/lib',
));
```

Las clases de un subnamespace or una subjerarquía de clases PEAR pueden buscarse en una lista localización para facilitar el vendoring de clases para proyectos grandes:

```
$loader->addPrefixes(array(
    'Doctrine\\Common'           => __DIR__.'/vendor/doctrine/common/lib',
    'Doctrine\\DBAL\\Migrations' => __DIR__.'/vendor/doctrine/migrations/lib',
    'Doctrine\\DBAL'             => __DIR__.'/vendor/doctrine/dbal/lib',
    'Doctrine'                   => __DIR__.'/vendor/doctrine/orm/lib',
));
```

En este ejemplo, si intentas usar una clase en el namespace **Doctrine\Common** o uno de sus hijos, el autoloader buscará primero una clase en el directorio doctrine-common. Si no se encuentra, retrocederá al directorio **Doctrine** (el último configurado) antes de abandonar. El orden de los prefixes es significante en este caso.

### PSR-4 Class Loader

Las librerías que siguen el estándar PSR-4 se pueden cargar con el Psr4ClassLoader.

Si administras tus dependencias con Composer, tienes un autoloader compatible con PSR-4 incorporado. Utiliza este loader en entornos en los que Composer no está disponible.

Todos los componentes Symfony siguen el estándar PSR-4.

Por ejemplo si tenemos los componentes **ClassLoader** y **Yaml** descomprimidos en el directorio _/libs_, la estructura del directorio es:
```
libs/
    ClassLoader/
        Psr4ClassLoader.php
        ...
    Yaml/
        Yaml.php
        ...
config.yml
demo.php
```

En demo.php var a cargar el archivo config.yml. Para hacerlo, primero necesitas configurar el **Psr4ClassLoader**:

```
use Symfony\Component\ClassLoader\Psr4ClassLoader;
use Symfony\Component\Yaml\Yaml;

require __DIR__.'/lib/ClassLoader/Psr4ClassLoader.php';

$loader = new Psr4ClassLoader();
$loader->addPrefix('Symfony\\Component\\Yaml\\', __DIR__.'/lib/Yaml');
$loader->register();

$data = Yaml::parse(file_get_contents(__DIR__.'/config.yml'));
```

El class loader se carga manualmente con la sentencia _require_, ya que todavía no hay un mecanismo de autoload. Con la llamada a _addPrefix()_, le dices al class loader donde buscar clases con el namespace prefix _Symfony\Component\Yaml\_. Después de registrar el autoloader, el componente Yaml está listo para usarse.

### MapClassLoader

El **MapClassLoader** te permite autocargar clases con un mapa estático desde clases a archivos. Esto es útil si usas librerías de terceros que no siguen los **estándares PSR-0** y no pueden usar el **PSR-0 class loader**.

El MapClassLoader puede usarse con el **PSR-0** class loader configurando y llamando al método _register()_ en ambos.

El comportamiento por defecto es añadir el MapClassLoader en la pila de autoload. Si quieres usarlo como el primer autoloader, añade true cuando llames al método _register()_. Tu class loader se añadirá a la pila de autocarga.

Para usarlo simplemente hay que pasar el mapa a su constructor cuando se cree la instancia de la clase MapClassLoader:

```
require_once '/path/to/src/Symfony/Component/ClassLoader/MapClassLoader.php';

$mapping = array(
    'Foo' => '/path/to/Foo',
    'Bar' => '/path/to/Bar',
);

$loader = new MapClassLoader($mapping);

$loader->register();
```

### Cachear un Class Loader

Encontrar el archivo para una clase en concreto puede ser una tarea costosa. Por suerte, el componente ClassLoader viene con dos clases para cachear el mapeo de una clase a su archivo correspondiente. Ambas **ApcClassLoader** y **XcacheClassLoader** envuelven un objeto que implementa el método _findFile()_ para encontrar el archivo de una clase.

ApcClassLoader y XcacheClassLoader pueden usarse para cachear el autoloader de Composer.

#### ApcClassLoader

ApcClassLoader envuelve una clase existente y cachea llamadas a su método _findFile()_ con APC:

```
require_once '/path/to/src/Symfony/Component/ClassLoader/ApcClassLoader.php';

// instance of a class that implements a findFile() method, like the ClassLoader
$loader = ...;

// sha1(__FILE__) generates an APC namespace prefix
$cachedLoader = new ApcClassLoader(sha1(__FILE__), $loader);

// register the cached class loader
$cachedLoader->register();

// deactivate the original, non-cached loader if it was registered previously
$loader->unregister();
```

#### XcacheClassLoader

XcacheClassLoader utiliza XCache para cachear un class loader:

```
require_once '/path/to/src/Symfony/Component/ClassLoader/XcacheClassLoader.php';

// instance of a class that implements a findFile() method, like the ClassLoader
$loader = ...;

// sha1(__FILE__) generates an XCache namespace prefix
$cachedLoader = new XcacheClassLoader(sha1(__FILE__), $loader);

// register the cached class loader
$cachedLoader->register();

// deactivate the original, non-cached loader if it was registered previously
$loader->unregister();
```

### ClassMapGenerator

Hoy en día las **librerías PHP** vienen con soporte a la autocarga a través de **Composer**. Pero en ocasiones es posible encontrar librerías de terceros que vienen sin soporte al autoloading y es necesario cargar cada clase manualmente. Si por ejemplo tenemos una librería con la siguiente estructura:
```
library/
├── bar/
│   ├── baz/
│   │   └── Boo.php
│   └── Foo.php
└── foo/
    ├── bar/
    │   └── Foo.php
    └── Bar.php
```

Estos archivos contienen las siguientes clases:

| | |
| -------- | -------- |
| **Archivo** | **Nombre de clase** |
| library/bar/baz/Boo.php | Acme\Bar\Baz |
| library/bar/Foo.php | Acme\Bar |
| library/foo/bar/Foo.php | Acme\Foo\Bar |
| library/foo/Bar.php | Acme\Foo |

El componente ClassLoader viene con una clase **ClassMapGenerator** que hace posible crear un mapa de nombres de clases a archivos. Para generar el mapa de clases se pasa el directorio de los archivos al método _createMap()_:

```
use Symfony\Component\ClassLoader\ClassMapGenerator;

var_dump(ClassMapGenerator::createMap(__DIR__.'/library'));
```

Dados los archivos y clases de la tabla anterior, deberías ver el siguiente output:

```
Array
(
    [Acme\Foo] => /var/www/library/foo/Bar.php
    [Acme\Foo\Bar] => /var/www/library/foo/bar/Foo.php
    [Acme\Bar\Baz] => /var/www/library/bar/baz/Boo.php
    [Acme\Bar] => /var/www/library/bar/Foo.php
)
```

El **ClassMapGenerator** también proporciona el método _dump()_ para guardar el mapa de clases generado en el sistema:

```
use Symfony\Component\ClassLoader\ClassMapGenerator;
ClassMapGenerator::dump(__DIR__.'/library', __DIR__.'/class_map.php');
```

Esta llamada a _dump()_ genera el mapa de clases y lo escribe en el archivo _class_map.php_ en el mismo directorio con el siguiente contenido:

```
<?php return array (
'Acme\\Foo' => '/var/www/library/foo/Bar.php',
'Acme\\Foo\\Bar' => '/var/www/library/foo/bar/Foo.php',
'Acme\\Bar\\Baz' => '/var/www/library/bar/baz/Boo.php',
'Acme\\Bar' => '/var/www/library/bar/Foo.php',
);
```

En lugar de cargar cada archivo manualmente, sólo tendrás que registrar el mapa de clases generado con, por ejemplo, MapClassLoader:

```
use Symfony\Component\ClassLoader\MapClassLoader;

$mapping = include __DIR__.'/class_map.php';
$loader = new MapClassLoader($mapping);
$loader->register();

// you can now use the classes:
use Acme\Foo;

$foo = new Foo();

// ...
```

El ejemplo asume que ya tienes funcionando autoloading (desde Composer o desde otros class loaders del componente ClassLoader).

Además de volcar el mapa de clases para un directorio, puedes también pasar un array de directorios para los que generar el mapa de clases:

```
use Symfony\Component\ClassLoader\ClassMapGenerator;

ClassMapGenerator::dump(
    array(__DIR__.'/library/bar', __DIR__.'/library/foo'),
    __DIR__.'/class_map.php'
);
```