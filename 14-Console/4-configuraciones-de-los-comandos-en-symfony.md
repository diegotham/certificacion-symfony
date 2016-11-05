En esta sección se muestran varias configuraciones que pueden emplearse con el **componente Console**:

1.  Cambiar el comando por defecto
2.  Llamar a un comando existente
3.  Construir una aplicación de un sólo comando

### 1. Cambiar el comando por defecto

El componente Console siempre llamará a _ListCommand_ cuando no se le pase ningún otro comando. Para cambiar el comando por defecto simplemente tienes que pasar el nombre del comando al método _setDefaultCommand_:

```
namespace Acme\Console\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class HelloWorldCommand extends Command
{
    protected function configure()
    {
        $this->setName('hello:world')
            ->setDescription('Devuelve \'Hola Mundo\'');
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $output->writeln('Hola Mundo');
    }
}
```

Ahora ejecutamos la aplicación cambiando el comando por defecto:

```
// application.php

use Acme\Console\Command\HelloWorldCommand;
use Symfony\Component\Console\Application;

$command = new HelloWorldCommand();
$application = new Application();
$application->add($command);
$application->setDefaultCommand($command->getName());
$application->run();
```

Testeamos el **nuevo comando de consola por defecto**:

```
php application.php
```

Y ahora se mostrará:

```
Hello World
```

La limitación de esta característica es que no se puede utilizar con argumentos de comandos.

### 2. Llamar a un comando existente

Si un comando depende de otro que se ejecuta antes que éste, en lugar de preguntar al usuario el orden de ejecución, puedes llamarlo directamente tú mismo. Esto también es útil si quieres **crear un comando "meta**" que simplemente **ejecuta un conjunto de comandos** (por ejemplo, todos los comandos que han de ejecutarse cuando el código del proyecto ha cambiado en los servidores de producción: limpiar la caché, generar Doctrine2 proxies, volcar assets de Assetic, etc).

Llamar a un comando desde otro es sencillo:

```
protected function execute(InputInterface $input, OutputInterface $output)
{
    $command = $this->getApplication()->find('demo:greet');

    $arguments = array(
        'command' => 'demo:greet',
        'name'    => 'Fabien',
        '--yell'  => true,
    );

    $greetInput = new ArrayInput($arguments);
    $returnCode = $command->run($greetInput, $output);

    // ...
}
```

Primero se emplea el método [_find()_](http://api.symfony.com/3.0/Symfony/Component/Console/Application.html#method_find) para encontrar el comando que queremos ejecutar pasándole el nombre del comando. Después creamos un [ArrayInput](http://api.symfony.com/3.0/Symfony/Component/Console/Input/ArrayInput.html) con los argumentos y opciones que queremos pasarle al comando. 

Finalmente llamando al método _run()_ se ejecuta el comando y devuelve el código devuelto por el comando (valor de retorno del método de comandos _execute()_). 

Si quieres suprimir el output del comando ejecutado, añade [NullOutput](http://api.symfony.com/3.0/Symfony/Component/Console/Output/NullOutput.html) como segundo argumento en _$command->execute()_. 

Todos los comandos se ejcutarán en el mismo proceso y algunos de los comandos incorporados de Symfony puede que no funcionen bien de esta forma. Por ejemplo los comandos cache:clear y cache:warmup cambian algunas definiciones de clases, por lo que ejecutar algo después de ellos hará que probablemente se interrumpa.

La mayoría de las veces **llamar a un comando desde el código** que no se ejecuta en la línea de comandos no es una buena idea por varias razones. En primer lugar, el output del comando está optimizado para la consola. Pero lo más importante, puede verse a un comando como un _controller_: debería utilizar el _model_ para hacer algo y mostrar un feedback al usuario. Así que en lugar de llamar a un comando desde la web, refactoriza el código y mueve la lógica a una nueva clase.

### 3\. Construir una aplicación de un sólo comando

Cuando se construye una herramienta de línea de comandos, puedes no necesitar emplear varios comandos. En ese caso, tener que pasar el nombre del comando cada vez puede resultar tedioso. Afortunadamente es posible remover esta necesidad extendiendo la aplicación:

```
namespace Acme\Tool;

use Symfony\Component\Console\Application;
use Symfony\Component\Console\Input\InputInterface;

class MyApplication extends Application
{
    /**
     * Obtiene el nombre del comando basado en input.
     *
     * @param InputInterface $input The input interface
     *
     * @return string The command name
     */
    protected function getCommandName(InputInterface $input)
    {
        // Esto debe devolver el nombre de tu comando.
        return 'my_command';
    }

    /**
     * Obtiene los comandos por defecto que siempre deben estar disponibles.
     *
     * @return array An array of default Command instances
     */
    protected function getDefaultCommands()
    {
        // Mantén los comandos por defecto con HelpCommand
        // que se emplea cuando se utiliza la opción --help
        $defaultCommands = parent::getDefaultCommands();

        $defaultCommands[] = new MyCommand();

        return $defaultCommands;
    }

    /**
     * Se sobreescribe de forma que la aplicación no espera que el nombre
     * del comando sea el primer argumento.
     */
    public function getDefinition()
    {
        $inputDefinition = parent::getDefinition();
        // se quita el primer argumento de normal, que es el nombre del comando
        $inputDefinition->setArguments();

        return $inputDefinition;
    }
}
```

Cuando se llame a un comando de consola, el comando **MyCommand** siempre se usará, sin tener que pasar su nombre. 

Puedes también simplificar cómo se ejecutará la aplicación:

```
#!/usr/bin/env php
<?php
// command.php

use Acme\Tool\MyApplication;

$application = new MyApplication();
$application->run();
```