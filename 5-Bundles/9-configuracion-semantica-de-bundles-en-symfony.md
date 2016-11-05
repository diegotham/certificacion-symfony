Si abres el archivo de configuración de tu aplicación (normalmente _app/config/config.yml_), verás un número de secciones diferentes de configuración, como **framework**, **twig** y **doctrine**. Cada uno de estos configura un bundle específico. Puedes definir opciones que puedan configurarse en el _config.yml_ y que administren el funcionamiento de tu bundle.

Por ejemplo, la siguiente configuración le dice a **FrameworkBundle** que active la integración de formularios, que incluye la definición de unos cuantos _services_ así como la integración de otros componentes relacionados:

```
framework:
    form: true
```

Es mejor utilizar **parámetros para configurar tu bundle**, aunque si no tienes planes de compartir tu bundle con otros proyectos, no tiene mucho sentido utilizar esta forma de configuración más avanzada. Si utilizas el bundle sólo en un proyecto, puedes simplemente cambiar la configuración del _service_ cada vez.

**Indice de contenido**

| | |
| -------- | -------- |
| 1. Cargar configuración de services | 4. Volcar la configuración |
| 2. Utilizar la extensión bundle | 5. Soporte XML |
| 3. Modificar la configuración de otro bundle | 6\. Compiler Passes |

### 1\. Cargar configuración de services

#### Crear la clase Extension

Para cargar la configuración del service tienes que crear una extensión **Dependency Injection** para tu bundle. Esta clase tiene algunas convenciones para poder ser detectado automáticamente, pero despues verás cómo puedes cambiarlo para tus propias necesidades. Por defecto, Extension tiene que ajustarse a las siguientes convenciones:

*   Tiene que estar en el namespace **DependencyInjection** del bundle.
*   El nombre es igual al nombre del bundle con el sufijo **Bundle** reemplazado por **Extension** (por ejemplo la clase Extension del bundle AppBundle sería AppExtension y la clase para AcmeHelloBundle sería AcmeHelloExtension).

La clase Extension debería implementar la **ExtensionInterface**, pero normalmente sólo tendrás que extender la clase **Extension**:

```
// src/AppBundle/DependencyInjection/AppExtension.php
namespace AppBundle\DependencyInjection;

use Symfony\Component\HttpKernel\DependencyInjection\Extension;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class AppExtension extends Extension
{
     public function load(array $configs, ContainerBuilder $container)
     {
         // Cargarás los archivos aquí después
     }
}
```

#### Registrar manualmente una clase Extension

Cuando no quieras seguir las convenciones, tendras que registrar manualmente tu extension. Para hacerlo, deberías sobreescribir el método _Bundle::getContainerExtension()_ para devolver la instancia de la extensión:

```
// ...
use AppBundle\DependencyInjection\UnconventionalExtensionClass;

class AppBundle extends Bundle
{
    public function getContainerExtension()
    {
        return new UnconventionalExtensionClass();
    }
}
```

Ya que la nueva clase **Extension** no sigue las convenciones de nombres, deberías también sobreescribir _Extension::getAlias()_ para devolver el correcto alias DI. El **alias DI** es el nombre utilizado para referirse al bundle en el container (por ejemplo en el archivo app/config/config.yml). Por defecto esto se hace removiendo el sufijo Extension y convirtiendo el nombre de clase a barras bajas (por ejemplo el alias ID de AcmeHelloExtension es acme_hello).

#### Usando el método load()

Con el método _load()_, todos los services y parámetros relacionados a esta extensión se cargarán. Este método no obtiene la instancia real del container, sino una copia. Este container sólo tiene los parámetros del container real. Después de cargar los services y parámetros, la copia se unirá al container real para asegurar que todos los services y parámetros también se añaden al container real.

En el método _load()_ puedes usar el código PHP para registrar **definiciones de services**, pero es más frecuente que pongas estas definiciones en un archivo de configuración (usando YAML, XML o PHP). Por suerte puedes emplear _file loaders_ en la extensión.

Por ejemplo si tenemos un archivo llamado _services.xml_ en el directorio _Resources/config_ de tu bundle, tu método load será así:

```
use Symfony\Component\DependencyInjection\Loader\XmlFileLoader;
use Symfony\Component\Config\FileLocator;

// ...
public function load(array $configs, ContainerBuilder $container)
{
    $loader = new XmlFileLoader(
        $container,
        new FileLocator(__DIR__.'/../Resources/config')
    );
    $loader->load('services.xml');
}
```

Otros loaders disponibles son YamlFileLoader, PhpFileLoader e IniFileLoader (este último sólo puede usarse para cargar parámetros y sólo puede cargarlos como strings).

### 2\. Utilizar la extensión Bundle

La idea básica es que en lugar de tener que modificar los parámetros individualmente, permites al desarrollador configurar sólo unas pocas opciones específicamente creadas. Como desarrollador del bundle, analizas esa configuración y cargas los _services_ correctos y parámetros dentro de una clase **Extension**.

Como ejemplo, imagina que estas creando un social bundle, que proporciona integración con **Twitter** y demás. Para poder reutilizar el bundle, tienes que hacer las variables **client_id** y **client_secret** configurables. La configuración de tu bundle sería así:

```
# app/config/config.yml
acme_social:
    twitter:
        client_id: 123
        client_secret: $ecret
```

Si un bundle proporciona una **clase Extension**, generalmente no deberías sobreescribir los parámetros del **service container** de ese bundle. La idea es que si hay una clase Extension, cada ajuste que se tenga que configurar debería estar presente en la configuración disponible proporcionada por esa clase. En otras palabras, la clase _extension_ define todos los ajustes de configuración públicos para mantener la compatibilidad de versiones.

#### Procesar al array de $configs

Una vez creada la clase Extension, cuando el desarrollador incluya la _key_ **acme_social** (que es el DI alias) en un archivo de configuración, la configuración se añadirá a un array de configuraciones y se pasará al método load() de tu extensión (Symfony automáticamente convierte XML y YAML a un array).

Para el ejemplo de configuración dado antes, el array que se pasa al método load es así:

```
array(
    array(
        'twitter' => array(
            'client_id' => 123,
            'client_secret' => '$ecret',
        ),
    ),
)
```

Nótese que es un array de arrays, no un simple array de los valores de configuración. Esto es intencionado ya que permite a **Symfony** analizar varias fuentes de configuración. Por ejemplo, si **acme_social** aparece en otro archivo de configuración, por ejemplo _config_dev.yml_, con valores diferentes, el array sería como sigue:

```
array(
    // valores de config.yml
    array(
        'twitter' => array(
            'client_id' => 123,
            'client_secret' => '$secret',
        ),
    ),
    // valores de config_dev.yml
    array(
        'twitter' => array(
            'client_id' => 456,
        ),
    ),
)
```

El orden de los dos arrays depende de cual se establece primero.

El componente **Config** de Symfony ayudará a unir estos valores, proporcionar valores por defecto y mostrará errores de validación en caso de configuración incorrecta. Primero creamos una clase **Configuration** en el directorio **DependencyInjection** y creamos un árbol que define la estructura de la configuración de tu bundle. 

```
// src/Acme/SocialBundle/DependencyInjection/Configuration.php
namespace Acme\SocialBundle\DependencyInjection;

use Symfony\Component\Config\Definition\Builder\TreeBuilder;
use Symfony\Component\Config\Definition\ConfigurationInterface;

class Configuration implements ConfigurationInterface
{
    public function getConfigTreeBuilder()
    {
        $treeBuilder = new TreeBuilder();
        $rootNode = $treeBuilder->root('acme_social');

        $rootNode
            ->children()
                ->arrayNode('twitter')
                    ->children()
                        ->integerNode('client_id')->end()
                        ->scalarNode('client_secret')->end()
                    ->end()
                ->end() // twitter
            ->end()
        ;

        return $treeBuilder;
    }
}
```

La clase **Configuration** puede ser mucho más complicada que el ejemplo anterior, soportando nodos prototype, validación avanzada, normalización específica de XML y uniones más avanzadas. Puedes leer más en la [documentación del componente Config](http://symfony.com/doc/current/components/config/definition.html), o ver ejemplos complejos de configuración del [FrameworkBundle](https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/DependencyInjection/Configuration.php) o del [TwigBundle](https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/TwigBundle/DependencyInjection/Configuration.php).

Configuration ahora puede emplearse en el método _load()_ para unir configuraciones y forzar la validación (por ejemplo, si se pasa una opción adicional, se lanzará una excepción):

```
public function load(array $configs, ContainerBuilder $container)
{
    $configuration = new Configuration();

    $config = $this->processConfiguration($configuration, $configs);
    // ...
}
```

El método _processConfiguration()_ utiliza el árbol de configuración que hemos definido en la clase Configuration para validar, normalizar y unir todos los arrays de configuración juntos.

En lugar de llamar a _processConfiguration()_ en tu extensión cada vez que proporcionas algunas opciones de configuración, puedes extender [ConfigurableExtension](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/DependencyInjection/ConfigurableExtension.html) para que se haga de forma automática:

```
// src/Acme/HelloBundle/DependencyInjection/AcmeHelloExtension.php
namespace Acme\HelloBundle\DependencyInjection;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\DependencyInjection\ConfigurableExtension;

class AcmeHelloExtension extends ConfigurableExtension
{
    // nótese que este método se llama loadInternal y no load
    protected function loadInternal(array $mergedConfig, ContainerBuilder $container)
    {
        // ...
    }
}
```

Esta clase utiliza el método _getConfiguration()_ para obtener la instancia **Configuration**. Deberías sobreescribirlo si tu clase de configuración no se llama Configuration o no está en el mismo namespace que la extensión.

#### Procesar tú mismo la configuración

Utilizar el componente Config es totalmente opcional. El método load() obtiene un array de valores de configuración. Puedes simplemente parsear estos arrays tú mismo (por ejemplo sobreescribiendo configuraciones y utilizando _**isset**_ para comprobar la existencia de un valor). Ten en cuenta que será complicado que soporte XML:

```
public function load(array $configs, ContainerBuilder $container)
{
    $config = array();
    // dejar que los resources sobreescriban el set de valores previo
    foreach ($configs as $subConfig) {
        $config = array_merge($config, $subConfig);
    }

    // ... ahora utiliza la etiqueta $config array
}
```

### 3\. Modificar la configuración de otro bundle

Si tiene múltiples bundles que dependen unos de otros, podría ser útil permitir a una **clase Extension** modificar la configuración de otra clase extension de otro bundle, como si se pudiera definir la configuración desde _app/config/config.yml_. Esto se puede conseguir utilizando una [prepend extension](http://symfony.com/doc/current/cookbook/bundles/prepend_extension.html). 

### 4\. Volcar la configuración

El comando _config:dump-reference_ vuelca la configuración por defecto de un bundle en la consola utilizando el formato Yaml.

Siempre que la configuración de tu bundle esté localizada en la localización estándar (_TuBundle\DependencyInjectio\Configuration_) y no requiere que se pasen argumentos al constructor, funcionará automáticamente. Si tienes algo diferente, tu clase Extension debe sobreescribir al método _Extension::getConfiguration()_ y devolver una instancia de tu **Configuration**.

### 5\. Soporte XML

Symfony posibilita proporcionar la configuración en tres formatos diferentes: YAML, XML y PHP. YAML y PHP utilizan la misma sintaxis y son soportados por defecto cuando se utiliza el componente Config. Soporte a XML requiere que hagas algunas cosas más. Pero cuando compartes tu bundle con otros es recomendable seguir los siguientes pasos.

#### Tener el árbol Config listo para XML

El **componente Config** proporciona algunos métodos por defecto para permitir procesar correctamente **configuración en XML**. Puedes ver la **Normalization** en la configuración del componente. Además, puedes hacer algunas cosas adicionales también, lo que mejorará la experiencia de **utilizar XML en la configuración**:

#### Elegir un namespace XML

En XML, el namespace XML se emplea para determinar qué elementos pertenecen a la configuración de un bundle específico. El namespace es devuelto por el método _Extension::getNamespace()_. Por convención, el namespace es una URL (no hace falta que sea válida ni que exista). Por defecto el namespace para un bundle es _http://example.org/dic/schema/DI_ALIAS_, donde DI_ALIAS es el DI alias de la extensión. Podrías querer cambiar esto a una URL más profesional:

```
// src/Acme/HelloBundle/DependencyInjection/AcmeHelloExtension.php

// ...
class AcmeHelloExtension extends Extension
{
    // ...

    public function getNamespace()
    {
        return 'http://acme_company.com/schema/dic/hello';
    }
}
```

#### Proporcionar un esquema XML

XML tiene una funcionalidad muy útil llamada [XML schema](http://diego.com.es/xml-principios-basicos#XMLSchemas). Esto te permite describir todos los posibles elementos y atributos y sus valores en un XML Schema Definition (y un archivo xsd). Este archivo XSD se emplea por IDEs para el autocompletado y es utilizado por el componente Config para validar elementos.

Para utilizar el schema, el **archivo de configuración XML** debe proporcionar un atributo xsi:schemaLocation apuntando al **archivo XSD** para un nampespace XML concreto. Esta localización siempre comienza con el **namespace XML**. Este namespace XML es entonces reemplazado por el directorio base de validación XSD devuelto por el método _Extension::getXsdValidationBasePath()_. A este namespace le sigue el resto del directorio desde el directorio base al mismo archivo.

Por convención, el archivo XSD está en _Resources/config/schema_, pero puedes emplazarlo en cualquier lado. Tienes que devolver este directorio como el directorio base:

```
// src/Acme/HelloBundle/DependencyInjection/AcmeHelloExtension.php

// ...
class AcmeHelloExtension extends Extension
{
    // ...

    public function getXsdValidationBasePath()
    {
        return __DIR__.'/../Resources/config/schema';
    }
}
```

Suponiendo que el **archivo XSD** se llama _hello-1.0.xsd_, la localización del schema será _http://acme_company.com/schema/dic/hello/hello-1.0.xsd_:

```
<!-- app/config/config.xml -->
<?xml version="1.0" ?>

<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:acme-hello="http://acme_company.com/schema/dic/hello"
    xsi:schemaLocation="http://acme_company.com/schema/dic/hello
        http://acme_company.com/schema/dic/hello/hello-1.0.xsd">

    <acme-hello:config>
        <!-- ... -->
    </acme-hello:config>

    <!-- ... -->
</container>
```

### 6. Compiler Passes

Los **Compiler Passes** permiten manipular otras definiciones de services que se han registrado en el **service container**. Puedes ver cómo se crean en la sección de [compilación del container](http://diego.com.es/compiler-passes-en-symfony). Para registrar un **compiler pass** desde un bundle hay que añadirlo al método _build_ de la clase que define el bundle:

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

Uno de los usos más comunes de los compiler passes es para los [tagged services](http://symfony.com/doc/current/components/dependency_injection/tags.html). Si utilizas **custom tags** en un bundle, por convención los nombres de los tags consisten en el **nombre del bundle** (minúsculas, barras bajas y separadores), seguidos de un **punto**, y finalmente el nombre "real". Por ejemplo, si quieres introducir algún tipo de tag "transport" en tu **AcmeMailerBundle**, se debería llamar **acme_mailer.transport**.