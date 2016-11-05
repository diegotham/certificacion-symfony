**Indice de contenido**

1.  Service Auto Wiring
2.  Parent Services

### 1\. Service Auto Wiring

El **componente Dependency Injection** es uno de los elementos más importantes  de las **aplicaciones Symfony**. Este componente permite a los desarrolladores configurar services en archivos **YAML**, **XML**, o **PHP** y dejar a **Symfony** que cree esos services automáticamente.

Los services normalmente tienen una opción _arguments_ que lista los argumentos que se pasan a sus constructores. Si la aplicación contiene las siguientes dos clases:

```
namespace AppBundle\Service;

class Service1
{
}

namespace AppBundle\Service;

use AppBundle\Service\Service1;

class Service2
{
    private $service1;

    public function __construct(Service1 $service1)
    {
        $this->service1 = $service1;
    }
}
```

La configuración en YAML sería como sigue:

```
# app/config/services.yml
services:
    service1:
        class: AppBundle\Service\Service1

    service2:
        class: AppBundle\Service\Service2
        arguments: ['@service1']
```

Con la funcionalidad de service auto wiring podemos saltarnos la definición de service1\. La razón es que el service container puede examinar los parámetros del constructor, crear un private service para la clase Service1 e inyectarlo en Service2.

Esta funcionalidad está desactivada por defecto y su comportamiento está restringido a los casos donde los services pueden adivinarse sin equivocación. Simplemente se establece la opción autowire a true en los services donde quieres activarla y dejar al service container que haga el resto. 

La configuración anterior quedaría así:

```
# app/config/services.yml
services:
    service2:
        class: AppBundle\Service\Service2
        autowire: true
```

### 2\. Parent Services

Cuando vas añadiendo más funcionalidades a una aplicación, puedes empezar a tener clases relacionadas que comparten algunas dependencias. Por ejemplo si tenemos un **Newsletter Manager** que emplea **setter injection** para establecer sus dependencias:

```
class NewsletterManager
{
    protected $mailer;
    protected $emailFormatter;

    public function setMailer(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function setEmailFormatter(EmailFormatter $emailFormatter)
    {
        $this->emailFormatter = $emailFormatter;
    }

    // ...
}
```

y también una clase **Greeting Card** que comparte las mismas dependencias:

```
class GreetingCardManager
{
    protected $mailer;
    protected $emailFormatter;

    public function setMailer(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function setEmailFormatter(EmailFormatter $emailFormatter)
    {
        $this->emailFormatter = $emailFormatter;
    }

    // ...
}
```

La configuración del service para estas clases sería algo así:

```
services:
    my_mailer:
        # ...

    my_email_formatter:
        # ...

    newsletter_manager:
        class: NewsletterManager
        calls:
            - [setMailer, ['@my_mailer']]
            - [setEmailFormatter, ['@my_email_formatter']]

    greeting_card_manager:
        class: 'GreetingCardManager'
        calls:
            - [setMailer, ['@my_mailer']]
            - [setEmailFormatter, ['@my_email_formatter']]
```

Hay mucha repetición en ambas clases y en la configuración. Esto significa que si por ejemplo cambiáramos el **Mailer** a inyectar en el constructor de las clases **EmailFormatter**, tendrías que actualizar la configuración en dos sitios. Igualmente si tenemos que hacer cambios en los métodos settes tendrías que hacerlo en ambas clases. La forma típica de tratar con los métodos comunes de estas clases relacionadas sería extraerlas en una super clase:

```
abstract class MailManager
{
    protected $mailer;
    protected $emailFormatter;

    public function setMailer(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function setEmailFormatter(EmailFormatter $emailFormatter)
    {
        $this->emailFormatter = $emailFormatter;
    }

    // ...
}
```

El **NewsletterManager** y **GreetingCardManager** pueden entonces extender esta super clase:

```
class NewsletterManager extends MailManager
{
    // ...
}
```

`y `

```

class GreetingCardManager extends MailManager
{
    // ...
}
```

El **Symfony service container** también soporta extender services en la configuración de forma que puedas reducir la repetición especificando un **parent para un service**.

```
# ...
services:
    # ...
    mail_manager:
        abstract:  true
        calls:
            - [setMailer, ['@my_mailer']]
            - [setEmailFormatter, ['@my_email_formatter']]

    newsletter_manager:
        class:  "NewsletterManager"
        parent: mail_manager

    greeting_card_manager:
        class:  "GreetingCardManager"
        parent: mail_manager
```

En este contexto, tener un **parent service** implica que los argumentos y las llamadas a métodos del parent service deban usarse para **child services**. Los métodos setter definidos en el parent service serán llamados cuando se instancien los child services.

Si removemos el key _parent_ en la configuración, los services serán instanciados y todavía extenderán a la clase **MailManager**. La diferencia es que omitir la key _parent_ supondrá que las llamadas definidas en el service **mail_manager** no serán ejecutadas cuando se instancien los child services.

Los atributos _abstract_ y _tags_ siempre se obtienen del child service.

El parent service es abstract y no se debería obtener directamente del container o pasarse a otro service. Sirve simplemente como **template** que pueden emplear otros services. Esta es la razón porque no puede tener una key _class_ en la configuración que provocaría una **excepción** de un service **non-abstract**.

Para que se resuelvan las dependencias parent, el **ContainerBuilder** ha de compilarse primero.

En los ejemplos mostrados, las clases que comparten la misma configuración también extienden desde la misma parent class en **PHP**. Esto no es necesario. Puedes simplemente **extraer partes comunes de definiciones de service similares en un parent service sin extender** también la clase parent en PHP.

#### Sobreescribir Parent Dependencies

Puede que haya veces donde quieras **sobreescribir qué dependencia ha de pasarse en sólo un child service**. Añadiendo la configuración _call_ en el child service, las dependencias establecidas en el parent class serán sobreescritas. Así que si quieres pasar diferentes dependencias sólo para la clase **Newsletter**, la configuración quedaría:

```
# ...
services:
    # ...
    my_alternative_mailer:
        # ...

    mail_manager:
        abstract: true
        calls:
            - [setMailer, ['@my_mailer']]
            - [setEmailFormatter, ['@my_email_formatter']]

    newsletter_manager:
        class:  'NewsletterManager'
        parent: mail_manager
        calls:
            - [setMailer, ['@my_alternative_mailer']]

    greeting_card_manager:
        class:  'GreetingCardManager'
        parent: mail_manager
```

**GreetingCardManager** recibirá las mismas dependencias que antes, pero a **NewsletterManager** se le pasará _my_alternative_mailer_ en lugar del service _my_mailer_.

**No puedes sobreescribir llamadas a métodos**. Cuando definimos nuevas llamadas a métodos en el child service, se añaden al conjunto de llamadas a métodos. Esto significa que funciona perfectamente cuando el setter sobreescribe la propiedad actual, pero no funciona cuando el setter se añade a los datos existentes (por ejemplo a un método _addFilters()_). En esos casos, la única solución es no extender el parent service y configurar el service sin esta funcionalidad.