Un bundle es como un plugin es cualquier otro software. En **Symfony** todo es un bundle, desde el núcleo del **framework** hasta el código que escribes para tu aplicación. Un **bundle** es un conjunto estructurado de archivos (PHP, CSS, JavaScript, imágenes...) que implementan una funcionalidad individual (un blog, un foro...) y que se puede compartir con otros desarrolladores.

Los **bundles** permiten la **flexibilidad** de poder usar funcionalidades de bundles de terceros o distribuir tus propios bundles, así puedes **optimizar tu aplicación** de la forma que quieras.

Symfony incluye por defecto el bundle **AppBundle** que puedes usar para empezar a desarrollar el proyecto. A partir de ahí puedes ir creando más bundles si tu aplicación lo requiere.

**Indice de contenido**

1.  Registrar un bundle
2.  Crear un bundle
3.  Configurar un bundle
4.  Referencias y herencias

### 1. Registrar un bundle

Una **aplicación** está construida a partir de bundles definidos en el método _registerBundles()_ de la clase **AppKernel**. Cada **bundle** es un directorio que contiene una clase individual **Bundle** que lo describe:

```
// app/AppKernel.php
public function registerBundles()
{
    $bundles = array(
        new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
        new Symfony\Bundle\SecurityBundle\SecurityBundle(),
        new Symfony\Bundle\TwigBundle\TwigBundle(),
        new Symfony\Bundle\MonologBundle\MonologBundle(),
        new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
        new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
        new Symfony\Bundle\AsseticBundle\AsseticBundle(),
        new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
        new AppBundle\AppBundle();
    );

    if (in_array($this->getEnvironment(), array('dev', 'test'))) {
        $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
        $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
        $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
    }

    return $bundles;
}
```

Además del AppBundle, el **kernel** habilita también otros bundles que son parte de Symfony, como el **FrameworkBundle**, **DoctrineBundle**, **SwiftmailerBundle** o **AsseticBundle**. 

Un bundle puede estar en cualquier parte del script siempre y cuando pueda autocargarse con el autoloader configurado en _app/autoload.php_.

### 2. Crear un bundle

Para ver como crear un bundle vamos a crear uno llamado **AcmeTestBundle**. La porción **Acme** es simplemente un nombre que representa la persona o institución que crea el bundle, no es obligatorio aunque incluirlo forma parte de las convenciones de nombres de bundles (podríamos crearlo como TestBundle). Por ejemplo, CompanyTestBundle será un bundle llamado TestBundle creado por la compañía Company.

Procedemos a crear el directorio _src/Acme/TestBundle_ y creamos en él un nuevo archivo llamado _AcmeTestBundle.php_:

```
namespace Acme\TestBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeTestBundle extends Bundle
{
}
```

Si hubiéramos llamado al bundle TestBundle el archivo y clase serían TestBundle, así como su namespace.

Esta clase vacía es lo único que se necesita para crear un bundle. Aunque normalmente está vacía, puede utilizarse para customizar el comportamiento del bundle.

Una vez creado el bundle, lo activamos a través de la clase AppKernel en _app/AppKernel.php_:

```
public function registerBundles()
{
    $bundles = array(
        // ...
        // Registramos el bundle
        new Acme\TestBundle\AcmeTestBundle(),
    );
    // ...
    return $bundles;
}
```

Aunque no haga nada de momento ya está listo para ser usado.

También existe un comando para generar el skeleton de un bundle básico:
```
php bin/console generate:bundle --namespace=Acme/TestBundle
```

El skeleton genera un controlador básico, template y routing resource que pueden customizarse.

Es importante recordar que siempre que se crea un nuevo bundle o se usa uno de un tercero hay que registrarlo en registerBundles(). Cuando se usa el comando esto se hace automáticamente.

### 3. Configurar un bundle

Cada bundle se puede configurar con archivos de configuración escritos en **YAML**, **XML**, o **PHP**. Echa un vistazo a este ejemplo de una **configuración Symfony por defecto**:

```
# app/config/config.yml
imports:
    - { resource: parameters.yml }
    - { resource: security.yml }
    - { resource: services.yml }

framework:
    #esi:             ~
    #translator:      { fallbacks: ["%locale%"] }
    secret:          "%secret%"
    router:
        resource: "%kernel.root_dir%/config/routing.yml"
        strict_requirements: "%kernel.debug%"
    form:            true
    csrf_protection: true
    validation:      { enable_annotations: true }
    templating:      { engines: ['twig'] }
    default_locale:  "%locale%"
    trusted_proxies: ~
    session:         ~

# Twig Configuration
twig:
    debug:            "%kernel.debug%"
    strict_variables: "%kernel.debug%"

# Swift Mailer Configuration
swiftmailer:
    transport: "%mailer_transport%"
    host:      "%mailer_host%"
    username:  "%mailer_user%"
    password:  "%mailer_password%"
    spool:     { type: memory }

# ...
```

Cada primer nivel (framework, twig, swiftmailer) define la configuración para cada **bundle** específico. Cada **entorno** puede sobreescribir la configuración por defecto empleando un archivo de configuración específico. Por ejemplo el **entorno de desarrollo** _**dev**_ carga el _config_dev.yml_, que carga la configuración principal (_config.yml_) y la modifica añadiendo **herramientas de depuración**:

```
# app/config/config_dev.yml
imports:
    - { resource: config.yml }

framework:
    router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
    profiler: { only_exceptions: false }

web_profiler:
    toolbar: true
    intercept_redirects: false

# ...
```

### 4\. Referencias y herencias

Un **bundle** puede extender otro bundle. La **herencia de bundles** permite sobreescribir cualquier bundle existente para customizar sus **controladores**, **plantillas** o cualquiera de sus **archivos**.

#### Referencias a archivos

Cuando quieres referenciar un archivo desde un bundle, usa esta notación: **@NOMBRE_BUNDLE/directorio/al/archivo**; Symfony resolverá **@NOMBRE_BUNDLE** apuntando al directorio real del bundle. Por ejemplo _@AppBundle/Controller/DefaultController.php_ se convertiría en _src/AppBundle/Controller/DefaultController.php_, porque **Symfony** conoce la localización del AppBundle.

#### Referencias en controladores

Para controladores, necesitas referenciar acciones usando el formato **NOMBRE_BUNDLE:NOMBRE_CONTROLADOR:NOMBRE_ACCION**. Por ejemplo, **AppBundle:Default:index** dirige al método indexAction de la clase _AppBundle\Controller\DefaultController_.

#### Extendiendo bundles

Si sigues estas convenciones, puedes usar la herencia de bundles para sobreescribir archivos, controladores o plantillas. Si por ejemplo creamos un nuevo bundle **NuevoBundle** y especificamos que sobreescribe a **AppBundle**, cuando Symfony inicie el **controlador** AppBundle:Default:index, buscará primero la clase DefaultController en el NuevoBundle, y si no existe, lo buscará en AppBundle. Un bundle puede extender casi cualquier otra parte de otro bundle.