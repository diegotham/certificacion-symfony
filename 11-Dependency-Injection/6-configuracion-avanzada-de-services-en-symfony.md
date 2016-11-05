**Indice de contenido**

1.  Marcar services como public o private
2.  Crear alias
3.  Decorar services
4.  Deprecating services
5.  Utilizar un Service Configurator

### 1\. Marcar services como public o private

Cuando se definen _services_, normalmente querrás acceder a sus definiciones dentro del código de la aplicación. Estos services se denominan _public_. Por ejemplo, el _doctrine service_ registrado en el container con el **DoctrineBundle** es un _public service_. Esto significa que puedes obtenerlo desde el container con el método _get()_:

```
$doctrine = $container->get('doctrine');
```

En algunos casos un service sólo existe para ser inyectado en otro service y no se tiene intención de obtenerlo de forma directa desde el container.

En estos casos, para obtener un pequeño aumento de rendimiento, puedes establecer el service como _not public_:

```
services:
   foo:
     class: Example\Foo
     public: false
```

Lo que hace a los private services especiales es que, si sólo se inyectan una vez, **se convierten de services a instancias alineadas** (poe ejemplo _new Privatething()_). Esto incrementa el rendimiento del container.

Ahora que el service está marcado como private no deberías acceder a él con _get()_. Podría funcionar o no, dependiendo de si el service se haya podido alinear. De todas formas, **un service se marca como private cuando no tienes intención de acceder a él desde el cógigo**.

Un service private también puede aliarse, como se verá a continuación.

### 2\. Crear alias

A veces puede resultar beneficioso utilizar **shorcuts** para acceder a algunos services. Puedes hacerlo creando un alias, ya sea en services _private_ o _public_:

```
services:
   foo:
     class: Example\Foo
   bar:
     alias: foo
```

Esto significa que cuando quieras acceder al container directamente, puedas hacerlo con cualquiera de los dos nombres, _foo_ o _bar_:

```
$container->get('bar'); // Devolverá el service foo
```

En **YAML** también puedes usar un **shortcut** para añadir un alias a un service:

```
services:
   foo:
     class: Example\Foo
   bar: '@foo'
```

### 3\. Decorar services

Cuando se sobreescribe una definición existente, el service antiguo se pierde:

```
$container->register('foo', 'FooService');

// Esto reemplazará la definición antigua con la nueva
// La definición antigua se pierde
$container->register('foo', 'CustomFooService');
```

La mayoría de las veces esto es lo que querrás hacer realmente, pero a veces podrías querer **decorar** el anterior. En este caso el service antiguo debe estar disponible para referenciarlo en el nuevo. Esta configuración reemplaza _foo_ con un nuevo objeto, pero mantiene una referencia al antiguo como _bar.inner_:

```
bar:
  public: false
  class: stdClass
  decorates: foo
  arguments: ["@bar.inner"]
```

El método _setDecoratedService()_ le dice al container que el service _bar_ debería reemplazar el service _foo_, renombrando _foo_ a _bar.inner_. Por convención, el antiguo _foo_ service será renombrado a _bar.inner_, por lo que puedes inyectarlo en el nuevo service.

El _inner id_ generado se basa en el id del **decorator service** (en este caso _bar_), no del **decorated service** (en este caso _foo_). Esto es necesario para permitir varios decorators en el mismo service (han de tener diferentes inner id's).

La mayoría de las veces el decorator debería declararse _private_, ya que no necesitarás obtenerlo como _bar_ desde el container. La visibilidad del _foo_ service decorado (que es un alias para _bar_) será la misma que la visibilidad del original _foo_. 

Podemos cambiar el nombre del inner service si queremos:

```
bar:
  class: stdClass
  public: false
  decorates: foo
  decoration_inner_name: bar.wooz
  arguments: ["@bar.wooz"]
```

Si quieres añadir más de un decorator a un service, puedes controlar su orden configurando la prioridad de decoración mediante un integer (a número más alto mayor prioridad):

```
foo:
    class: Foo

bar:
    class: Bar
    public: false
    decorates: foo
    decoration_priority: 5
    arguments: ['@bar.inner']

baz:
    class: Baz
    public: false
    decorates: foo
    decoration_priority: 1
    arguments: ['@baz.inner']
```

El código generado será como sigue:

```
$this->services['foo'] = new Baz(new Bar(new Foo())));
```

### 4\. Deprecating services

Una vez que has decidido marcar a un service como deprecated (obsoleto), puedes hacerlo así:

```
acme.my_service:
    class: ...
    deprecated: El service "%service_id%" está obsoleto desde la 2.8 y se eliminará en 3.0.
```

Ahora, cada vez que se utilice este service, se lanzará un **deprecation warning**, avisándote para que puedas cambiar el uso de ese service.

El mensaje es un _message template_, que reemplaza ocurrencias del placeholder _%service_id%_ con el id del service. Debes tener al menos una ocurrencia del placeholder _%service_id%_ en la template.

El mensaje de deprecation es opcional. Si no se establece, Symfony mostrará uno por defecto: _The "%service_id%" service is deprecated. You should stop using it, as it will soon be removed..._

Se recomienda totalmente establecer un **mensaje personalizado** porque el de por defecto es muy genérico. Un buen mensaje informa de cuando el service se estableció como deprecated, hasta cuando se mantendrá y services alternativos a usar, si exiten.

Para **service decorators**, si la definición no modifica el estado deprecated, heredará el estado de la definición que está decorada.

### 5\. Utilizar un Service Configurator

El **Service Configurator** es una funcionalidad del **Dependency Injection Container** que permite usar un callable para configurar un service después de su instanciación.

Puedes especificar un método en otro service, una **función PHP** o un método estático en una clase. La instancia del service se pasa al callable, permitiendo al configurator hacer lo que necesite para configurar el service después de su creación.

Un **Service Configurator** puede usarse, por ejemplo, cuando tienes un _service_ que requiere una instalación compleja basada en ajustes de configuración desde diferentes fuentes o services. Con un configurator externo, puedes mantener la implementación del service limpia y mantenerlo desacoplado de los otros objetos que proporcionan la configuración necesaria.

Otro ejemplo de uso es cuando tienes múltiples objetos que comparten una configuración común o que deberían configurarse de forma parecida en el inicio.

Por ejemplo, tenemos una aplicación donde envías **diferentes tipos de email a los usuarios**. Los emails se pasan a través de diferentes _formatters_ que pueden activarse o no dependiendo de algunos ajustes dinámicos de la aplicación. Empezamos definiendo la clase **NewsletterManager**:

```
class NewsletterManager implements EmailFormatterAwareInterface
{
    protected $mailer;
    protected $enabledFormatters;

    public function setMailer(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function setEnabledFormatters(array $enabledFormatters)
    {
        $this->enabledFormatters = $enabledFormatters;
    }

    // ...
}
```

y una clase **GreetingCardManager**:

```
class GreetingCardManager implements EmailFormatterAwareInterface
{
    protected $mailer;
    protected $enabledFormatters;

    public function setMailer(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function setEnabledFormatters(array $enabledFormatters)
    {
        $this->enabledFormatters = $enabledFormatters;
    }

    // ...
}
```

El objetivo es establecer los formatters en tiempo de ejecución dependiendo de los **ajustes de la aplicación**. Para hacerlo, también tienes una clase **EmailFormatterManager** responsable de cargar y validar formatters activados en la aplicación:

```
class EmailFormatterManager
{
    protected $enabledFormatters;

    public function loadFormatters()
    {
        // código que configura qué formatters usar
        $enabledFormatters = array(...);
        // ...

        $this->enabledFormatters = $enabledFormatters;
    }

    public function getEnabledFormatters()
    {
        return $this->enabledFormatters;
    }

    // ...
}
```

Si el objetivo es evitar tener que juntar **NewsletterManager** y **GreetingCardManager** con **EmailFormatterManager**, podemos crear una clase configurator para configurar estas instancias:

```
class EmailConfigurator
{
    private $formatterManager;

    public function __construct(EmailFormatterManager $formatterManager)
    {
        $this->formatterManager = $formatterManager;
    }

    public function configure(EmailFormatterAwareInterface $emailManager)
    {
        $emailManager->setEnabledFormatters(
            $this->formatterManager->getEnabledFormatters()
        );
    }

    // ...
}
```

El trabajo de **EmailConfigurator** es inyectar los formatters activados en **NewsletterManager** y **GreetingCardManager** porque no saben de dónde vienen los formatters. Por otra parte, el **EmailFormatterManager** sabe acerca de los formatters activados y cómo cargarlos, manteniendo el principio de _single responsibility_. 

La configuración de services para las clases anteriores quedaría algo así:

```
services:
    my_mailer:
        # ...

    email_formatter_manager:
        class:     EmailFormatterManager
        # ...

    email_configurator:
        class:     EmailConfigurator
        arguments: ['@email_formatter_manager']
        # ...

    newsletter_manager:
        class:     NewsletterManager
        calls:
            - [setMailer, ['@my_mailer']]
        configurator: ['@email_configurator', configure]

    greeting_card_manager:
        class:     GreetingCardManager
        calls:
            - [setMailer, ['@my_mailer']]
        configurator: ['@email_configurator', configure]
```