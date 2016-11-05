Una aplicación consiste en una **colección de bundles** que representan todas las características y funcionalidades de la aplicación. Cada bundle puede customizarse con archivos de configuración escritos en **YAML**, **XML** o **PHP**. Por defecto el principal archivo de configuración se encuentra en el directorio _app/config_ y puede ser _config.yml_, _config.xml_ y _config.php_ (por defecto es YAML):

```
# app/config/config.yml
imports:
    - { resource: parameters.yml }
    - { resource: security.yml }

framework:
    secret:          '%secret%'
    router:          { resource: '%kernel.root_dir%/config/routing.yml' }
    # ...

# Twig Configuration
twig:
    debug:            '%kernel.debug%'
    strict_variables: '%kernel.debug%'

# ...
```

Los niveles top como _framework_ o _twig_ definen la configuración de un bundle en particular. Por ejemplo, el key _framework_ define la configuración para para el core **Symfony FrameworkBundle** e incluye configuración para el routing, templating y otros sistemas core.

La Symfony Standard Edition define tres entornos de desarrollo llamados **dev**, **prod** y **test**. Un entorno simplemente representa una forma de ejecutar el mismo código base con diferentes configuraciones.

Para seleccionar el archivo de configuración para cargarlo en cada entorno, Symfony ejecuta el método _registerContainerConfiguration()_ de la clase **AppKernel**:

```
// app/AppKernel.php
use Symfony\Component\HttpKernel\Kernel;
use Symfony\Component\Config\Loader\LoaderInterface;

class AppKernel extends Kernel
{
    // ...
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load($this->getRootDir().'/config/config_'.$this->getEnvironment().'.yml');
    }
}
```

Este método carga el archivo _app/config/config_dev.yml_ para el entorno **dev**, y hará lo mismo para los demás entornos configurados. Este archivo también carga la configuración común que está en el archivo _app/config/config.yml_. La estructura básica que siguen los archivos de configuración de la Edición Estándar de Symfony es la siguiente:
```
your-project/
├─ app/
│  ├─ ...
│  └─ config/
│     ├─ config.yml
│     ├─ config_dev.yml
│     ├─ config_prod.yml
│     ├─ config_test.yml
│     ├─ parameters.yml
│     ├─ parameters.yml.dist
│     ├─ routing.yml
│     ├─ routing_dev.yml
│     └─ security.yml
├─ ...
```

Esta estructura por defecto es así por simplicidad, un archivo por entorno, pero se puede configurar como se quiera. En los siguientes apartados veremos diferentes formas de organizar los archivos de configuración (lo veremos sólo en los entornos dev y prod).

### Diferentes directorios por entorno

En lugar de añadir sufijos a los archivos con _dev y _prod, esta técnica agrupa todos los archivos de configuración relacionados bajo un directorio con el mismo nombre del entorno:
```
your-project/
├─ app/
│  ├─ ...
│  └─ config/
│     ├─ common/
│     │  ├─ config.yml
│     │  ├─ parameters.yml
│     │  ├─ routing.yml
│     │  └─ security.yml
│     ├─ dev/
│     │  ├─ config.yml
│     │  ├─ parameters.yml
│     │  ├─ routing.yml
│     │  └─ security.yml
│     └─ prod/
│        ├─ config.yml
│        ├─ parameters.yml
│        ├─ routing.yml
│        └─ security.yml
├─ ...
```

Para hacer que funcione sólo hay que cambiar el método _registerContainerConfiguration()_:

```
// app/AppKernel.php
use Symfony\Component\HttpKernel\Kernel;
use Symfony\Component\Config\Loader\LoaderInterface;

class AppKernel extends Kernel
{
    // ...

    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load($this->getRootDir().'/config/'.$this->getEnvironment().'/config.yml');
    }
}
```

Después hay que asegurarse de que cada archivo _config.yml_ carga el resto de los archivos de configuración, incluyendo los archivos en **common**. Por ejemplo, esta sería la importación necesaria para el archivo _app/config/dev/config.yml_:

```
# app/config/dev/config.yml
imports:
- { resource: '../common/config.yml' }
    - { resource: 'parameters.yml' }
    - { resource: 'security.yml' }
# ...
```

Debido a la forma en que se resuelven los parámetros, no puedes utilizarlos para construir directorios en imports de forma dinámica. Esto significa que no funcionaría algo como lo siguiente:

```
# app/config/config.yml
imports:
    - { resource: '%kernel.root_dir%/parameters.yml' }
```

### Archivos de configuración semánticos

Una estrategia de organización diferente puede ser necesaria para aplicaciones complejas con largos archivos de configuración. Por ejemplo, puedes crear un archivo por bundle y varios archivos para definir todos los servicios de la aplicación:
```
your-project/
├─ app/
│  ├─ ...
│  └─ config/
│     ├─ bundles/
│     │  ├─ bundle1.yml
│     │  ├─ bundle2.yml
│     │  ├─ ...
│     │  └─ bundleN.yml
│     ├─ environments/
│     │  ├─ common.yml
│     │  ├─ dev.yml
│     │  └─ prod.yml
│     ├─ routing/
│     │  ├─ common.yml
│     │  ├─ dev.yml
│     │  └─ prod.yml
│     └─ services/
│        ├─ frontend.yml
│        ├─ backend.yml
│        ├─ ...
│        └─ security.yml
├─ ...
```

De nuevo habrá que cambiar el código del método registerContainerConfiguration() para que Symfony reconozga la nueva configuración:

```
// app/AppKernel.php
use Symfony\Component\HttpKernel\Kernel;
use Symfony\Component\Config\Loader\LoaderInterface;

class AppKernel extends Kernel
{
    // ...
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load($this->getRootDir().'/config/environments/'.$this->getEnvironment().'.yml');
    }
}
```

Al igual que con la técnica anterior, hay que asegurarse de importar los archivos de configuración apropiados para cada archivo principal (_common.yml_, _dev.yml_ y _prod.yml_). 

### Técnicas de configuración avanzadas

#### Cargar y mezclar formatos de configuración

Symfony carga los archivos de configuración mediante el componente Config, que proporciona características más avanzadas para configuraciones.

Los archivos de configuración pueden importar archivos definidos en cualquier otro formato de configuración (.yml, .xml, .php, .ini).

```
# app/config/config.yml
imports:
    - { resource: 'parameters.yml' }
    - { resource: 'services.xml' }
    - { resource: 'security.yml' }
    - { resource: 'legacy.php' }
# ...
```

La clase **IniFileLoader** analiza los contenidos del archivo con la función [parse_ini_function()](http://php.net/manual/en/function.parse-ini-file.php). Por lo tanto, sólo se pueden establecer parámetros mediante strings. Puedes emplear cualquiera de los otros loaders para otros data types (boolean, integer, etc).

Si usas cualquier otro formato de configuración, debes definir tu propia clase loader extendiéndola de **FileLoader**. Cuando los valores de configuración son dinámicos, puedes usar el **archivo de configuración PHP** para ejecutar tu propia lógica. Además, puedes definir tus propios services para cargar configuraciones desde **databases** o **web services**.

#### Archivos de configuración globales

Algunos administradores pueden preferir guardar parámetros sensibles en archivos fuera del directorio del proyecto. Por ejemplo si los datos de configuración de la base de datos para el sitio web están en el archivo _/etc/sites/misitio.com/parameters.yml_. Cargar este archivo es tan simple como indicar el directorio del archivo cuando se importa desde otro archivo de configuración:

```
# app/config/config.yml
imports:
- { resource: 'parameters.yml' }
    - { resource: '/etc/sites/misitio.com/parameters.yml' }
# ...
```

La mayoría del tiempo los que desarrollan de forma local no tendrán los mismos archivos que existen en los servidores de producción. Por esta razón, el componente Config proporciona la opción ignore_errors para silenciar los errores cuando el archivo a cargar no existe:

```
# app/config/config.yml
imports:
- { resource: 'parameters.yml' }
    - { resource: '/etc/sites/mysite.com/parameters.yml', ignore_errors: true }
# ...
```

Como se ha comprobado existen muchas formas de organizar los archivos de configuración, pueden emplearse cualquiera de los métodos anteriores o crear nuevos para personalizar la configuración.