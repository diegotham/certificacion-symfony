El **componente Console** permite la creación de interfaces de línea de comandos para después emplearse en tareas como **cronjobs**, importaciones y otras agrupaciones de tareas.

Para **crear un comando básico** que saluda desde la línea de comandos, creamos el archivo _GreetCommand.php_ y añadimos lo siguiente:

```
namespace Acme\Console\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;

class GreetCommand extends Command
{
    protected function configure()
    {
        $this
            ->setName('demo:greet')
            ->setDescription('Saludar a alguien')
            ->addArgument(
                'nombre',
                InputArgument::OPTIONAL,
                'A quién quieres saludar?'
            )
            ->addOption(
               'gritar',
               null,
               InputOption::VALUE_NONE,
               'Si se establece, se mostrará en mayúsculas'
            )
        ;
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $name = $input->getArgument('nombre');
        if ($name) {
            $text = 'Hola '.$name;
        } else {
            $text = 'Hola';
        }

        if ($input->getOption('gritar')) {
            $text = strtoupper($text);
        }

        $output->writeln($text);
    }
}
```

Además necesitaremos crear el archivo que arrancará en la línea de comandos que crea una **Application** y le añade comandos:

```
#!/usr/bin/env php
<?php
// application.php

require __DIR__.'/vendor/autoload.php';

use Acme\Console\Command\GreetCommand;
use Symfony\Component\Console\Application;

$application = new Application();
$application->add(new GreetCommand());
$application->run();
```

Para probar el nuevo **comando de consola** podemos escribir lo siguiente:

```
php application.php demo:greet Fabien
```

Esto devolverá:

```
Hello Fabien
```

Si queremos especificar la opción _--gritar_:

```
php application.php demo:greet Fabien --yell
```

Esta vez devolverá el texto en mayúsculas:

```
HELLO FABIEN
```

### Ciclo de los comandos

Los comandos tienen tres métodos lifecycle:

*   _initialize()_. **Opcional**. Se ejectua antes que los métodos _interact()_ y _execute()_. Su principal objetivo es iniciar variables que se usarán en el resto de los métodos de comandos.
*   _interact()_. **Opcional**. Este método se ejecuta después de _initialize()_ y antes de _execute()_.
*   _execute()_. **Requerido**. Este método se ejecuta después de _interact()_ e _initialize()_. Contiene la lógica que quieres que ejecute el comando.

### Colorear el output

*En sistemas Windows es necesario [añadir alguna aplicación](http://symfony.com/doc/current/components/console/introduction.html#coloring-the-output).

Al mostrar el output podemos **añadir etiquetas a los textos para añadirles colores**:

```
// texto en verde
$output->writeln('<info>foo</info>');

// texto en amarillo
$output->writeln('<comment>foo</comment>');

// texto en negro con un fondo cián
$output->writeln('<question>foo</question>');

// texto en blanco con un fondo rojo
$output->writeln('<error>foo</error>');
```

La etiqueta de cierre puede reemplazarse por _</>_ que revoca todas las opciones de formateo establecidas por la última etiqueta abierta.

Puedes también definir tus propios estilos con la clase **OutputFormatterStyle**:

```
use Symfony\Component\Console\Formatter\OutputFormatterStyle;

// ...
$style = new OutputFormatterStyle('red', 'yellow', array('bold', 'blink'));
$output->getFormatter()->setStyle('fire', $style);
$output->writeln('<fire>foo</>');
```

 Los **colores disponibles** son: _black_, _red_, _green_, _yellow_, _blue_, _magenta_, _cyan_ y _white_.

Y las **opciones disponibles** son: _bold_, _underscore_, _blink_, _reverse_ (activa el modo "reverse video" donde los colores del fondo y el texto se intercambian) y _conceal_ (establece el color de fuente a transparente, haciendo que el texto sea invisible, aunque puede ser seleccionado y copiado. Esta opción es usada normalmente para información sensible como contraseñas).

 Puedes también establecer estos colores y opciones dentro del nombre de la etiqueta:

```
// texto en verde
$output->writeln('<fg=green>foo</>');

// texto en negro con fondo cián
$output->writeln('<fg=black;bg=cyan>foo</>');

// texto en negrita en fondo amarillo
$output->writeln('<bg=yellow;options=bold>foo</>');
```