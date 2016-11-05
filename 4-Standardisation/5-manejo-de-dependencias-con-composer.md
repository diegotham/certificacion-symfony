**Composer** es una herramienta para el manejo de dependencias en PHP. Permite declarar las librerías de las que tu proyecto depende y se encarga de administrarlas (tanto en su instalación como en su actualización).

Composer no es un administrador de paquetes. Trabaja con paquetes o librerías, pero los administra por proyectos, instalándolos en un directorio (/vendor) dentro del proyecto. Por defecto no instala nada globalmente, por lo que es un **gestor de dependencias**. También tiene soporte para proyectos globales a través del comando **global**.

Composer está inspirado en el **npm** de **node** y en el **bundler** de **ruby**.

Puedes ver cómo se instala en las diferentes plataformas en [este enlace](https://getcomposer.org/doc/00-intro.md).

**Indice de contenido**

1.  Uso básico
2.  Packagist
3.  Autoloading

### Uso básico

Para explicar su uso básico vamos a instalar una dependencia, _monolog/monolog,_ una librería de **logging**. Suponemos que la instalación la hemos hecho de forma [local](https://getcomposer.org/doc/00-intro.md#locally).

Para empezar a usar composer en un proyecto simplemente hay que tener un archivo _composer.json_. Este archivo describe las dependencias de tu proyecto y puede contener otros metadatos.

La primera (y muchas veces única) _key_ a especificar en _composer.json_ es **_require_**. Con esto se le dice a Composer en que paquetes depende tu proyecto:

```
{
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```

_**require**_ es un objeto que mapea **nombres de paquetes** (en este caso _monolog/monolog_) con sus **versiones**.

El nombre de los paquetes consiste en un **vendor name** y en el **project's name**. A menudo estos serán idénticos, el vendor name está para evitar colisiones entre nombres. Permite a dos personas diferentes crear una librería llamada json, por ejemplo: martag/json y josel/json.

En este ejemplo requerimos _monolog/monolog_, siendo el vendor el mismo que el nombre del proyecto. Para proyectos con un único nombre esto es lo recomendable. También permite añadir proyectos relacionados bajo el mismo namespace más adelante.

En el ejemplo hemos requerido la versión 1.0.* de Monolog. Esto significa cualquier versión en la rama de desarrollo 1.0\. Es el equivalente a decir las versiones >=1.0 < 1.1\. Las versiones se pueden especificar de múltiples formas, se puede leer más sobre ello [aquí](https://getcomposer.org/doc/articles/versions.md). 

Por defecto sólo se toman en consideración versiones estables. Si quieres descargar una RC, beta, alpha o dev puedes hacerlo utilizando [stability flags](https://getcomposer.org/doc/04-schema.md#package-links). Para cambiar esto en todos los paquetes en lugar de hacerlo dependencia por dependencia puedes utilizar el ajuste [minimum-stability](https://getcomposer.org/doc/04-schema.md#minimum-stability).

Para instalar las dependencias definidas en el archivo _composer.json_, simplemente hay que emplear el siguiente comando (teniendo en cuenta que lo hemos instalado de forma local):

```
php composer.phar install 
```

Este encontrará la última versión de _monolog/monolog_ y la descargará en _/vendor_. Lo normal es guardar todo el código de terceros en un directorio llamado _vendor_. En el caso de Monolog lo pondrá en _vendor/monolog/monolog_.

Si utilizas **git** debes añadir el directorio vendor en _.gitignore_.

Con el comando _install_ también se genera un archivo llamado _composer.lock_. Después de instalar las dependencias, Composer escribe la lista de versiones exactas que instaló en un archivo _composer.lock_. Esto bloquea el proyecto a esas versiones específicas.

Haz **commit** en git tanto de _composer.lock_ como de _composer.json_. Esto es importante porque el comando install comprueba si existe un archivo lock, y si existe, descarga las versiones especificadas aquí (independientemente de lo que diga composer.json).

Esto significa que cualquiera que instale el proyecto descargará la versión exacta de las dependencias. Tu servidor y otros desarrolladores trabajarán con las mismas dependencias, lo que evita generar bugs generados por incompatibilidad de versiones. Incluso si desarrollas por tu cuenta, en seis meses cuando reinstales el proyecto podrás estar seguro de que las dependencias instaladas todavía funcionan aún cuando tus dependencias hayan desarrollado versiones nuevas desde entonces. 

Si no existe un archivo _composer.lock_, Composer leerá las dependencias y versiones desde _composer.json_ y creará el archivo lock después de ejecutar los comandos **update** o **install**. Esto significa que si cualquiera de las dependencias tiene una nueva versión, no se instalarán en el proyecto de forma automática. Para actualizar a la nueva versión, utiliza el comando update. Esto buscará las últimas versiones que coinciden (según el archivo _composer.json_) y actualizará el archivo lock con la nueva versión.

```
php composer.phar update
```

Composer lanzará un **warning** cuando se ejecute el comando install si _composer.lock_ y _composer.json_ no están sincronizados.

Si sólo quieres instalar o actualizar una dependencia, puedes hacerlo individualmente:

php composer.phar update monolog/monolog

### Packagist

[**Packagist**](https://packagist.org/) es el principal repositorio de Composer. Un repositorio de Composer es básicamente una fuente de paquetes, un sitio desde el que poder obtener paquetes. Puedes hacer **require** con cualquiera de los paquetes que aparecen ahí.

Cualquier proyecto open source que utilice Composer es recomendable que publique el paquete en Packagist. Una librería no necesita estar en Packagist para ser usada por Composer, pero permite que otros desarrolladores la descubran más rápidamente.

### Autoloading

Para librerías que especifican información de **autoload**, Composer genera un archivo _vendor/autoload.php_. Puedes incluir simplemente este archivo y directamente ya tendrás un autoloading:

```
require __DIR__ . '/vendor/autoload.php';
```

Esto hace que sea fácil utilizar código de terceros. Por ejemplo, si tu proyecto depende de **Monolog**, puedes simplemente empezar a utilizar clases de ahí, y serán autocargadas:

```
$log = new Monolog\Logger('name');
$log->pushHandler(new Monolog\Handler\StreamHandler('app.log', Monolog\Logger::WARNING));
$log->addWarning('Foo');
```

Puedes incluso añadir tu propio código al autoloader añadiendo el campo autoload en _composer.json_:

```
{
    "autoload": {
        "psr-4": {"Acme\\": "src/"}
    }
}
```

Composer registrará un autoloader PSR-4 del namespace Acme.

Hay que definir el mapeo de **namespaces** a **directorios**. El directorio _/src_ estaría en el _root_ de tu proyecto, en el mismo nivel en el que está el directorio _vendor_. Un ejemplo de nombre de archivo sería _src/Foo.php_ que contiene una clase _Acme\Foo_. 

Después de añadir el campo autoload, tienes que regenerar el archivo _vendor/autoload.php_ mediante el comando _dump-autoload_.

Incluir el archivo _vendor/autoload.php_ también devolverá una instancia del autoloader, por lo que puedes guardar el valor de retorno en una variable y añadir más namespaces. Esto puede resultar útil para cargar clases en testing, por ejemplo.

```
$loader = require __DIR__ . '/vendor/autoload.php';
$loader->add('Acme\\Test\\', __DIR__);
```

Además del autoloading PSR-4, Composer también soporta PSR-0, mapeo de clases y autocarga de archivos, puedes encontrar más información [aquí](https://getcomposer.org/doc/04-schema.md#autoload). 

Composer proporciona su propio autoloader. Si no quieres utilizar este, puedes incluir archivos en _vendor/composer/autoload\_*.php_ que devuelvan arrays asociativos permitiendo configurar tu propio autoloader.