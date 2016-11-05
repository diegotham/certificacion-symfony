De la misma forma que se añaden etiquetas (_tags_) en una entrada de un blog, los **services del container** también se pueden etiquetar. En el service container una tag implica que el service se usará para algo concreto:

```
# app/config/services.yml
services:
    foo.twig.extension:
        class: AppBundle\Extension\FooExtension
        public: false
        tags:
            -  { name: twig.extension }
```

La etiqueta _twig.extension_ la utiliza **TwigBundle** durante la configuración. Proporcionando en el service esta etiqueta, el bundle sabe que el service _foo.twig.extension_ debería ser una extensión de Twig. Twig localiza todos los services con la etiqueta twig.extension y automáticamente los registra como extensiones. Puedes ver el listado completo de tags incorporados en Symfony [aquí](http://symfony.com/doc/current/reference/dic_tags.html).

Los tags por sí mismos **no alteran la funcionalidad de los services**. Pero si quieres, puedes pedir a un **container builder** una lista de todos los services etiquetados con una etiqueta específica. Esto es útil en **compiler passes** donde puedes encontrar estos services y usarlos o modificarlos de alguna forma.

Por ejemplo, si usamos **Swift Mailer** podríamos implementar una etiqueta "_transport chain_", que es una colección de clases implementando **\Swift_Transport**. Con ello querrías que Swift Mailer intentara varias formas de transportar el mensaje hasta que uno funcione.

Primero podemos definir la clase **TransportChain**:

```
class TransportChain
{
    private $transports;

    public function __construct()
    {
        $this->transports = array();
    }

    public function addTransport(\Swift_Transport $transport)
    {
        $this->transports[] = $transport;
    }
}
```

Entonces lo definimos como service:

```
services:
    acme_mailer.transport_chain:
        class: TransportChain
```

### Definir services con un tag personalizado

Si ahora queremos que varias clases **\Swift_transport** se instancien y añadan al **TransportChain** automáticamente con el método _addTransport_:

```
services:
    acme_mailer.transport.smtp:
        class: \Swift_SmtpTransport
        arguments:
            - '%mailer_host%'
        tags:
            -  { name: acme_mailer.transport }
    acme_mailer.transport.sendmail:
        class: \Swift_SendmailTransport
        tags:
            -  { name: acme_mailer.transport }
```

Observa que a cada uno se le asigna una etiqueta _acme_mailer.transport_. Esta es la etiqueta personalizada que se empleará en el **compiler pass**. El compiler pass es el que hace que la etiqueta signifique algo.

### Crear un CompilerPass

El **compiler pass** puede ahora solicitar al container _services_ con la etiqueta personalizada:

```
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\Reference;

class TransportCompilerPass implements CompilerPassInterface
{
    public function process(ContainerBuilder $container)
    {
        if (!$container->has('acme_mailer.transport_chain')) {
            return;
        }

        $definition = $container->findDefinition(
            'acme_mailer.transport_chain'
        );

        $taggedServices = $container->findTaggedServiceIds(
            'acme_mailer.transport'
        );
        foreach ($taggedServices as $id => $tags) {
            $definition->addMethodCall(
                'addTransport',
                array(new Reference($id))
            );
        }
    }
}
```

El método _process()_ comprueba la existencia del service _acme_mailer.transport_chain_, después busca a todos los services etiquetados con _acme_mailer.transport_. Añade a la definición del service _acme_mailer.transport_chain_ una llamada a _addTransport_ para cada service _acme_mailer.transport_ que encuentre. El primer argumento de cada una de estas llamadas será el mismo service _mailer transport_.

### Registrar el Pass con el Container

También tenemos que registrar el **pass** con el **container**, y entonces se ejecutará cuando el container se compile. En un bundle, por ejemplo **AcmeMailerBundle**:

```
// src/Acme/MailerBundle/AcmeMailerBundle.php
namespace Acme\MailerBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;
use Symfony\Component\DependencyInjection\ContainerBuilder;

use Acme\MailerBundle\DependencyInjection\Compiler\TransportCompilerPass;

class AcmeMailerBundle extends Bundle
{
    public function build(ContainerBuilder $container)
    {
        parent::build($container);

        $container->addCompilerPass(new CustomCompilerPass());
    }
}
```

### Añadir atributos adicionales en Tags

A veces podemos necesitar **información adicional sobre cada service** que está etiquetado en un **tag**. Por ejemplo, podríamos querer añadie un alias a cada miembro de **TransportChain**. Primero cambiamos la clase:

```
class TransportChain
{
    private $transports;

    public function __construct()
    {
        $this->transports = array();
    }

    public function addTransport(\Swift_Transport $transport, $alias)
    {
        $this->transports[$alias] = $transport;
    }

    public function getTransport($alias)
    {
        if (array_key_exists($alias, $this->transports)) {
            return $this->transports[$alias];
        }
    }
}
```

Ahora, cuando se llama a _addTransport_, no sólo toma un objeto **Swift_Transport**, sino también un alias para el transport.

Ahora, para permitir que cada _transport service_ etiquetado porporciona un alias, cambiamos la declaración del _service_:

```
services:
    acme_mailer.transport.smtp:
        class: \Swift_SmtpTransport
        arguments:
            - '%mailer_host%'
        tags:
            -  { name: acme_mailer.transport, alias: foo }
    acme_mailer.transport.sendmail:
        class: \Swift_SendmailTransport
        tags:
            -  { name: acme_mailer.transport, alias: bar }
```

Hemos añadido un key _alias_ a la etiqueta. Para usarlo, actualizamos el compiler:

```
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\Reference;

class TransportCompilerPass implements CompilerPassInterface
{
    public function process(ContainerBuilder $container)
    {
        if (!$container->hasDefinition('acme_mailer.transport_chain')) {
            return;
        }

        $definition = $container->getDefinition(
            'acme_mailer.transport_chain'
        );

        $taggedServices = $container->findTaggedServiceIds(
            'acme_mailer.transport'
        );
        foreach ($taggedServices as $id => $tags) {
            foreach ($tags as $attributes) {
                $definition->addMethodCall(
                    'addTransport',
                    array(new Reference($id), $attributes["alias"])
                );
            }
        }
    }
}
```

El doble loop es debido a que un **service puede tener más de un tag**. Podemos etiquetar un service dos veces o más con _acme_mailer.transport_. El segundo foreach loop itera sobre las etiquetas _acme_mailer.transport_ establecidas en el service actual y te devuelve los atributos.