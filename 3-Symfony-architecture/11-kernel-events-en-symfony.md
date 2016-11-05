El componente **HttpKernel** proporciona una forma estructurada de convertir un **Request** en un **Response** mediante el componente **EventDispatcher**. Es suficientemente flexible para crear un framework (**Symfony**), un micro-framework (**Silex**) o un sistema avanzado de CMS (**Drupal**).

```
namespace Symfony\Component\HttpKernel;

use Symfony\Component\HttpFoundation\Request;

interface HttpKernelInterface
{
    // ...

    /**
     * @return Response A Response instance
     */
    public function handle(
        Request $request,
        $type = self::MASTER_REQUEST,
        $catch = true
    );
}
```

Internamente, _HttpKernel::handle()_ define un workflow que comienza en un Request y termina en un Response. Los detalles de este workflow son la clave para entender como funciona el **kernel** y **Symfony**.

El método _HttpKernel::handle()_ funciona internamente lanzando **eventos**, por lo que todo el trabajo del framework y de una aplicación es a través de **listeners**.

El uso de HttpKernel es muy sencillo e implica crear un **event dispatcher** y un **controller resolver**. Para completas el kernel, tú mismo añadirás listeners a los eventos que ocurren durante el proceso.

```
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpKernel\HttpKernel;
use Symfony\Component\EventDispatcher\EventDispatcher;
use Symfony\Component\HttpKernel\Controller\ControllerResolver;

// Crear el objeto Request
$request = Request::createFromGlobals();
$dispatcher = new EventDispatcher();

// ... Añadir algunos listeners

// Crear el controller resolver
$resolver = new ControllerResolver();
// Instanciar el kernel
$kernel = new HttpKernel($dispatcher, $resolver);

// Ejecutar el kernel, que transforma el request en response
// lanzando eventos, llamando a un controller, y devolviendo la respuesta
$response = $kernel->handle($request);

// Enviar los headers y mostrar el contenido
$response->send();

// Dispara el evento kernel.terminate
$kernel->terminate($request, $response);
```

**Indice de contenido**

| | |
| -------- | -------- |
| 1\. [El evento kernel.request](#ElEventoKernelRequest) | 7\. [El evento kernel.response](#ElEventoKernelResponse) |
| 2\. [Resolver el Controller](#ResolverElController) | 8\. [El evento kernel.terminate](#ElEventoKernelTerminate) |
| 3\. [El evento kernel.controller](#ElEventoKernelController) | 9\. [El evento kernel.exception](#ElEventoKernelException) |
| 4\. [Obtener los argumentos del Controller](#ObtenerLosArgumentosDelController) | 10\. [Crear un Event Listener](#CrearUnEventListener) |
| 5\. [Llamar al Controller](#LlamarAlController) | 11\. [Ejemplo del HttpKernel](#EjemploDeHttpKernel) |
| 6\. [El evento kernel.view](#ElEventoKernelView) | 12\. [Sub Requests](#SubRequests) |

### <a id="ElEventoKernelRequest" name="ElEventoKernelRequest"></a>1\. El evento kernel.request

**Uso típico**: Añadir más información al request, iniciar partes del sistema o devolver un **Response** si es necesario.

![Evento kernel.request](http://symfony.com/doc/current/_images/02-kernel-request.png)

Es el primer evento lanzado desde _HttpKernel::handle_, y puede tener diferentes **listeners**. Algunos de ellos (como alguno de seguridad) puede tener suficiente información como para crear un objeto **Response** inmediatamente. Si un listener de seguridad determina que un usuario no tiene acceso, el listener puede devolver un **RedirectResponse** a la página de login o responder con un código de estado **403 Access Denied response**.

Si se devuelve una respuesta en esta etapa el proceso salta directamente al evento _kernel.response_:

![Evento kernel.request con respuesta inmediata](http://symfony.com/doc/current/_images/03-kernel-request-response.png)

Otros listeners simplemente inician procesos o añaden más información al request. Por ejemplo, un listener podría determinar y establecer el locale en el objeto **Request**.

Otro listener muy común es routing. Un router listener procesa el Request y determina el controller que ha de ser renderizado. El objeto Request tiene una **bolsa de atributos** donde se guardan datos específicos del request en la aplicación. Esto significa que si tu router listener determina el controlador, puede guardarlo en los atributos Request (que pueden ser usados por el controller resolver).

Resumiendo, el objetivo del evento _kernel.request_ es o crear un Response directamente o añadir información al Request (añadiendo el locale u otra información en los atributos Request).

Cuando se establece una respuesta en el evento _kernel.request_ se para la propagación, por lo que los listeners con menor prioridad nunca serán ejecutados.

El listener más importante del _kernel.request_ en Symfony es el **RouterListener**. Esta clase ejecuta el routing layer, que devuelve un _array_ de información sobre el request, incluyendo el _controller y cualquier otro placeholder que esté en la route (por ejemplo, _{slug}_). Este array de información se guarda en el array de atributos del objeto **Request**. Añadir la información de routing aquí no hará nada, pero se usa después cuando se resuelve el controller.

### <a id="ResolverElController" name="ResolverElController"></a>2\. Resolver el Controller

Si ningún listener del _kernel.request_ ha creado un objeto **Response**, el siguiente paso en el **HttpKernel** es determinar y preparar el controller. El controller es la parte de la aplicación que se encarga de crear y devolver un objeto Response para una página específica. El único requisito es que tiene que ser un callable PHP (función, método de un objeto o Closure).

Cómo determinar el controller exacto para un request depende totalmente de tu aplicación. Esta es la tarea del "**_controller resolver_**", una clase que implementa **ControllerResolverInterface** y que forma parte de los argumentos constructores del HttpKernel.

![El controller resolver de Symfony](http://symfony.com/doc/current/_images/04-resolve-controller.png)

Tu tarea es crear una clase que implemente la interface y cumpla con sus dos métodos: _getController_ y _getArguments_. De hecho ya existe una implementación por defecto, que puedes usar directamente o utilizarla para ver cómo funciona: **ControllerResolver**.

```
namespace Symfony\Component\HttpKernel\Controller;

use Symfony\Component\HttpFoundation\Request;

interface ControllerResolverInterface
{
    public function getController(Request $request);

    public function getArguments(Request $request, $controller);
}
```

Internamente, el método _HttpKernel::handle_ primero llama a _getController()_ en el controller resolver. A este método se le pasa el **Request** y es responsable de alguna forma de determinar y devolver un **callable PHP** (el controller) basándose en la información del request.

El segundo método, _getArguments()_, será llamado después de que otro evento, _kernel.controller_, sea lanzado.

#### Resolver el Controller en Symfony

El **framework de Symfony** utiliza la clase **ControllerResolver** (bueno realmente utiliza una subclase con alguna funcionalidad extra). Esta clase trata la información de la propiedad _**attributes**_ del objeto Request durante el **RouterListener**.

El ControllerResolver busca una key **_controller** en la propiedad attributes del objeto Request. Este string es entonces transformado en un callable PHP a través de los siguientes pasos:

1.  El formato _AcmeDemoBundle:Default:index_ en el key _controller se cambia a otro string que contiene la clase entera y el nombre del método del controller siguiendo las convenciones usadad en **Symfony**: _Acme\DemoBundle\Controller\DefaultController::indexAction_. Esta transformación es específica a la subclase **ControllerResolver** utilizada en el framework.
2.  Una nueva instancia de la clase de tu controller es instanciada sin argumentos constructores.
3.  Si el controller implementa **ControllerAwareInterface**, se llama a _setContainer_ en el objeto controller y se pasa el contenedor. Este paso es específico de la subclase ControllerResolver utilizada en el framework Symfony.

Existen otras variantes del proceso anterior (por ejemplo si registras los controllers como **services**).

### <a id="ElEventoKernelController" name="ElEventoKernelController"></a>3\. El evento kernel.controller

**Uso típico**: iniciar procesos o cambiar el controller justo antes de que el controller sea ejecutado.

Después de que se haya determinado el **controller callable**, _HttpKernel::handle_ lanza el evento _kernel.controller_. Los listeners de este evento pueden iniciar alguna parte del sistema que necesite ser iniciada después de que se hayan determinado algunas cosas (como el controller o la información del routing) pero antes de que el controller sea ejecutado. 

![El evento kernel.controller](http://symfony.com/doc/current/_images/06-kernel-controller.png)

Los listeners de este evento pueden tambien cambiar el controller callable completamente llamando al método _FilterControllerEvent::setController_ en el objeto del evento que se pasa a los listeners de este evento.

Hay unos pocos listeners del evento kernel.controller en Symfony, y muchos de ellos están relacionados con la información del profiler cuando está activado.

Un listener a destacar viene del bundle [SensioFrameworkExtraBundle](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/index.html), incluído en la **Symfony Standard Edition**. La funcionalidad del [_@ParamConverter_](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html) de este listener permite pasar un objeto (por ejemplo, un objeto **Post**) al controller en lugar de un valor escalar (por ejemplo, un parámetro _id_ que estaba en la route). El listener es **ParamConverterListener**, y utiliza reflection para ver cada uno de los argumentos del controller e intenta usar diferentes métodos para convertirlos a objetos, los cuales son entonces guardados en la propiedad _attributes_ del objeto **Request**.

### <a id="ObtenerLosArgumentosDelController" name="ObtenerLosArgumentosDelController"></a>4\. Obtener los argumentos del Controller

Después, _HttpKernel::handle_ llama al método _getArguments_. Hay que recordar que el controller devuelto en _getController_ es un callable. El objetivo de _getArguments_ es devolver un _array_ de argumentos que deberían pasarse al controller. Como se hace esto exactamente depende del desarrollador, aunque el [ControllerResolver](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/Controller/ControllerResolver.html) que viene incorporado es un buen ejemplo.

![Obtener los argumentos del controlador en Symfony](http://symfony.com/doc/current/_images/07-controller-arguments.png)

En este punto el kernel tiene un **callable** (el controller) y un **array** de argumentos que deberían pasarse cuando se ejecute ese callable.

Ahora que ya sabemos lo que es el controller callable (normalmente un método dentro de un objeto controller), el **ControllerResolver** utiliza [reflection](http://diego.com.es/reflection-en-php) en el callable para devolver un _array_ con los nombres de cada uno de los argumentos. Es entonces cuando itera sobre cada uno de los argumentos y utiliza lo siguiente para determinar qué valores deben pasarse para cada argumento:

1.  Si el array de atributos del Request contiene un _key_ que coincide con el nombre del argumento, se utiliza ese valor. Por ejemplo, si el primer argumento de un controller es _$slug_, y hay un _key_ **slug** en el array de atributos, se usa ese valor (y normalmente este valor viene del **RouterListener**).
2.  Si el argumento en el controller es type-hinted con el objeto **Request**, entonces el Request se pasa como el valor.

### <a id="LlamarAlController" name="LlamarAlController"></a>5\. Llamar al Controller

El siguiente paso es sencillo: _HttpKernel::handle_ ejecuta el controlador.

![Llamada al controller en Symfony](http://symfony.com/doc/current/_images/08-call-controller.png)

La tarea del controller es construir una respuesta para el _resource_ dado. Esto puede ser una **página HTML**, un **string JSON** o cualquier otra cosa. Al contrario que con los procesos que se han dado hasta ahora, este paso se implementa por el desarrollador, para cada página que se construya.

Normalmente el controller devolverá un objeto Response. Si es true, el trabajo del kernel está casi terminado, el siguiente paso es el evento _kernel.response_.

![Etapas del proceso habitual en Symfony](http://symfony.com/doc/current/_images/09-controller-returns-response.png)

Pero si el controller devuelve cualquier otra cosa que no sea el **Response**, el kernel tiene algo más de trabajo que hacer, el evento _kernel.view_ (ya que el objetivo siempre es generar un objeto Response). Un controller siempre tiene que devolver algo. Si devuelve **null**, se lanzará una excepción. 

### <a id="ElEventoKernelView" name="ElEventoKernelView"></a>6\. El evento kernel.view

**Uso típico**: transformar un valor que no es Response de un controller en un Response.

Si el controller no devuelve un objeto **Response**, el kernel lanza otro evento, el _kernel.view_. La tarea de un listener de este evento es utilizar el valor de retorno del controller (como un array de datos) para crear un Response.

![El evento kernel.view](http://symfony.com/doc/current/_images/10-kernel-view.png)

Esto puede resultar útil si quieres usar un **view layer**: en lugar de devolver un Response del controller, devuelves datos que representan la página. Un listener de este evento podría usar estos datos para crear un Response en el formato correcto (HTML, JSON, etc).

En esta etapa, si ningún listener responde al evento, se lanzará una excepción: ya sea el controller o uno de los view listeners deben siempre devolver un Response.

Cuando se establece una respuesta para el evento _kernel.view_ la propagación se para, lo que significa que los listeners con menor priodidad no se ejecutarán.

No hay ningún listener por defecto para el evento _kernel.view_ en Symfony. Sin embargo, el bundle incorporado SensioFrameworkExtraBundle añade un listener a este evento. Si tu controller devuelve un array y añades la anotación [@Template](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/view.html) encima del controller, este listener renderiza una template, pasa el array que generes en el controller a esa template y crea un Response conteniendo el contenido devuelto de esa template.

Además el popular bundle [FOSRestBundle](https://github.com/friendsofsymfony/FOSRestBundle) implementa un listener en este evento cuyo objeto es proporcionar un view layer capaz de usar un controller para devolver contenidos muy diferentes (HTML, JSON, XML, etc).

### <a id="ElEventoKernelResponse" name="ElEventoKernelResponse"></a>7\. El evento kernel.response

**Uso típico**: modificar el objeto Response justo antes de ser enviado.

El **objetivo final del kernel** es transformar un **Request** en un **Response**. El Response puede ser creado durante el evento _kernel.request,_ devuelto por el controller o devuelto por uno de los listeners del evento _kernel.view_.

Independientemente de quien genere el Response, otro evento, _kernel.response_, se lanza inmediatamente después. Un listener típico de este evento modificará el objeto Response de alguna forma, ya sean los headers, añadir cookies, o incluso cambiar el contenido de la respuesta (inyectar Javascript antes del final de la etiqueta </body> de una respuesta HTML).

Una vez que se lance este evento, el objeto final Response se devuelve desde _handle()_. En el caso más típico, puedes llamar entonces el método _send()_, que envía los headers y devuelve el contenido de Response.

En Symfony hay pocos listeners de este evento, y la mayoría modifican la respuesta de alguna forma. Por ejemplo el **WebDebugToolbarListener** inserta **Javascript** en la parte de abajo del sitio web en el entorno **dev** que hace que se muestre la barra de debugging. Otro listener, **ContextListener**, serializa la información actual del usuario en la sesión de forma que puede cargarse en el siguiente request.

### <a id="ElEventoKernelTerminate" name="ElEventoKernelTerminate"></a>8\. El evento kernel.terminate

**Uso típico**: para llevar acabo acciones pesadas después de que la respuesta se haya enviado al usuario. 

El evento final en el proceso **HttpKernel** es _kernel.terminate_ y es único porque ocurre después del método _HttpKernel::handle_ y después de que la respuesta se haya enviado al usuario.

```
// Enviar los headers y mostrar el contenido
$response->send();

// Lanzar el evento kernel.terminate
$kernel->terminate($request, $response);
```

Llamando a _$kernel->terminate_ después de enviar la respuesta se lanza el evento _kernel.terminate_ donde se pueden realizar ciertas tareas que se han dejado para después para que la respuesta cargue más rápido (como enviar emails).

Internamente el HttpKernel utiliza la función PHP [fastcgi_finish_request](http://php.net/manual/en/function.fastcgi-finish-request.php). Esto significa que por el momento, sólo el API del servidor [PHP FPM](http://php.net/manual/en/install.fpm.php) puede enviar una respuesta al cliente mientras el proceso PHP del servidor todavía realiza algunas tareas. Con otras APIs de servidor, los listeners del evento _kernel.terminate_ todavía se ejecutan, pero la respuesta no se envía al cliente hasta que se completan del todo.

Usar el evento _kernel.terminate_ es opcional, y sólo debería aplicarse si tu kernel implementa TerminableInterface.

Si utilizas el **SwiftMailerBundle** con Symfony y empleas **memory spooling**, entonces el **EmailSenderListener** está activado, y envía cualquier email programado para ser enviado durante el request.

### <a id="ElEventoKernelException" name="ElEventoKernelException"></a>9\. El evento kernel.exception

**Uso típico**: manejar algún tipo de excepción y crear un Response apropiado para devolver la excepción.

Si se lanza una excepción en cualquier punto de _HttpKernel::handle_, se lanza el evento _kernel.exception_. Internamente el cuerpo de la función handle está dentro de un bloque try/catch. Cuando se lanza una excepción, el evento _kernel.exception_ se lanza de forma que el sistema puede responder a la excepción.

![El evento kernel.exception en Symfony](http://symfony.com/doc/current/_images/11-kernel-exception.png)

A cada listener de este evento se le pasa un objeto **GetResponseForExceptionEvent**, que puedes utilizar para acceder a la excepción original a través del método _getException()_. Un listener típico de este evento comprobará el tipo concreto de excepción y creará un Response de error para ese tipo.

Por ejemplo, para **generar una página de error 404**, puedes lanzar un tipo especial de excepción y entonces añadir un listener de este evento que busque esta excepción y cree y devuelva un 404 Response. De hecho, el componente HttpKernel viene con un **ExceptionListener**, que si decides usarlo, hará esto y más por defecto.

Cuando se establece una respuesta para el evento _kernel.exception_, la propagación se para, por lo que listeners con menor prioridad no se ejecutarán.

En Symfony existen **dos listeners principales** para el evento _kernel.exception_:

*   [ExceptionListener en HttpKernel](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/EventListener/ExceptionListener.html). Este listener tiene tres objetivos:

1\. La excepción lanzada se convierte en un objeto [FlattenException](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/Exception/FlattenException.html), que contiene toda la información sobre el request y puede ser impresa o serializada.

2\. Si la excepción original implementa **HttpExceptionInterface**, se llama a _getStatusCode()_ y _getHeaders()_ y se utilizan para rellenar los headers y códigos de estado del objeto **FlattenException**. La idea es que se utilicen en el siguiente paso a la hora de crear una respuesta final.

3\. Se ejecuta un controlador y se pasa el objeto FlattenException. El controlador a renderizar se pasa como argumento del constructor de este listener. El controlador devolverá el objeto Response final para esta página de error.

*   [ExceptionListener en Security](http://api.symfony.com/3.0/Symfony/Component/Security/Http/Firewall/ExceptionListener.html). El objetivo de este listener es manejar excepciones de seguridad, y cuando sea apropiado ayudar al usuario a autenticarse (por ejemplo redireccionar a la página de login).

### <a id="CrearUnEventListener" name="CrearUnEventListener"></a>10\. Crear un Event Listener

Puedes crear y enlazar **event listeners** a cualquiera de los eventos lanzados durante el ciclo _HttpKernel::handle_. El nombre de cada uno de los **kernel events** es definido como una constante en la clase **KernelEvents**. Además, a cada event listener se le pasa un argumento, el cual es una subcalse de KernelEvent. Este objeto contiene información acerca del estado actual del sistema y cada evento tiene su propio **event object**:

| | | |
| -------- | -------- |
| **Nombre** | **Constante KernelEvent** | **Argumento para el listener** |
| kernel.request | KernelEvents::REQUEST | GetResponseEvent |
| kernel.controller | KernelEvents::CONTROLLER | FilterControllerEvent |
| kernel.view | KernelEvents::VIEW | GetResponseForControllerResultEvent |
| kernel.response | KernelEvents::RESPONSE | FilterResponseEvent |
| kernel.finish_request | KernelEvents::FINISH_REQUEST | FinishRequestEvent |
| kernel.terminate | KernelEvents::TERMINATE | PostResponseEvent |
| kernel.exception | KernelEvents::EXCEPTION | GetResponseForExceptionEvent |

### <a id="EjemploDeHttpKernel" name="EjemploDeHttpKernel"></a>11\. Ejemplo de HttpKernel

Cuando se utiliza el componente **HttpKernel**, eres libre de añadir cualquier listener a los eventos incorporados y usar cualquier **controller resolver** que implemente **ControllerResolverInterface**. Sin embargo, el componente HttpKernel viene con listeners incorporados y un **ControllerResolver** incorporado que pueden utilizarse para crear el siguiente ejemplo:

```
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\RequestStack;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\HttpKernel;
use Symfony\Component\EventDispatcher\EventDispatcher;
use Symfony\Component\HttpKernel\Controller\ControllerResolver;
use Symfony\Component\HttpKernel\EventListener\RouterListener;
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;
use Symfony\Component\Routing\Matcher\UrlMatcher;
use Symfony\Component\Routing\RequestContext;

$routes = new RouteCollection();
$routes->add('hello', new Route('/hello/{name}', array(
        '_controller' => function (Request $request) {
            return new Response(
                sprintf("Hello %s", $request->get('name'))
            );
        }
    )
));

$request = Request::createFromGlobals();

$matcher = new UrlMatcher($routes, new RequestContext());

$dispatcher = new EventDispatcher();
$dispatcher->addSubscriber(new RouterListener($matcher, new RequestStack()));

$resolver = new ControllerResolver();
$kernel = new HttpKernel($dispatcher, $resolver);

$response = $kernel->handle($request);
$response->send();

$kernel->terminate($request, $response);
```

### <a id="SubRequests" name="SubRequests"></a>12\. Sub requests

Además del request principal enviado desde _HttpKernel::handle_, puedes también enviar **sub requests**. Un sub request parece y actúa como cualquier otro request, pero normalmente sirve para renderizar sólo una pequeña porción de una página en lugar de la página completa. Se usan frecuentemente desde el controller (o a veces dentro de una template renderizada desde un controller).

![Sub requests en Symfony](http://symfony.com/doc/current/_images/sub-request.png)

Para ejecutar un sub request, usa _HttpKernel::handle_, pero cambia el segundo argumento de la siguiente forma:

```
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpKernel\HttpKernelInterface;

// ...

// create some other request manually as needed
$request = new Request();
// for example, possibly set its _controller manually
$request->attributes->set('_controller', '...');

$response = $kernel->handle($request, HttpKernelInterface::SUB_REQUEST);
// do something with this response
```

Esto crea otro ciclo completo request-response donde el nuevo **Request** se transforma en **Response**. La única diferencia internamente es que algunos listeners (como de seguridad) puede que sólo actúen en el request principal. A cada listener se le pasa alguna subclase de KernelEvent, cuyo método isMasterRequest() puede usarse para comprobar si el request actual es "master" o "sub request".

Por ejemplo, un listener que sólo necesita actuar sobre el master request:

```
use Symfony\Component\HttpKernel\Event\GetResponseEvent;
// ...

public function onKernelRequest(GetResponseEvent $event)
{
    if (!$event->isMasterRequest()) {
        return;
    }

    // ...
}
```