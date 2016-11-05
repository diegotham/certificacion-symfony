### Helpers

El **componente Console** también tiene una serie de helpers, pequeñas herramientas que pueden ayudarte en diferentes tareas: 

*   [Question Helper](http://symfony.com/doc/current/components/console/helpers/questionhelper.html): pregunta interactivamente al usuario para obtener información
*   [Formatter Helper](http://symfony.com/doc/current/components/console/helpers/formatterhelper.html): customiza el output con colores
*   [Progress Bar](http://symfony.com/doc/current/components/console/helpers/progressbar.html): muestra una barra de progreso
*   [Table](http://symfony.com/doc/current/components/console/helpers/table.html): muestra datos tabulados como una tabla

### Testing

Symfony proporciona varias herramientas para **testear los comandos**. La más útil es la clase [CommandTester](http://api.symfony.com/3.0/Symfony/Component/Console/Tester/CommandTester.html). Utiliza clases _input_ y _output_ especiales para facilitar el testing sin una consola real:

```
use Acme\Console\Command\GreetCommand;
use Symfony\Component\Console\Application;
use Symfony\Component\Console\Tester\CommandTester;

class ListCommandTest extends \PHPUnit_Framework_TestCase
{
    public function testExecute()
    {
        $application = new Application();
        $application->add(new GreetCommand());

        $command = $application->find('demo:greet');
        $commandTester = new CommandTester($command);
        $commandTester->execute(array('command' => $command->getName()));

        $this->assertRegExp('/.../', $commandTester->getDisplay());

        // ...
    }
}
```

El método [getDisplay()](http://api.symfony.com/3.0/Symfony/Component/Console/Tester/CommandTester.html#method_getDisplay) devuelve lo que se hubiera mostrado durante una llamada normal en la consola.

Puedes testear enviar argumentos y opciones al comando añadiéndolos como array al método [execute()](http://api.symfony.com/3.0/Symfony/Component/Console/Tester/CommandTester.html#method_execute):

```
use Acme\Console\Command\GreetCommand;
use Symfony\Component\Console\Application;
use Symfony\Component\Console\Tester\CommandTester;

class ListCommandTest extends \PHPUnit_Framework_TestCase
{
    // ...

    public function testNameIsOutput()
    {
        $application = new Application();
        $application->add(new GreetCommand());

        $command = $application->find('demo:greet');
        $commandTester = new CommandTester($command);
        $commandTester->execute(array(
            'command'      => $command->getName(),
            'name'         => 'Fabien',
            '--iterations' => 5,
        ));

        $this->assertRegExp('/Fabien/', $commandTester->getDisplay());
    }
}
```

Puedes también testear una aplicación console entera con [AplicationTester](http://api.symfony.com/3.0/Symfony/Component/Console/Tester/CommandTester.html).