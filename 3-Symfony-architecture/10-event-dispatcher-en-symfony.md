El componente **EventDispatcher** proporciona herramientas que permiten a los componentes de tu aplicación comunicarse entre ellos lanzando eventos y recibiéndolos.

El **código orientado a objetos** ha evolucionado para ser extensible. Creando clases que definen correctamente responsabilidades, el código es más flexible y un desarollador puede extenderlo con subclases para modificar su comportamiento. Pero si se quiere compartir los cambios con otros desarrolladores que también han construído sus propias subclases, la **herencia** no es la solución.

Si por ejemplo quieres proporcionar un **sistema de plugins** en un proyecto, cada plugin debería poder añadir métodos o hacer algo antes o después de que un método se ejecute, sin interferir con otros plugins. Este problema no es fácil de resolver con **herencia simple o múltiple**.

El componente EventDispatcher implementa el patrón [Mediator](https://en.wikipedia.org/wiki/Mediator_pattern) de forma simple y efectiva para hacer todo esto posible y para permitir la creación de proyectos extensibles. 

Un ejemplo es el componente **HttpKernel**. Una vez que se ha creado un objeto **Response**, puede resultar útil permitir a otros elementos en el sistema modificarlo (por ejemplo, añadir **cache headers**) antes de que se use. Para hacerlo posible, el kernel de **Symfony** lanza un evento _kernel.response_. Funciona así:

*   Un **listener** (un objeto PHP) le dice al objeto _dispatcher_ que quiere recibir al evento _kernel.response_.
*   En un punto del proceso, el kernel le dice al objeto _dispatcher_ que lance el evento _kernel.response_, pasándolo junto a un objeto **Event** que tiene acceso al objeto **Response**.
*   El _dispatcher_ notifica (llama a un método) a todos los listeners del evento _kernel.response_, permitiendo a cada uno de ellos hacer modificaciones del objeto Response.

**Indice de contenido**

| | |
| -------- | -------- |
| 1\. Eventos | 6\. Paralizar la propagación de eventos |
| 2\. El dispatcher | 7\. Event Dispatcher Aware Events y Listeners |
| 3\. Conectar listeners | 8\. Dispatcher Shortcuts |
| 4\. Crear y lanzar eventos | 9\. Introspección de nombres de eventos |
| 5\. User Event Subscribers | |

### 1\. Eventos

Cuando se lanza un evento se identifica con un nombre único (por ejemplo, _kernel.response_),  y cualquier número de listeners pueden recibirlo. Una instancia de la clase **Event** también se crea y se pasa a todos los listeners. El objeto **Event** contiene normalmente datos sobre el evento lanzado. 

El nombre del evento único puede ser cualquier string, pero opcionalmente sigue las siguientes convenciones:

*   Usar sólo letras minúsculas, números, puntos (.) y barras bajas (_)
*   Prefijar los nombres con el namespace seguido de un punto (por ejemplo: _kernel._)
*   Terminar los nombres con un verbo que indica la acción que se llevará a cabo (por ejemplo: _request_)

Algunos nombres: _kernel.response_, _form.pre_set_data_

Cuando el _dispatcher_ notifica listeners, pasa un objeto **Event** a esos listeners. La clase base Event es muy simple: contiene un método para parar la propagación de eventos, pero no mucho más.

A menudo, datos acerca de un evento específico han de pasarse junto con el objeto **Event** de forma que los listeners tengan la información necesaria. En el caso de un evento _kernel.response_, el objeto Event que se ha creado y pasado a cada listener es realmente del tipo **FilterResponseEvent**, una subclase del objeto base Event. Esta clase contiene métodos como getResponse y setResponse, permitiendo a los listeners obtener o incluso reemplazar el objeto Response.

Cuando se crea un listener para un evento, el objeto Event que se ha pasado al listener puede ser una subclase especial que tiene métodos adicionales para obtener información del evento.

### 2\. El dispatcher

El **dispatcher** es el objeto central del sistema de lanzamiento de eventos. En general se crea sólo un sólo dispatcher, que mantiene un registro de listeners. Cuando se lanza un evento a través del dispatcher, notifica a todos los listeners registrados con ese evento:

```
use Symfony\Component\EventDispatcher\EventDispatcher;

$dispatcher = new EventDispatcher();
```

### 3\. Conectar listeners

Para aprovechar un evento existente hay que conectar un listener al dispatcher de forma que puede ser notificado cuando el evento se lanza. Una llamada al método _addListener()_ asocia cualquier **callable de PHP** a un evento:

```
$listener = new AcmeListener();
$dispatcher->addListener('foo.action', array($listener, 'onFooAction'));
```

El método _addListener()_ acepta hasta tres argumentos:

1.  El nombre del evento (string) que este listener quiere recibir.
2.  Un callable PHP que será notificado cuando el evento se lance.
3.  Un integer opcional, que representa la prioridad del listener frente a otros listeners (por defecto es 0), cuanto más alto antes se disparará. Si dos listeners tienen la misma prioridad, se ejecutan en el orden en que fueron añadidos al dispatcher.

Se pueden registrar como listeners de eventos tanto **objetos PHP** como **Closures**:

```
use Symfony\Component\EventDispatcher\Event;
$dispatcher->addListener('foo.action', function (Event $event) {
    // Se ejecutará cuando el evento foo.action se lance
});
```

Una vez que el listener es registrado con el dispatcher, espera hasta que el evento es notificado. En el ejemplo anterior, cuando el evento _foo.action_ es lanzado, el dispatcher llama al método **AcmeListener::onFooAction** y pasa el objeto **Event** como argumento:

```
use Symfony\Component\EventDispatcher\Event;

class AcmeListener
{
    // ...

    public function onFooAction(Event $event)
    {
        // ... hacer algo
    }
}
```

En muchos casos una **subclase especial Event** específica para el evento se pasa al listener. Esto le da al listener acceso a información especial sobre el evento. Puedes ver la documentación o implementación de cada evento para determinar la instancia exacta de _Symfony\Component\EventDispatcher\Event_ que se pasa. Por ejemplo, el evento _kernel.response_ pasa una instancia de _Symfony\Component\HttpKernel\Event\FilterResponseEvent_:

```
use Symfony\Component\HttpKernel\Event\FilterResponseEvent;

public function onKernelResponse(FilterResponseEvent $event)
{
    $response = $event->getResponse();
    $request = $event->getRequest();

    // ...
}
```

### 4\. Crear y lanzar eventos

Además de registrar listeners para eventos ya existentes, podemos crear y lanzar nuestros propios eventos. Esto es útil para crear librerías de terceros y para mantener componentes diferentes en el sistema de forma flexible y desacoplada.

#### La clase estática para eventos

Suponemos que queremos crear un nuevo **Event** llamado _store.order_ que es lanzado cada vez que se crea un pedido dentro de la aplicación. Para mantener el orden, creamos una clase StoreEvents dentro de la aplicación que sirve para definir y documentar eventos:

```
namespace Acme\StoreBundle;
final class StoreEvents
{
    /**
     * El evento store.order se lanza cada vez que se crea un pedido
     * en el sistema.
     *
     * El event listener recibirá una instancia de
     * Acme\StoreBundle\Event\FilterOrderEvent.
     *
     * @var string
     */
    const STORE_ORDER = 'store.order';
}
```

Nótese que esta clase realmente no hace nada. El objetivo de **StoreEvents** es servir para que la información sobre eventos comunes pueda centralizarse. Nótese también que una clase especial **FilterOrderEvent** se pasará a cada listener de este evento.

#### Crear un objeto Event

Después, cuando se lance el evento, se creará una instancia de Event y se pasará al dispatcher. El dispatcher entonces pasará esta misma instancia a cada uno de los listeners del evento. Si no se necesita pasar ninguna información a los listeners, se puede utilizar la clase por defecto _Symfony\Component\EventDispatcher\Event_. De todas formas la mayoría del tiempo se necesita pasar información sobre el evento a cada listener. Para conseguirlo, tendremos que crear una nueva clase que extienda a _Symfony\Component\EventDispatcher\Event_.

En este ejemplo cada listener necesitará acceso a algún objeto **Order**. Creamos una clase **Event** para hacerlo posible:

```
namespace Acme\StoreBundle\Event;

use Symfony\Component\EventDispatcher\Event;
use Acme\StoreBundle\Order;

class FilterOrderEvent extends Event
{
    protected $order;

    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    public function getOrder()
    {
        return $this->order;
    }
}
```

Ahora cada listener tiene acceso al objeto **Order** a través del método **getOrder**.

#### Lanzar el Event

El método _dispatch()_ notifica a todos los listeners de un evento dado. Requiere dos argumentos: el nombre del evento a lanzar y la instancia Event para pasar cada a cada listener del evento:

```
use Acme\StoreBundle\StoreEvents;
use Acme\StoreBundle\Order;
use Acme\StoreBundle\Event\FilterOrderEvent;
​
// El pedido se obtiene o se crea de alguna forma
$order = new Order();
// ...

// Creamos el evento FilterOrderEvent y lo lanzamos
$event = new FilterOrderEvent($order);
$dispatcher->dispatch(StoreEvents::STORE_ORDER, $event);
```

El objeto **FilterOrderEvent** se crea y pasa al método _dispatch_. Ahora cada listener del evento _store.order_ recibirá el evento FilterOrderEvent y tendrá acceso al objeto **Order** a través del método _getOrder_:

```
// Alguna clase listener que se ha registrado para el evento "store.order"
use Acme\StoreBundle\Event\FilterOrderEvent;

public function onStoreOrder(FilterOrderEvent $event)
{
    $order = $event->getOrder();
    // ...Hacer algo con el pedido
}
```

### 5\. Usar Event Subscribers

La forma más común de recibir un evento es registrar un listener con el dispatcher. Este listener puede recibir uno o más eventos y es notificado cada vez que esos eventos son lanzados.

Otra forma de recibir eventos es a través de un **event subscriber**. Un event subscriber es una clase PHP que puede decirle al dispatcher exactamente a qué eventos debería subscribirse. Implementa la interface EventSubscriberInterface, que requiere un método estático llamado getSubscribedEvents. Veamos un ejemplo de un subscriber que subscribe los eventos kernel.response y store.order:

```
namespace Acme\StoreBundle\Event;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\FilterResponseEvent;

class StoreSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        return array(
            'kernel.response' => array(
                array('onKernelResponsePre', 10),
                array('onKernelResponseMid', 5),
                array('onKernelResponsePost', 0),
            ),
            'store.order'     => array('onStoreOrder', 0),
        );
    }
    public function onKernelResponsePre(FilterResponseEvent $event)
    {
        // ...
    }

    public function onKernelResponseMid(FilterResponseEvent $event)
    {
        // ...
    }

    public function onKernelResponsePost(FilterResponseEvent $event)
    {
        // ...
    }

    public function onStoreOrder(FilterOrderEvent $event)
    {
        // ...
    }
}
```

Es muy similar a una clase listener, excepto que la misma clase puede decirle al dispatcher qué eventos debería recibir. Para registrar un subscriber con el dispatcher se utiliza el método _addSubscriber()_:

```
use Acme\StoreBundle\Event\StoreSubscriber;

$subscriber = new StoreSubscriber();
$dispatcher->addSubscriber($subscriber);
```

El dispatcher registrará automáticamente el subscriber para cada evento devuelto por el método _getSubscribedEvents_. Este método devuelve un array indexado por nombres de eventos cuyos valores son el nombre del método al que llamar o un array compuesto por el nombre del método al que llamar y una prioridad. El ejemplo anterior muestra como registrar varios métodos listener para el mismo evento en el subscriber y también muestra como pasar la prioridad de cada método listener. Cuanto más alta sea la prioridad, antes se llamará al método. En el ejemplo anterior, cuando el evento _kernel.response_ es lanzado, los métodos **onKernelResponse**, **onKernelResponseMid** y **onKernelResponsePost** son llamados en ese orden.

### 6\. Paralizar la propagación de eventos

En algunos casos puede ser necesario para un listener evitar que otros listeners sean llamados. Es decir, el listener necesita decirle al dispatcher que pare la propagación del evento a listeners futuros (por ejemplo, no notificar a más listeners). Esto se puede conseguir desde un listener con el método _stopPropagation()_:

```
use Acme\StoreBundle\Event\FilterOrderEvent;

public function onStoreOrder(FilterOrderEvent $event)
{
    // ...

    $event->stopPropagation();
}
```

Ahora cualquier listener en _store.order_ que no haya sido llamado todavía no será llamado.

Es posible detectar si un evento fue paralizado mediante el método _isPropagationStopped()_ que devuelve un booleano:

```
$dispatcher->dispatch('foo.event', $event);
if ($event->isPropagationStopped()) {
    // ...
}
```

### 7\. EventDispatcher Aware Events y Listeners

El **EventDispatcher** siempre pasa el evento lanzado, el nombre del evento y una referencia de sí mismo a los listeners. Esto hace que se pueda utilizar de forma avanzada el EventDispatcher para **lanzar otros eventos y listeners**, **enlazar eventos** o **lazy loading de más listeners** en el objeto dispatcher como se muestra en los siguientes ejemplos:

**Lazy loading listeners**:

```
use Symfony\Component\EventDispatcher\Event;
use Symfony\Component\EventDispatcher\EventDispatcherInterface;
use Acme\StoreBundle\Event\StoreSubscriber;

class Foo
{
    private $started = false;

    public function myLazyListener(
        Event $event,
        $eventName,
        EventDispatcherInterface $dispatcher
    ) {
        if (false === $this->started) {
            $subscriber = new StoreSubscriber();
            $dispatcher->addSubscriber($subscriber);
        }

        $this->started = true;

        // ... more code
    }
}
```

**Lanzar otro evento desde un listener**:

```
use Symfony\Component\EventDispatcher\Event;
use Symfony\Component\EventDispatcher\EventDispatcherInterface;

class Foo
{
    public function myFooListener(
        Event $event,
        $eventName,
        EventDispatcherInterface $dispatcher
    ) {
        $dispatcher->dispatch('log', $event);

        // ... more code
    }
}
```

Los anteriores suelen bastar para la mayoría de casos, pero si tu aplización utiliza múltiples instancias de EventDispatcher, podrías necesitar inyectar específicamente una instancia concreta del EventDispatcher en tus listeners. Esto puede hacerse mediante un constructor o con setter injection:

**Constructor injection**:

```
use Symfony\Component\EventDispatcher\EventDispatcherInterface;

class Foo
{
    protected $dispatcher = null;

    public function __construct(EventDispatcherInterface $dispatcher)
    {
        $this->dispatcher = $dispatcher;
    }
}
```

**Setter injection**:

```
use Symfony\Component\EventDispatcher\EventDispatcherInterface;

class Foo
{
    protected $dispatcher = null;

    public function setEventDispatcher(EventDispatcherInterface $dispatcher)
    {
        $this->dispatcher = $dispatcher;
    }
}
```

Elegir entre cualquiera de los dos depende de las preferencias de cada desarrollador. Con el **constructor injection** los objetos se inician automáticamente al iniciar la construcción, pero si hay una gran lista de dependencias, utilizar **setter injection** puede ser más conveniente, sobre todo cuando hay dependencias opcionales.

### 8\. Dispatcher Shortcuts

El método EventDispatcher::dispatch siempre devuelte un objeto Event. Esto posibilita varios shortcuts. Por ejemplo, si no se necesita un objeto event personalizado, se puede utilizar el objeto Event vacío. Ni siquiera se necesita pasarlo al dispatcher ya que creará uno por defecto a no ser que específicamente lo pases:

```
$dispatcher->dispatch('foo.event');
```

El event dispatcher siempre devuelte el objeto **Event** que fue lanzado, ya sea el evento que se haya pasado o el evento creado internamente por el dispatcher. Esto permite varios shortcuts:

```
if (!$dispatcher->dispatch('foo.event')->isPropagationStopped()) {
    // ...
}
```

o

```
$barEvent = new BarEvent();
$bar = $dispatcher->dispatch('bar.event', $barEvent)->getBar();
```

o

```
$bar = $dispatcher->dispatch('bar.event', new BarEvent())->getBar();
```

etc.

### 9\. Introspección de nombres de eventos

La instancia de EventDispatcher, así como el nombre del evento que es lanzado, se pasan como argumentos al listener:

```
use Symfony\Component\EventDispatcher\Event;
use Symfony\Component\EventDispatcher\EventDispatcherInterface;

class Foo
{
    public function myEventListener(Event $event, $eventName, EventDispatcherInterface $dispatcher)
    {
        // ... do something with the event name
    }
}
```