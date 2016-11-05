Durante la ejecución de una aplicación **Symfony**, se lanzan muchas notificaciones de eventos. Tu aplicación puede atender a estas notificaciones y responderlas ejecutando cualquier porción de código.

Eventos internos proporcionados por Symfony se definen en la clase **KernelEvents**. Bundles de terceros y librerías también lanzan muchos eventos y tu propia aplicación puede lanzar eventos personalizados.

Todos los ejemplos mostrados en este artículo utilizan el mismo evento **KernelEvents::EXCEPTION** por temas de consistencia. En tu aplicación puedes usar cualquier evento e incluso mezclar algunos de ellos en el mismo subscriber.

**Indice de contenido**

1.  Crear un Event Listener
2.  Crear un Event Subscriber
3.  Request Events, Checking Types
4.  Listeners or Subscribers
5.  Debugging Event Listeners

### 1. Crear un Event Listener

La forma más común de atender a un evento es **registrar un event listener**:

```
// src/AppBundle/EventListener/ExceptionListener.php
namespace AppBundle\EventListener;

use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Exception\HttpExceptionInterface;

class ExceptionListener
{
    public function onKernelException(GetResponseForExceptionEvent $event)
    {
        // Obtienes el objeto exception del evento recibido
        $exception = $event->getException();
        $message = sprintf(
            'Mi error dice: %s en el código: %s',
            $exception->getMessage(),
            $exception->getCode()
        );

        // Customiza tu response object para mostrar los detalles de excepción
        $response = new Response();
        $response->setContent($message);

        // HttpExceptionInterface es un tipo especial de excepción
        // que mantiene los detalles de header y código de estado
        if($exception instanceof HttpExceptionInterface) {
            $response->setStatusCode($exception->getStatusCode());
            $response->headers->replace($exception->getHeaders());
        } else {
            $response->setStatusCode(Response::HTTP_INTERNAL_SERVER_ERROR);
        }

        // Enviar el objeto Response modificado al evento
        $event->setResponse($response);
    }
}
```

Cada evento recibe un tipo ligeramente diferente de objeto _$event_. Para el evento _kernel.exception_ es **GetResponseForExceptionEvent**. Para ver que tipo de objeto recibe cada event listener, puedes ver la documentación de [KernelEvents](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/KernelEvents.html) o la documentación especial del evento al que estás atendiendo.

Ahora que la clase ya está creada, simplemente necesitas registrarla como _service_ y notificar a **Symfony** que es un _listener_ del evento _kernel.exception_ empleando una etiqueta especial:

```
# app/config/services.yml
services:
    app.exception_listener:
        class: AppBundle\EventListener\ExceptionListener
        tags:
            - { name: kernel.event_listener, event: kernel.exception }
```

Hay una etiqueta opcional llamada **method** que define que método ejecutar cuando se dispara el evento. Por defecto el nombre del método es **on + camelCaseEventName**. Si el evento es **kernel.exception** el método ejecutado por defecto es _onKernelException()_.

La otra etiqueta opcional es **priority**, que por defecto es 0 y controla el orden en el que los listeners son ejecutados (cuanta más prioridad, más pronto se ejecutará un listener). Esto es útil cuando necesitas garantizar que un listener es ejecutado antes que otro. Las prioridades de los listeners internos de Symfony suelen ir de -255 a 255 pero tus propios listeners pueden emplear cualquier integer positivo o negativo.

### 2. Crear un Event Subscriber

Otra forma de atender a eventos es a través de un **event subscriber**, que es una clase que define uno o más métodos que atienden a uno o varios eventos. La principal diferencia con los **event listeners** es que los **subscribers** siempre saben a qué evento están atendiendo.

Dado un subscriber, métodos diferentes pueden atender al mismo evento. El orden en que se ejecutan estos métodos se define por el parámetro priority de cada método (cuanta más prioridad antes se llamará al método). El siguiente ejemplo muestra un **event subcriber** que define varios métodos que atienden al mismo evento **kernel.exception**:

```
// src/AppBundle/EventSubscriber/ExceptionSubscriber.php
namespace AppBundle\EventSubcriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
use Symfony\Component\HttpKernel\Exception\HttpExceptionInterface;

class ExceptionSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        // devuelve los eventos suscritos, sus métodos y prioridades
        return array(
            'kernel.exception' => array(
                array('processException', 10),
                array('logException', 0),
                array('notifyException', -10),
            )
        );
    }

    public function processException(GetResponseForExceptionEvent $event)
    {
        // ...
    }

    public function logException(GetResponseForExceptionEvent $event)
    {
        // ...
    }

    public function notifyException(GetResponseForExceptionEvent $event)
    {
        // ...
    }
}
```

Ahora simplemente tienes que registrar la clase como _service_ y añadir la etiqueta **kernel.event_subcriber** para decirle a **Symfony** que este es un **event subscriber**:

```
# app/config/services.yml
services:
    app.exception_subscriber:
        class: AppBundle\EventSubscriber\ExceptionSubscriber
        tags:
            - { name: kernel.event_subscriber }
```

### 3. Request Events, Checking Types

Una simple página puede hacer varios requests (un **master request**, y después múltiples **subrequests** normalmente [embebiendo controllers](http://symfony.com/doc/current/book/templating.html#templating-embedding-controller)). Para los eventos del núcleo de **Symfony**, podrías necesitar comprobar si el evento es para un master request o para un subrequest.

```
// src/AppBundle/EventListener/RequestListener.php
namespace AppBundle\EventListener;

use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\HttpKernel\HttpKernel;
use Symfony\Component\HttpKernel\HttpKernelInterface;

class RequestListener
{
    public function onKernelRequest(GetResponseEvent $event)
    {
        if(!$event->isMasterRequest()){
            // No hacer nada si no es el master request
            return;
        }
    }
}
```

Ciertas cosas, como comprobar la información en el request real, podrían no necesitar hacerse en los listeners de los **subrequests**.

### 4. Listeners o Subcribers

Los listeners y los subscribers pueden usarse en una aplicación indistintamente. La decisión de usar uno de los dos es normalmente a gusto del desarrollador. Sin embargo, hay algunas ventajas para cada uno:

*   Los **subscribers** son más fáciles de reutilizar porque los eventos se conocen en la clase en lugar de en la definición del service. Esta es la razón por la que Symfony utiliza subscribers internamente.
*   Los **listeners** son más flexibles porque los bundles pueden activarlos o desactivarlos incondicionalmente dependiendo de algún valor de configuración. 

### 5. Debugging Event Listeners

Puedes averiguar que listeners están registrados en el **event dispatcher** utilizando la consola. Para mostrar todos los **eventos** y sus **listeners**:

```
php bin/console debug:event-dispatcher
```

Puedes tener listeners registrados para un evento particular especificando su nombre:

```
php bin/console debug:event-dispatcher kernel.exception
```