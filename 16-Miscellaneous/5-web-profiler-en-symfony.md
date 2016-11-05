El **Web Profiler** y la **Web Debug Toolbar** son dos aspectos importantes para el **desarrollo de aplicaciones web con Symfony**. Utilizar un profiler en un sitio web es muy útil no sólo para **debugging** y **testear el rendimiento** sino también para ver cómo tu sitio maneja los requests. El web profiler recopila toda esta información y la web debug toolbar permite mostrarla de forma visual y accesible. 

![Web debug toolbar Symfony 3](https://farm2.staticflickr.com/1634/26652396382_57c05e5064_z.jpg)

Dejando el ratón sobre cualquiera de los botones te mostrará información revelante. Haciendo click en cualquiera de ellos te dirigirá al profiler donde podrás ver información más detallada. En la barra de la imagen se pueden ver los siguientes campos: información del request, velocidad de carga, uso de memoria, formularios, traducciones, usuario, renderización de plantillas, llamadas a la base de datos (**Doctrine** y **ElasticSearch**) y **EasyAdminBundle**.

Además de los botones que vienen por defecto, cada bundle puede [añadir su propia template a la toolbar](https://diego.com.es/crear-un-data-collector-en-symfony).

Puedes cambiar la [configuración básica del web profiler](http://symfony.com/doc/current/reference/configuration/web_profiler.html):

```
web_profiler:
    # muestra la debug toolbar en la parte de abajo
    # con el sumario de la información del profiler
    toolbar:              false
    position:             bottom

    # permite ver los datos recopilados
    # antes de continuar la redirección
    intercept_redirects: false

    # Excluye AJAX requests en la toolbar para paths específicos
    excluded_ajax_paths:  ^/bundles|^/_wdt
```

**Indice de contenido**

1.  [Activar el profiler de forma condicional](#ActivarElProfilerDeFormaCondicional)
2.  [Cambiar el almacenamiento del profiler](#CambiarElAlmacenamientoDelProfiler)
3.  [Acceder a datos del profiler de forma programática](#AccederADatosDelProfilerDeFormaProgramatica)

### <a id="ActivarElProfilerDeFormaCondicional" name="ActivarElProfilerDeFormaCondicional"></a>Activar el profiler de forma condicional

El profiler se activa sólo en el entorno de desarrollo, pero puedes utilizar matchers para activar el profiler de forma condicional.

Un _request matcher_ es una clase que comprueba si una instancia **Request** dada coincide con una serie de condiciones. Symfony proporciona un _built-in matcher_ que comprueba _paths_ e _ips_. Por ejemplo si quieres sólo mostrar el profiler cuando se accede a la página con la IP 168.0.0.1, puedes emplera esta configuración:

```
# app/config/config.yml
framework:
    # ...
    profiler:
        matcher:
            ip: 168.0.0.1
```

Puedes también establecer una opción _path_ para definir el path en el que el profiler debería activarse. Por ejemplo estableciéndolo a _^/admin/_ activará el profiler sólo para las URLs que comienzan con _/admin/_.

#### Crear un matcher customizado

Para crear un matcher customizado creamos una clase que implementa [RequestMatcherInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/RequestMatcherInterface.html). Esta interface requiere un método: [matches()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/RequestMatcherInterface.html#method_matches). Este método devuelve _false_ cuando el request no coincide con las condiciones, y _true_ si coincide. Por tanto, el matcher customizado debe devolver _false_ para desactivar el profiler y _true_ para activarlo. 

Supón que el profiler ha de activarse cuando un usuario con el role **ROLE_SUPER_ADMIN** se logea. Este es el único código necesario para ese matcher customizado. 

```
// src/AppBundle/Profiler/SuperAdminMatcher.php
namespace AppBundle\Profiler;

use Symfony\Component\Security\Core\Authorization\AuthorizationCheckerInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\RequestMatcherInterface;

class SuperAdminMatcher implements RequestMatcherInterface
{
    protected $authorizationChecker;

    public function __construct(AuthorizationCheckerInterface $authorizationChecker)
    {
        $this->authorizationChecker = $authorizationChecker;
    }

    public function matches(Request $request)
    {
        return $this->authorizationChecker->isGranted('ROLE_SUPER_ADMIN');
    }
}
```

Configura entonces un nuevo _service_ y establécelo como _private_ porque la aplicación no lo usará directamente:

```
# app/config/services.yml
services:
    app.super_admin_matcher:
        class: AppBundle\Profiler\SuperAdminMatcher
        arguments: ['@security.authorization_checker']
        public: false
```

Una vez que el service está registrado, lo único que falta es configurar el profiler para usar este service como el matcher:

```
# app/config/config.yml
framework:
    # ...
    profiler:
        matcher:
            service: app.super_admin_matcher
```

### <a id="CambiarElAlmacenamientoDelProfiler" name="CambiarElAlmacenamientoDelProfiler"></a>Cambiar el almacenamiento del profiler

En versiones de Symfony anteriores a la 3, los profiles podían guardarse en archivos, bases de datos, servicios como Redis y Memcache, etc. A partir de Symfony 3 el único mecanismo con soporte incorporado es mediante archivos.

Por defecto el profile guarda los datos recopilados en el directorio _%kernel.cache_dir%/profiler/_. Si quieres utilizar otra localización para guardar los profiles, define la opción _dsn_ del _framework.profiler_:

```
# app/config/config.yml
framework:
    profiler:
        dsn: 'file:/tmp/symfony/profiler'
```

Puedes crear también tu propio _profile storage service_ implementando la clase _Symfony\Component\HttpKernel\Profiler\ProfilerStorageInterface_ y sobreescribiendo el service _profiler.storage_.

### <a id="AccederADatosDelProfilerDeFormaProgramatica" name="AccederADatosDelProfilerDeFormaProgramatica"></a>Acceder a datos del profiler de forma programática

La mayoría de las veces se accede a la información del profiler con la toolbar. Sin embargo, también puedes obtener esa información de forma programática gracias a los métodos proporcionados por el _profiler_ service. 

Cuando el objeto response está disponible, utiliza el método [loadProfileFromResponse()](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/Profiler/Profiler.html#method_loadProfileFromResponse) para acceder a su profile asociado.

```
// ... $profiler es el service 'profiler'
$profile = $profiler->loadProfileFromResponse($response);
```

Cuando el profiler almacena datos de un request, también asocia un token. El token está disponible en el **header HTTP X-Debug-Token** de la respuesta. Con este token, puedes acceder al profile de cualquier respuesta anterior gracias al método [loadProfile()](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/Profiler/Profiler.html#method_loadProfile):

```
$token = $response->headers->get('X-Debug-Token');
$profile = $container->get('profiler')->loadProfile($token);
```

Cuando está activado el profiler pero no la toolbar, inspecciona la página con las **developer tools** de tu navegador para obtener el valor del HTTP header _X-Debug-Token_.

El profiler service también proporciona el método [find()](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/Profiler/Profiler.html#method_find) para buscar tokens basándose en algún criterio. 

```
// obtener los últimos 10 tokens
$tokens = $container->get('profiler')->find('', '', 10, '', '');

// obtener los últimos 10 tokens para todas las URL que contienen /admin/
$tokens = $container->get('profiler')->find('', '/admin/', 10, '', '');

// obtener los últimos 10 tokens de requests locales
$tokens = $container->get('profiler')->find('127.0.0.1', '', 10, '', '');

// obtener los últimos 10 tokens para requests que ocurrieron entre 2 y 4 días atrás
$tokens = $container->get('profiler')
    ->find('', '', 10, '4 days ago', '2 days ago');
```

Finalmente, si quieres manipular datos del profiler en una máquina diferente a la cual la información fue generada, emplea los comandos _profiler:export_ y _profiler:import_:

```
# en la máquina de producción
$ php bin/console profiler:export > profile.data

# en la máquina de desarrollo
$ php bin/console profiler:import /path/to/profile.data

# puedes también usar pipe desde el STDIN
$ cat /path/to/profile.data | php bin/console profiler:import
```