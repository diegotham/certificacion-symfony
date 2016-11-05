Los resources de los bundles en Symfony se guardan en la carpeta _/Resources_:

*   _config/_. Guarda **configuración** específica del bundle, como puede ser _routing_ o _services_.
*   _doc/_. Guarda la **documentación** del bundle.
*   _translations/_. Guarda los archivos de **traducción** del bundle.
*   _views/_. Guarda las **templates** del bundle.
*   _public/_. Guarda los **assets** del bundle.

En el caso de las templates, archivos de traducción y opciones de configuración, **Symfony** recomienda que se guarden en el directorio central _app/Resources_ en lugar de en _AppBundle/Resources_. Los assets también recomienda guardarlos fuera del bundle, en el directorio _/web_.

El componente [Config](https://symfony.com/doc/current/components/config/index.html) proporciona diferentes **clases** para ayudar a encontrar, cargar, combinar, autorellenar y validar **valores de configuración** de cualquier tipo, ya sea en formato YAML, XML, INI o una base de datos.

### Localizar resources

La carga de la configuración normalmente empieza con una **búsqueda de recursos** (en la mayoría de los casos **archivos**). Esto se hace con [FileLocator](http://api.symfony.com/3.0/Symfony/Component/Config/FileLocator.htmlg/FileLocator.html).

```
use Symfony\Component\Config\FileLocator;

$configDirectories = array(__DIR__.'/app/config');

$locator = new FileLocator($configDirectories);
$yamlUserFiles = $locator->locate('users.yml', null, false);
```

El **localizador** recibe una colección de localizaciones en donde debería buscar archivos. El primer argumento de _**locate()**_ es el **nombre del archivo a buscar**. El segundo argumento por defecto es el **directorio actual**, y si se suministra alguno, el localizador buscará en ese directorio primero. El tercer argumento indica si el localizador debe devolver o no el primer archivo que ha encontrado o un array con todos los que ha encontrado (false si se desea el array).

### Resource Loaders

Para cada tipo de **recurso** (YAML, XML, annotation, etc) se debe definir un **loader**. Cada loader deberá implementar **LoaderInterface** o extender la clase abstracta **FileLoader**, que permite importar otros recursos de forma **recursiva**:

```
use Symfony\Component\Config\Loader\FileLoader;
use Symfony\Component\Yaml\Yaml;

class YamlUserLoader extends FileLoader
{
    public function load($resource, $type = null)
    {
        $configValues = Yaml::parse(file_get_contents($resource));

        // ... manejar los valores de configuración

        // se puede importar otro resource:

        // $this->import('extra_users.yml');
    }

    public function supports($resource, $type = null)
    {
        return is_string($resource) && 'yml' === pathinfo(
            $resource,
            PATHINFO_EXTENSION
        );
    }
}
```

### Encontrar el Loader correcto

El **LoaderResolver** recibe como primer argumento del **constructor** una colección de loaders. Cuando se carga un recurso (por ejemplo un archivo XML), se produce un **loop** a través de esta colección de loaders y devuelve el loader que soporta este tipo de recurso particular.

El **DelegatingLoader** usa el **LoaderResolver**. Cuando se le pregunta para cargar un recurso, delega la pregunta al LoaderResolver. En caso de que el **resolver** haya encontrado un **loader** adecuado, se le pedirá al loader que cargue el recurso:

```
use Symfony\Component\Config\Loader\LoaderResolver;
use Symfony\Component\Config\Loader\DelegatingLoader;

$loaderResolver = new LoaderResolver(array(new YamlUserLoader($locator)));
$delegatingLoader = new DelegatingLoader($loaderResolver);

$delegatingLoader->load(__DIR__.'/users.yml');
/*
EL YamlUserLoader se usará para cargar este resource,
ya que soporta archivos con la extensión "yml"
*/
```

Cuando se cargan todos los recursos de **configuración**, se pueden procesar los valores de configuración y combinarlos en un archivo. Este archivo actúa como una **cache**. Su contenido no tiene que regenerarse cada vez que se inicia la **aplicación**.

Después de cargar los valores de configuración para todo tipo de recursos, los valores y sus estructuras pueden definirse con la parte **Definition** del componente **Config**. Toda la información acerca de **definir y procesar valores de configuración** puede encontrarse [aquí](http://symfony.com/doc/2.3/components/config/definition.html).