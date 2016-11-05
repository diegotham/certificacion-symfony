La clase **Application** el componente Console permite entrar en el _lifecycle_ de la aplicación de consola a través de **eventos**. Para ello utiliza el componente **EventDispatcher**:

```
use Symfony\Component\Console\Application;
use Symfony\Component\EventDispatcher\EventDispatcher;

$dispatcher = new EventDispatcher();

$application = new Application();
$application->setDispatcher($dispatcher);
$application->run();
```

Los eventos de consola sólo son lanzados por el comando principal que se ejecute. Los comandos llamados por el comando principal no lanzan ningún eventos.

### El evento ConsoleEvents::COMMAND

**Uso típico**: Hacer algo antes de que se ejecute cualquier comando (como logear qué comando va a ejecutarse), o mostrar algo sobre el evento que va a ejecutarse.

Justo antes de ejecutar cualquier comando, se lanza el evento **ConsoleEvents::COMMAND**. Los listeners reciben un evento [ConsoleCommandEvent](http://api.symfony.com/3.0/Symfony/Component/Console/Event/ConsoleCommandEvent.html):

```
use Symfony\Component\Console\Event\ConsoleCommandEvent;
use Symfony\Component\Console\ConsoleEvents;

$dispatcher->addListener(ConsoleEvents::COMMAND, function (ConsoleCommandEvent $event) {
    // get the input instance
    $input = $event->getInput();

    // get the output instance
    $output = $event->getOutput();

    // get the command to be executed
    $command = $event->getCommand();

    // write something about the command
    $output->writeln(sprintf('Antes de arrancar el comando <info>%s</info>', $command->getName()));

    // get the application
    $application = $command->getApplication();
});
```

#### Desactivar los comandos dentro de listeners

Con el método [disableCommand()](http://api.symfony.com/3.0/Symfony/Component/Console/Event/ConsoleCommandEvent.html#method_disableCommand), puedes desactivar un comando dentro de un listener. La aplicación no ejecutará entonces el comando, pero devolverá el código _113_ (definido en **ConsoleCommandEvent::RETURN_CODE_DISABLED**). Este código es uno de los códigos de exit para comandos de consola que se ajustan a los estándares  de C/C++:

```
use Symfony\Component\Console\Event\ConsoleCommandEvent;
use Symfony\Component\Console\ConsoleEvents;

$dispatcher->addListener(ConsoleEvents::COMMAND, function (ConsoleCommandEvent $event) {
    // obtener el comando a ejecutar
    $command = $event->getCommand();

    // ... comprobar si el comando puede ejecutarse

    // desactivar el comando, esto hará que se salte el comando
    // y el código 113 sea devuelto por Application
    $event->disableCommand();

    // es posible activar el comando en un listener posterior
    if (!$event->commandShouldRun()) {
        $event->enableCommand();
    }
});
```

### El evento ConsoleEvents::TERMINATE

**Uso típico**: realizar algunas acciones de limpieza después de que se haya ejecutado el comando. 

Despues de haberse ejecutado el comando, se lanza el evento **ConsoleEvents::TERMINATE**. Puede emplearse para hacer cualquier acción que necesite ejecutarse en todos los comandos o para limpiar lo que ya iniciaste en un listener **ConsoleEvents::COMMAND** (como enviar logs, cerrar una conexión a una base de datos, enviar emails...). Un listener podría también cambiar el código exit.

Los listeners reciben un evento [ConsoleTerminateEvent](http://api.symfony.com/3.0/Symfony/Component/Console/Event/ConsoleTerminateEvent.html): 

```
use Symfony\Component\Console\Event\ConsoleTerminateEvent;
use Symfony\Component\Console\ConsoleEvents;

$dispatcher->addListener(ConsoleEvents::TERMINATE, function (ConsoleTerminateEvent $event) {
    // obtener el output
    $output = $event->getOutput();

    // obtener el comando que se ha ejecutado
    $command = $event->getCommand();

    // mostrar algo
    $output->writeln(sprintf('After running command <info>%s</info>', $command->getName()));

    // cambiar el código exit
    $event->setExitCode(128);
});
```

Este evento también se lanza cuando la aplicación ha lanzado una excepción. Se lanza entonces justo antes del evento **ConsoleEvents::EXCEPTION**. El código exit recibido en este caso es el código de la excepción.

### El evento ConsoleEvents::EXCEPTION

**Uso típico**: Manejar exceptiones lanzadas durante la ejecución de un comando. 

Cuando un comando lanza una excepción, se lanza el evento **ConsoleEvents::EXCEPTION**. Un listener puede envolver o cambiar la excepción o hacer algo útil antes de que la aplicación lance la excepción.

Los listeners reciben un evento [ConsoleExceptionEvent](http://api.symfony.com/3.0/Symfony/Component/Console/Event/ConsoleExceptionEvent.html):

```
use Symfony\Component\Console\Event\ConsoleExceptionEvent;
use Symfony\Component\Console\ConsoleEvents;

$dispatcher->addListener(ConsoleEvents::EXCEPTION, function (ConsoleExceptionEvent $event) {
    $output = $event->getOutput();

    $command = $event->getCommand();

    $output->writeln(sprintf('Oops, excepción lanzada mientras se ejecutaba <info>%s</info>', $command->getName()));

    // obtener el código exit actual (el código de excepción o exit establecidos por el evento ConsoleEvents::TERMINATE)
    $exitCode = $event->getExitCode();

    // cambiar la excepción a otra
    $event->setException(new \LogicException('Excepción capturada', $exitCode, $event->getException()));
});
```