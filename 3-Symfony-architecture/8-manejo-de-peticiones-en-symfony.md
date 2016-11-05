El **proceso de petición-respuesta de Symfony** se basa en el componente [HttpKernel](http://symfony.com/doc/current/components/http_kernel/introduction.html) haciendo uso del [EventDispatcher](http://symfony.com/doc/current/components/event_dispatcher/introduction.html). El proceso comienza en el archivo _web/app.php_:

```
$kernel = new AppKernel('prod', false);
$request = Request::createFromGlobals();
$response = $kernel->handle($request);
$response->send();
$kernel->terminate($request, $response);
```

Primero se instancia **AppKernel** (_$kernel = new AppKernel('prod', false)_) con el que se selecciona la configuración básica que ha de cargar la aplicación. Puede ser cualquier _string,_ por defecto hay 3 posibilidades: **prod**, **dev** y **test**. La función **registerContainerConfiguration** es la que carga los archivos de configuración:

```
// app/AppKernel.php
public function registerContainerConfiguration(LoaderInterface $loader)
{
    $loader->load($this->getRootDir().'/config/config_'.$this->getEnvironment().'.yml');
}
```

En este caso es **prod**, y el **debug mode **es false. El **modo debug** se aplica para el **entorno de desarrollo** (si vas a _app/app_dev.php_ verás que esta opción es true) y permite ver **errores y excepciones** con mayor detalle, **invalida la caché** y proporciona más información del **web profiler**.

Después se crea un **objeto Request** (_$request = Request::createFromGlobals()_) a partir de las **variables PHP superglobals**. Este objeto te permitirá poder obtener [cualquier información que se obtiene de las superglobals](http://symfony.com/doc/current/book/http_fundamentals.html#requests-and-responses-in-symfony), como:

```
$request->getPathInfo(); // Devuelve la URI de la petición
$request->server->get('HTTP_HOST'); // Devuelve variables del servidor
$request->getMethod(); // Devueve el método (GET, POST, PUT, DELETE, HEAD)
```

El método **handle()** es la base a la hora de **procesar las peticiones** (_$response = $kernel->handle($request)_), y su resultado será un **objeto Response**, que será devuelto al cliente que hizo la petición. Internamente este **método se encarga de enviar eventos**, ejecutando funciones mediante **event listeners**. Este método iniciará todos los **bundles registrados** además del **service container**.

Los bundles se registran en el _AppKernel.php_ (es el paso principal a la hora de instalar cualquier bundle en una aplicación Symfony).

En el proceso de arranque los bundles de la aplicación pueden **registrar sus propios servicios al service container** o **modificar parámetros** antes de que el contenedor se termine de construir. Es por ello que suele haber un archivo llamado _AppBundleExtension.php_ dentro de la carpeta **DependencyInjection**:

```
namespace AppBundle\DependencyInjection;

use Symfony\Component\DependencyInjection\Loader;
use Symfony\Component\Config\FileLocator;
use Symfony\Component\HttpKernel\DependencyInjection\Extension;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class AppBundleExtension extends Extension
    {
        public function load(array $configs, ContainerBuilder $container)
        {
            $configuration = new Configuration();
            $config = $this->processConfiguration($configuration, $configs);

            $loader = new Loader\YamlFileLoader($container, new FileLocator(__DIR__.'/../Resources/config'));
            $loader->load('services.yml');
        }
    }
```

Una vez que los **bundles** se han cargado y han añadido sus **servicios** y **parámetros**, se crean dos archivos, uno **XML** con definiciones y parámetros, y uno **PHP** que será el **service container final** de la aplicación. Estos archivos se guardan en la caché del entorno correspondiente. El archivo PHP contiene un método por cada servicio que han generado los bundles de la aplicación. 

En el método **handle()** se incluye el método **handleRaw()** (_Symfony\Component\HttpKernel_), que permite poder **manejar la petición**, registrar **event listeners** para dar respuestas y **devolver controladores**. El evento **kernel.request** es primero que se ejecuta. Añade información al request, inicia partes del sistema o genera respuestas concretas. Entre sus diferentes listeners se encuentra el **RouterListener**. Este es el que se encarga de recibir la **URI** del **objeto Request** y casarlo con una ruta. El evento **kernel.controller** inicia el proceso o cambia el controlador antes de ejecutarlo. 

En un archivo de _routing.yml_ así puede ser la configuración:

```
main_page:
    pattern:  /
    defaults: { _controller: AppBundle:Default:index }
```

La función de devolver controladores se realiza a través del objeto **ControllerResolver**, que ejecuta los controladores. Estos son los que devuelven los objetos **Response**, que permitirán devolver una respuesta al cliente. Si el controlador no proporciona una respuesta, el kernel lanza otro evento: el **evento kernel.view**. La función de un listener de este evento es devolver valores del controlador (como un array de datos) para crear la respuesta. Esto permite que se pueda emplear un **template engine** como **Twig**:

```
class PagesController extends Controller {

    public function aboutAction()
    {
        return $this->render('AppBundle:Pages:about.html.twig');
    }
```

Un último evento es lanzado antes de generar la respuesta: el **kernel.response**. Su función es modificar la respuesta justo antes de enviarla (añadir cookies, cambiar el contenido de la respuesta final...).

Pero hay un último evento que se encarga de tomar alguna acción de mayor peso justo después de haber enviado la petición, se trata del **evento kernel.terminate**. Este evento permite poder enviar la respuesta con mayor rapidez. Un ejemplo es el **envío de emails**. Al usar el **SwiftmailerBundle** puedes activar el **memory spooling**, que activa el **EmailSenderListener** y envía los emails que se hayan programado para enviar durante la petición.