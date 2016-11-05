El **service container se puede compilar** por varias razones. Estas razones incluyen **comprobar problemas potenciales** como referencias circulares y hacer al **container más eficiente** resolviendo parámetros y removiendo services no utilizados. Además algunas funcionalidades como [parent services](http://symfony.com/doc/current/components/dependency_injection/parentservices.html) requieren compilar el container. Se compila ejecutando:

```
$container->compile();
```

El método _compile_ utiliza **Compiler Passes** para la compilación. El componente **DependencyInjection** viene con algunos passes que son registrados automáticamente para compilar. Por ejemplo el **CheckDefinitionValidityPass** comprueba varios problemas potenciales con las definiciones establecidas en el container. Después de éste y otros passes que comprueban la validez del container, se emplean más compiler passes para **optimizar la configuración** antes de cachearse. Por ejemplo, los _private services_ y _abstract services_ se eliminan y los _aliases_ se resuelven.

**Indice de contenido**

1.  Administrar la configuración con extensiones
2.  Anteponer la configuración de las extensiones
3.  Crear un compiler pass
4.  Registrar un compiler pass
5.  Volcar la configuración para el rendimiento

### 1\. Administrar la configuración con extensiones

Además de cargar la configuración directamente en el container, también podemos administrarla **registrando extensiones con el container**. El primer paso en el proceso de compilación es **cargar configuración de cualquier clase extension** registrada en el container. A diferencia de la configuración cargada directamente, sólo se procesan cuando el container es compilado. Si tu aplicación es modular, las extensiones permiten a cada módulo registrar y administrar su propia configuración de services.

La extensión debe implementar **ExtensionInterface** y puede registrarse en el container con:

```
$container->registerExtension($extension);
```

La principal tarea de la extensión es la del método _load_. En el método load puedes **cargar configuración de uno o más archivos de configuración** así como **manipular las definiciones del container con métodos** [Container Service Definitions](http://symfony.com/doc/current/components/dependency_injection/definitions.html).

Al método _load_ se le pasa un container para instalar, el cual se une después al container en el que está registrado. Esto te permite tener varias extensiones administrando definiciones de containers de forma independiente. Las **extensiones** no se añaden a la **configuración del container** cuando se añaden, sino que se procesan cuando se llama al método _compile_ del container.

Una extensión muy simple puede simplemente cargar archivos de configuración en el container:

```
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Loader\XmlFileLoader;
use Symfony\Component\DependencyInjection\Extension\ExtensionInterface;
use Symfony\Component\Config\FileLocator;

class AcmeDemoExtension implements ExtensionInterface
{
    public function load(array $configs, ContainerBuilder $container)
    {
        $loader = new XmlFileLoader(
            $container,
            new FileLocator(__DIR__.'/../Resources/config')
        );
        $loader->load('services.xml');
    }

    // ...
}
```

Con esto no se gana mucho comparado con cargar el archivo directamente en el _container_ construído normalmente. Simplemente permite que los **archivos puedan dividirse en módulos/bundles**. Poder modificar la configuración de un módulo desde archivos de configuración fuera del módulo o bundle es necesario para configurar bien una aplicación compleja. Esto es posible haciendo que las **especificaciones de secciones de archivos de configuración** se carguen directamente en el container como una **extensión** particular. Estas secciones en la configuración no se procesarán directamente por el container sino por la extensión relevante.

La extensión debe especificar el método _getAlias_ para implementar la interface:

```
// ...

class AcmeDemoExtension implements ExtensionInterface
{
    // ...

    public function getAlias()
    {
        return 'acme_demo';
    }
}
```

Para **archivos de configuración YAML** especificar el _alias_ para la extensión como una _key_ significará que esos valores se pasen al método _load_ de la extensión:

```
# ...
acme_demo:
    foo: fooValue
    bar: barValue
```

Si este archivo se carga en la configuración, los valores en él sólo se procesan cuando el container se compila, en cuyo punto se cargan las Extensiones:

```
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\Loader\YamlFileLoader;

$container = new ContainerBuilder();
$container->registerExtension(new AcmeDemoExtension);

$loader = new YamlFileLoader($container, new FileLocator(__DIR__));
$loader->load('config.yml');

// ...
$container->compile();
```

Cuando se carga un **archivo de configuración** que usa un alias de extensión como _key_, la extensión debe haber sido registrada con el **container builder** o se lanzará una **excepción**.

Los valores de esas secciones de los archivos de configuración se pasan como primer argumento en el método _load_ de la extensión:

```
public function load(array $configs, ContainerBuilder $container)
{
    $foo = $configs[0]['foo']; //fooValue
    $bar = $configs[0]['bar']; //barValue
}
```

El argumento _$configs_ es un array que contiene cada archivo diferente de configuración que se ha cargado en el container. Sólo cargas un archivo de configuración en el ejemplo anterior pero estára en un array. El array es del estilo:

```
array(
    array(
        'foo' => 'fooValue',
        'bar' => 'barValue',
    ),
)
```

Aunque puedes manejarlo manualmente uniendo diferentes archivos, es mejor utilizar el [componente Config](http://symfony.com/doc/current/components/config/introduction.html) para unir y validar los valores de configuración. Con el **configuration processing** puedes acceder al valor de configuración de la siguiente forma:

```
use Symfony\Component\Config\Definition\Processor;
// ...

public function load(array $configs, ContainerBuilder $container)
{
    $configuration = new Configuration();
    $processor = new Processor();
    $config = $processor->processConfiguration($configuration, $configs);

    $foo = $config['foo']; //fooValue
    $bar = $config['bar']; //barValue

    // ...
}
```

Existen dos método más que se han de implementar. Uno para devolver el namespace XML de forma que las partes relevantes de un archivo de configuración XML se pasen a la extensión. El otro para especificar el base path para archivos XSD para validar la configuración XML:

```
public function getXsdValidationBasePath()
{
    return __DIR__.'/../Resources/config/';
}

public function getNamespace()
{
    return 'http://www.example.com/symfony/schema/';
}
```

La **validación XSD** es opcional, puedes devolver false del método _getXsdValidationBasePath_ para desactivarlo.

La versión XML de la configuración sería algo así:

```
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:acme_demo="http://www.example.com/symfony/schema/"
    xsi:schemaLocation="http://www.example.com/symfony/schema/ http://www.example.com/symfony/schema/hello-1.0.xsd">

    <acme_demo:config>
        <acme_demo:foo>fooValue</acme_hello:foo>
        <acme_demo:bar>barValue</acme_demo:bar>
    </acme_demo:config>
</container>
```

En la **versión full-stack de Symfony** hay una clase **base Extension** que implementa estos métodos además de un método **shotcurt** para procesar la configuración.

El valor de configuración procesado puede anadirse ahora como **parámetros de container** como si fuera listada en una sección _parameters_ del archivo de configuración pero con el beneficio adicional de **unir múltiples archivos** y **validación para la configuración**. 

```
public function load(array $configs, ContainerBuilder $container)
{
    $configuration = new Configuration();
    $processor = new Processor();
    $config = $processor->processConfiguration($configuration, $configs);

    $container->setParameter('acme_demo.FOO', $config['foo']);

    // ...
}
```

Se pueden añadir requisitos de configuración más complejos en las clases Extension. Por ejemplo, puedes cargar un archivo principal de configuración de services pero también cargar un archivo secundario sólo si se establece un parámetro en concreto:

```
public function load(array $configs, ContainerBuilder $container)
{
    $configuration = new Configuration();
    $processor = new Processor();
    $config = $processor->processConfiguration($configuration, $configs);

    $loader = new XmlFileLoader(
        $container,
        new FileLocator(__DIR__.'/../Resources/config')
    );
    $loader->load('services.xml');

    if ($config['advanced']) {
        $loader->load('advanced.xml');
    }
}
```

Sólo registrar una extensión con el container no es suficiente para incluirla en las extensiones procesadas cuando el container se compila. Cargar una configuración que usa el alias de la extensión como key como en los ejemplos anteriores asegurará que se carga. Se le puede decir también al container builder que lo cargue con su método _loadFromExtension()_:

```
use Symfony\Component\DependencyInjection\ContainerBuilder;

$container = new ContainerBuilder();
$extension = new AcmeDemoExtension();
$container->registerExtension($extension);
$container->loadFromExtension($extension->getAlias());
$container->compile();
```

Si necesitas manipular la configuración cargada por una extensión no podrás hacerlo desde otra extensión ya que utiliza un nuevo container. Deberías emplear un **compiler pass** que funciona con el container después de que se haya procesado la extensión.

### 2\. Anteponer la configuración de las extensiones

Una extensión puede anteponerse a la configuración de cualquier bundle antes de que el método _load()_ sea llamado implementando **PrependExtensionInterface**:

```
use Symfony\Component\DependencyInjection\Extension\PrependExtensionInterface;
// ...

class AcmeDemoExtension implements ExtensionInterface, PrependExtensionInterface
{
    // ...

    public function prepend()
    {
        // ...

        $container->prependExtensionConfig($name, $config);

        // ...
    }
}
```

### 3\. Crear un Compiler Pass

También puedes **crear y registrar tus propios compiler passes** con el container. Para crear un compiler pass se ha de implementar la interface **CompilerPassInterface**. El compiler pass ofrece una oportunidad de manipular las definiciones de services que se hayan compilado. Esto puede ser muy útil, pero no se necesita siempre.

El compiler pass debe tener el método process que se pasa al container que se está compilando:

```
use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class CustomCompilerPass implements CompilerPassInterface
{
    public function process(ContainerBuilder $container)
    {
       // ...
    }
}
```

Los parámetros y definiciones del container pueden manipularse utilizando [Container Service Definitions](http://symfony.com/doc/current/components/dependency_injection/definitions.html). Una cosa común a hacer en un compiler pass es **buscar todos los services que tienen una etiqueta en común** para procesarlos de alguna forma o **conectarlos dinámicamente** en algún otro service.

### 4\. Registrar un Compiler Pass

Necesitas registrar tu pass personalizado en el container. Su método para procesar se llamará cuando se compile el container. Por ejemplo en un bundle AcmeMailerBundle:

```
// src/Acme/MailerBundle/AcmeMailerBundle.php
namespace Acme\MailerBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;
use Symfony\Component\DependencyInjection\ContainerBuilder;

use Acme\MailerBundle\DependencyInjection\Compiler\CustomCompilerPass;

class AcmeMailerBundle extends Bundle
{
    public function build(ContainerBuilder $container)
    {
        parent::build($container);

        $container->addCompilerPass(new CustomCompilerPass());
    }
}
```

#### Controlar el Pass Ordering

Los **compiler passes** por defecto se agrupan en **optimization passes** y **removal passes**. Los **passes de optimización** se ejecutan primero e incluyen tareas como resolver referencias en las definiciones. Los **passes de eliminación** llevan a cabo tareas como eliminar aliases _private_ y services inutilizados. Puedes elegir dónde ejecutar los compiler passes customizados. Por defecto se ejecutarán antes de los passes de optimización.

Puedes emplear las siguientes constantes como segundo argumento cuando se registra un pass en el container para controlar el orden:

*   PassConfig::TYPE_BEFORE_OPTIMIZATION
*   PassConfig::TYPE_OPTIMIZE
*   PassConfig::TYPE_BEFORE_REMOVING
*   PassConfig::TYPE_REMOVE
*   PassConfig::TYPE_AFTER_REMOVING

Por ejemplo, para ejecutar el pass personalizado después de que los passes de eliminación se hayan ejecutado:

```
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Compiler\PassConfig;

$container = new ContainerBuilder();
$container->addCompilerPass(
    new CustomCompilerPass,
    PassConfig::TYPE_AFTER_REMOVING
);
```

### 5\. Volcar la configuración para el rendimiento

Utilizar **archivos de configuración para administrar el service container** puede ser mucho más fácil de entender que utilizar **PHP** directamente, especialmente cuando ya hay muchos services. Sin embargo, esta facilidad tiene la contra de que los archivos de configuración necesitan ser analizados por lo que tiene un pequeño **impacto en el rendimiento**. Este proceso de compilación hace al container más eficiente pero lleva algo de tiempo al ejecutarse. Puedes tener lo mejor de ambas formas empleando archivos de configuración y volcando y cacheando la configuración resultante. El **PhpDumper** hace que volcar el container compilado sea fácil:

```
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Dumper\PhpDumper;

$file = __DIR__ .'/cache/container.php';

if (file_exists($file)) {
    require_once $file;
    $container = new ProjectServiceContainer();
} else {
    $container = new ContainerBuilder();
    // ...
    $container->compile();

    $dumper = new PhpDumper($container);
    file_put_contents($file, $dumper->dump());
}
```

**ProjectServiceContainer** es el nombre por defecto que se le da a la clase container volcada, puedes cambiarlo con la opción _class_ cuando lo vuelques:

```
// ...
$file = __DIR__ .'/cache/container.php';

if (file_exists($file)) {
    require_once $file;
    $container = new MyCachedContainer();
} else {
    $container = new ContainerBuilder();
    // ...
    $container->compile();

    $dumper = new PhpDumper($container);
    file_put_contents(
        $file,
        $dumper->dump(array('class' => 'MyCachedContainer'))
    );
}
```

Ahora tendremos la **velocidad de un container configurado en PHP** con la **facilidad del uso de los archivos de configuración**. Además, volcar el container de esta forma promueve la optimización de cómo se crean los services por el container.

En el ejemplo anterior necesitarás eliminar el archivo container cacheado siempre que hagas cambios. Añadir un control en una variable que determina si estás en modo debug permite mantener la velocidad del container cachead en producción pero manteniendo una configuración actualizada en el modo de desarrollo:

```
// ...

// basado en algo de tu proyecto
$isDebug = ...;

$file = __DIR__ .'/cache/container.php';

if (!$isDebug && file_exists($file)) {
    require_once $file;
    $container = new MyCachedContainer();
} else {
    $container = new ContainerBuilder();
    // ...
    $container->compile();

    if (!$isDebug) {
        $dumper = new PhpDumper($container);
        file_put_contents(
            $file,
            $dumper->dump(array('class' => 'MyCachedContainer'))
        );
    }
}
```

Esto puede mejorarse más sólo **recompilando el container en modo debug** cuando los cambios se han realizado en la configuración en lugar de en cada **request**. Esto puede hacerse **cacheando los archivos resource** empleados para configurar el container como se indica en el [componente Config para el cacheo en resources](http://symfony.com/doc/current/components/config/caching.html).

No necesitas saber qué archivos cachear ya que el **container builder** realiza un **seguimiento de los resources utilizados** para configurarlo, no sólo los **archivos de configuración**, también las **clases extension** y los **compiler passes**. Esto significa que cualquier cambio a cualquiera de estos archivos invalidará la cache y lanzará el container que se está reconstruyendo. Sólo necesitas pedir al container estos resources y usarlos como metadatos para la cache:

```
// ...

// basado en algo de tu proyecto
$isDebug = ...;

$file = __DIR__ .'/cache/container.php';
$containerConfigCache = new ConfigCache($file, $isDebug);

if (!$containerConfigCache->isFresh()) {
    $containerBuilder = new ContainerBuilder();
    // ...
    $containerBuilder->compile();

    $dumper = new PhpDumper($containerBuilder);
    $containerConfigCache->write(
        $dumper->dump(array('class' => 'MyCachedContainer')),
        $containerBuilder->getResources()
    );
}

require_once $file;
$container = new MyCachedContainer();
```

Ahora el **dumper container** cacheado se usa independientemente del **modo debug** o no. La diferencia es que **ConfigCache** se establece en modo debug con el segundo argumento del constructor. Cuando la cache no está en modo debug el container cacheado siempre se usará si existe. En modo debug, un archivo adicional de metadatos se escribe con los timestamps de todos los archivos resource. Estos entonces se comprueban para ver si los archivos se han cambiado, y si es así la caché se considerará _stale_ (obsoleta).

En la **versión full-stack de Symfony** la compilación y el caching del container se hace de forma automática.