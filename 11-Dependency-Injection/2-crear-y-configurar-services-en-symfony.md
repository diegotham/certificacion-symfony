Para **crear y configurar services en Symfony** sólo hay que registrarlos en un archivo de configuración.

Si tenemos un objeto **Mailer**, le decimos al _container_ cómo crear el Mailer _service_ con la siguiente configuración:

```
# app/config/services.yml
services:
    app.mailer:
        class:        AppBundle\Mailer
        arguments:    [sendmail]
```

Cuando **Symfony** arranca, crea el _service container_ empleando la configuración de la aplicación (por defecto _app/config/config.yml_).

Una instancia de la clase _AppBundle\Mailer_ estará disponible ahora a través del service container. El container está disponible en cualquier controller de Symfony donde puedas acceder a los services del container a través del método _get()_:

```
class HelloController extends Controller
{
    // ...

    public function sendEmailAction()
    {
        // ...
        $mailer = $this->get('app.mailer');
        $mailer->send('ryan@foobar.net', ...);
    }
}
```

Este controller tiene acceso al service container con _get()_ porque extiende el [controller base de Symfony](http://diego.com.es/la-clase-base-controller-de-symfony).

Cuando solicitas el service _app.mailer_ al container, éste construye el objeto y lo devuelve. Un service núnca se construye hasta que se le llama. Si defines un service pero nunca lo llamas en el request, el service nunca se crea. Esto ahorra memoria e incrementa la velocidad de la aplicación, ya que no se usa memoria simplemente por nombrar services en el archivo de configuración.

Además, el service **Mailer** sólo se crea una vez y la misma instancia se devuelve cada vez que solicitas el service. Este será casi siempre el comportamiento que necesitas (es más potente y flexible), aunque también puedes crear services con diferentes instancias.

### Service Parameters

La **creación de nuevos services** a través del container es muy directa. Los parámetros hace que que definir servicios sea más organizado y flexible:

```
# app/config/services.yml
parameters:
    app.mailer.transport:  sendmail

services:
    app.mailer:
        class:        AppBundle\Mailer
        arguments:    ['%app.mailer.transport%']
```

El resultado es el mismo que antes. La diferencia es está en cómo se define el service. Con los símbolos de porcentaje % el container sabe que tiene que buscar un parámetro con ese nombre. Cuando se construye el container, busca el valor en cada parámetro y lo usa en la definición del service.

Si quieres usar un **símbolo de porcentaje % o de arroba @** en la definición de argumentos, simplemente tienes que duplicarlo, por ejemplo:

```
# app/config/parameters.yml
parameters:
    # Esto se traducirá como un string '@securepass'
    mailer_password: '@@securepass'
```

El objetivo de los parámetros es proporcionar información en los services. Se puede definir el service sin parámetros, pero hacerlo tiene algunas ventajas:

*   **Separación y organización** de las opciones del service bajo la key _parameters_.
*   Los valores del parámetro pueden **usarse en varias definiciones de services**.
*   Cuando se crea un service en un bundle, usar parámetros permite **personalizar el service fácilmente en la aplicación**.

Los bundles de terceros de calidad siempre usan parámetros ya que hacen al service guardado en el container más configurable. Para services en tu aplicación, puede que no necesites esta flexibilidad de parámetros.

### Importar otras fuentes de configuración

Los **archivos de configuración de los services** se nombran como resources, ya que además de poder cargar archivos YAML, XML o PHP, Symfony es tan flexible que la configuración puede cargarse desde cualquier parte (desde una base de datos o un web service, por ejemplo).

El service container está construido con sólo un resource (por defecto _app/config/config.yml_). Toda la demás configuración de services (incluyendo el core de Symfony y configuración de bundles de terceros) debe importarse desde este archivo. Esto te da completa flexibilidad para el manejo de los services en la aplicación.

#### Importar configuración con imports

Hasta ahora hemos puesto la definición del service _app.mailer_ directamente en el archivo de configuración de la aplicación. Ya que la clase **Mailer** se encuentra dentro de un bundle, tiene más sentido poner la definición de _app.mailer_ dentro del bundle también.

Primero, movemos la definición a un nuevo archivo resource dentro del bundle **AcmeHelloBundle**. Si no existe el directorio _Resources/config_, se crea:

```
# src/Acme/HelloBundle/Resources/config/services.yml
parameters:
    app.mailer.transport: sendmail

services:
    app.mailer:
        class:        AppBundle\Mailer
        arguments:    ['%app.mailer.transport%']
```

La definición no ha cambiado, sólo su localización. El service container todavía desconoce la localización del nuevo archivo resource. Para ello simplemente importamos el archivo con la _key_ _imports_ en la configuración de la aplicación:

```
# app/config/config.yml
imports:
    - { resource: '@AcmeHelloBundle/Resources/config/services.yml'
```

Dado a la forma en la que se resuelven los parámetros, no puedes usarlos para construir paths de forma dinámica. Lo siguiente no funciona:

```
# app/config/config.yml
imports:
    - { resource: '%kernel.root_dir%/parameters.yml' }
```

La directiva imports le permite a tu aplicación incluir los archivos de configuración resources desde cualquier localización (normalmente desde bundles). La localización _resource_, para archivos, es el directorio absoluto al archivo resource. La sintaxis especial @AcmeHelloBundle resuelve el directorio del bundle AcmeHelloBundle. Esto ayuda a especificar el directorio del resource sin preocuparse en si después mueves el bundle a otro directorio.

#### Importar configuración con Container Extensions

Lo más común es emplear _imports_ para **importar configuración del container** de los bundles que hayas creado específicamente para la aplicación. La configuración de bundles de terceros, incluyendo la de Symfony, normalmente se cargan con otro método más flexible y fácil de configurar en la aplicación.

Internamente cada bundle define los services como hemos hecho hasta ahora. Un bundle emplea uno o más archivos resource de configuración (a menudo XML) para especificar los parámetros y services para ese bundle. Sin embargo, en lugar de importar cada uno de los resources directamente con imports, puedes simplemente invocar un _service container extension_ dentro del bundle para que lo haga automáticamente. Esto es una clase PHP creada por el autor del bundle que consigue dos cosas:

*   Importar todos los resources del service container para configurar los services para el bundle.
*   Proporcionar configuración semántica y directa para que el bundle pueda configurarse sin interactuar con los parámetros de la configuración del service container del bundle.

En otras palabras, una **extensión service container** configura los services para un bundle por tí. El siguiente código en la configuración de tu aplicación invoca la service container extension dentro del FrameworkBundle:

```
# app/config/config.yml
framework:
    secret:          xxxxxxxxxx
    form:            true
    csrf_protection: true
    router:        { resource: '%kernel.root_dir%/config/routing.yml' }
    # ...
```

Cuando **Symfony** analiza la configuración, el container busca una extensión que pueda manejar la directiva de configuración _framework_. Esta extensión en cuestión, que se encuentra en el **FrameworkBundle**, se invoca y la configuración de services del FrameworkBundle se carga. Si eliminas el key _framework_ de la configuración de la aplicación, los services de Symfony no se cargarán. Tú tienes el control: el framework Symfony no hace nada sobre la que no tengas control alguno.

Puedes hacer más que simplemente activar el service container extension para el FrameworkBundle. Cada extensión te permite configurar el bundle, sin preocuparte sobre los services internos que estén definidos. 

En este caso, la extensión permite personalizar las configuraciones de _error_handler_, _csrf_protextion_, _router_, etc. Internamente, el **FrameworkBundle** utiliza las opciones especificadas aquí para definir y configurar services específicos. El bundle se encarga de crear todos los parametros y services para el service container, y a la vez permite personalizarlos fácilmente. Además, la mayoría de extensiones service container pueden realizar también validación (ofreciendo notificaciones sobre opciones no configuradas o que no son del type correcto).

Cuando se instala y configura un bundle, hay que leer la configuración del bundle para ver cómo los services del bundle han de instalarse y configurarse. Las opciones de los bundles incorporados en **Symfony** pueden leerse en la [guía de referencia](http://symfony.com/doc/current/reference/index.html).

Inicialmente, el service container sólo reconoce las directivas _parameters_, _services_ e _imports_. Cualquier otra directiva es por las extensiones service container.

### Inyectar Services

Hasta ahora el service _app.mailer_ original es simple: toma sólo un argumento en su constructor, lo que es fácilmente configurable. El verdadero poder del container es cuando necesitas crear un service que depende de uno o más services del container.

Por ejemplo tenemos un _service_, **NewsletterManager**, que administra la preparación y envío de un mensaje de email a una colección de direcciones. El service _app.mailer_ ya es muy bueno enviando mensajes, por lo que lo usaremos dentro de NewsletterManager para manejar el envío de mensajes. La clase será:

```
// src/AppBundle/Newsletter/NewsletterManager.php
namespace AppBundle\Newsletter;

use AppBundle\Mailer;

class NewsletterManager
{
    protected $mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    // ...
}
```

Sin el _service container_ puedes crear un nuevo **NewsletterManager** desde el controller:

```
use AppBundle\Newsletter\NewsletterManager;

// ...

public function sendNewsletterAction()
{
    $mailer = $this->get('app.mailer');
    $newsletter = new NewsletterManager($mailer);
    // ...
}
```

Este enfoque está bien, pero si por ejemplo queremos añadir más argumentos al constructor de NewsletterManager o refactorizar el código y renombrar la clase, tendremos que encontrar cada sitio donde hemos instanciado NewsletterManager. El service container ofrece una opción mucho más cómoda:

```
# app/config/services.yml
services:
    app.mailer:
        # ...

    app.newsletter_manager:
        class:     AppBundle\Newsletter\NewsletterManager
        arguments: ['@app.mailer']
```

En **YAML**, la sintaxis especial _@app.mailer_ le dice al container que busque un service llamado _app.mailer_ y pasa ese objeto al constructor de NewsletterManager. El service especificado _app.mailer_ debe existir, sino, se lanza una **excepción** (aunque es posible marcar uns dependencia como opcional).

Usar referencias es una herramienta muy potente que permite crear clases _service_ independientes con **dependencias** bien definidas. En ese ejemplo, el service _app.newsletter\_manager_ necesita al service _app.mailer_ para funcionar. En el momento en que defines esta dependencia en el service container, éste se encarga de todo el trabajo de instanciar clases. 

#### Emplear el Expressión Language

El service container también soporta una expresión que permite inyectar valores muy expecíficos en un service.

Por ejemplo, tenemos un tercer service (no se muestra aquí), llamado _mailer_configuration_, que tiene el método _getMailerMethod()_, y devolverá un string _sendmail_ basado en alguna configuración. El primer argumento en el service my_mailer es el string _sendmail_:

```
# app/config/services.yml
services:
    app.mailer:
        class:        AppBundle\Mailer
        arguments:    [sendmail]
```

Pero en lugar de escribirlo directamente, podemos obtener el valor que se extrae de getMailerMehotd() del service mailer_configuration usando una expresión:

```
# app/config/config.yml
services:
    my_mailer:
        class:        Acme\HelloBundle\Mailer
        arguments:    ["@=service('mailer_configuration').getMailerMethod()"]
```

En este contexto tenemos acceso a 2 funciones:

*   _service_. Devuelve un service
*   _parameter_. Devuelve el valor de un parámetro específico. 

También tenemos acceso al **ContainerBuilder** a través de una variable _container_. Aquí hay otro ejemplo:

```
services:
    my_mailer:
        class:     Acme\HelloBundle\Mailer
        arguments: ["@=container.hasParameter('some_param') ? parameter('some_param') : 'default_value'"]
```

#### Dependencias opcionales: setter injection

Si tienes dependencias opcionales para una clase, puedes emplear una _setter injection_, inyectar la dependencia con una llamada a un método en vez de con el constructor. La clase puede ser:

```
namespace AppBundle\Newsletter;

use AppBundle\Mailer;

class NewsletterManager
{
    protected $mailer;

    public function setMailer(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    // ...
}
```

Inyectar la dependencia con el setter sólo necesita un pequeño cambio en la sintaxis:

```
# app/config/services.yml
services:
    app.mailer:
        # ...

    app.newsletter_manager:
        class:     AppBundle\Newsletter\NewsletterManager
        calls:
            - [setMailer, ['@app.mailer']]
```

Se han visto hasta ahora **constructor injection** y **setter injection**, pero también existe **property injection**.

### Inyectar el request

Para inyectar el request hay que inyectar el service _request_stack_ y acceder a **Request** llamando al método _getCurrentRequest_:

```
namespace Acme\HelloBundle\Newsletter;

use Symfony\Component\HttpFoundation\RequestStack;

class NewsletterManager
{
    protected $requestStack;

    public function __construct(RequestStack $requestStack)
    {
        $this->requestStack = $requestStack;
    }

    public function anyMethod()
    {
        $request = $this->requestStack->getCurrentRequest();
        // ... do something with the request
    }

    // ...
}
```

Ahora inyectamos el request_stack, que funciona como cualquier otro service:

```
# src/Acme/HelloBundle/Resources/config/services.yml
services:
    newsletter_manager:
        class:     Acme\HelloBundle\Newsletter\NewsletterManager
        arguments: ["@request_stack"]
```

### Referencias opcionales

A veces alguno de los services pueden tener una dependencia opcional, por lo que no es requerida para que el _service_ funcione correctamente. En el ejemplo anterior, el service _app.mailer_ debe existir, sino se lanzará una excepción. Modificando el service _app.newsletter_manager_ podemos hacer esta referencia opcional. El container lo inyectará si existe o no hará nada si no existe:

```
# app/config/services.yml
services:
    app.newsletter_manager:
        class:     AppBundle\Newsletter\NewsletterManager
        arguments: ['@?app.mailer']
```

En YAML, la sintaxis especial @? le dice al service container que la dependencia es opcional. el NewsletterManager también debe reescribirse para permitir una dependencia opcional:

```
public function __construct(Mailer $mailer = null)
{
    // ...
}
```