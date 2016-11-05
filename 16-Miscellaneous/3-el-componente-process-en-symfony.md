El **componente Process** ejecuta comandos en subprocesos. 

```
use Symfony\Component\Process\Process;
use Symfony\Component\Process\Exception\ProcessFailedException;

$process = new Process('ls -lsa');
$process->run();

// se ejecuta después de que el comando finaliza
if (!$process->isSuccessful()) {
    throw new ProcessFailedException($process);
}

echo $process->getOutput();
```

El componente tiene en cuenta las diferencias entre las diferentes plataformas cuando ejecuta el comando.

El método _getOutput()_ siempre devuelve todo el contenido del output estándar del comando y _getErrorOutput()_ el contenido del error output. Alternativamente, los métodos _getIncrementalOutput()_ y _getIncrementalErrorOutput()_ devuelven los nuevos outputs desde la última llamada. 

El método _clearOutput()_ limpia los contenidos del output y _clearErrorOutput()_ limpia los contenidos del error output.

El método _mustRun()_ es idéntico a _run()_, excepto que lanzará una [ProcessFailedException](http://api.symfony.com/3.0/Symfony/Component/Process/Exception/ProcessFailedException.html) si el proceso no se ha podido ejecutar de forma satisfactoria (por ejemplo, el proceso finalizó con un código non-zero).

```
use Symfony\Component\Process\Exception\ProcessFailedException;
use Symfony\Component\Process\Process;

$process = new Process('ls -lsa');

try {
    $process->mustRun();

    echo $process->getOutput();
} catch (ProcessFailedException $e) {
    echo $e->getMessage();
}
```

### Obtener output en tiempo real

Cuando se ejecuta un comando de larga ejecución (como archivos rsync-ing a un servidor remoto), puedes dar un feedback al usuario final en tiempo real pasando una función anónima al método [run()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_run):

```
use Symfony\Component\Process\Process;

$process = new Process('ls -lsa');
$process->run(function ($type, $buffer) {
    if (Process::ERR === $type) {
        echo 'ERR > '.$buffer;
    } else {
        echo 'OUT > '.$buffer;
    }
});
```

### Ejecutar procesos de forma asíncrona

Puedes también empezar el subproceso y dejarlo correr de forma asíncrona, devolviendo output y el estado del proceso principal cuando lo necesites. Utiliza el método [start()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_start) para comenzar un proceso asíncrono, el método [isRunning()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_isRunning) para comprobar si el proceso ha terminado y el método [getOutput()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_getOutput) para obtener el output:

```
$process = new Process('ls -lsa');
$process->start();

while ($process->isRunning()) {
    // esperando a que el proceso finalice
}

echo $process->getOutput();
```

Puedes también esperar a que finalice el proceso si lo comenzaste de forma asíncrona y has terminado con otras cosas.

```
$process = new Process('ls -lsa');
$process->start();

// ... hacer otras cosas

$process->wait(function ($type, $buffer) {
    if (Process::ERR === $type) {
        echo 'ERR > '.$buffer;
    } else {
        echo 'OUT > '.$buffer;
    }
});
```

El método [wait()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_wait) está bloqueando, lo que significa que tu código se va a interrumpir en esta línea hasta que el proceso externo esté completado.

### Parar un proceso

Cualquier proceso asíncrono puede pararse en cualquier momento con el método [stop()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_stop). Este método toma dos argumentos: un timeout y una señal. Una vez que se alcanza el timeout, la señal se envía al proceso en marcha. La señal enviada por defecto es **SIGKILL**.

```
$process = new Process('ls -lsa');
$process->start();

// ... hacer otras cosas

$process->stop(3, SIGINT);
```

### Ejecutar código PHP aislado

Si quieres **ejecutar código PHP aislado**, utiliza _PhpProcess_:

```
use Symfony\Component\Process\PhpProcess;

$process = new PhpProcess(<<<EOF
    <?php echo 'Hello World'; ?>
EOF
);
$process->run();
```

Para que tu código funcione mejor en todas las plataformas, podrías querer utilizar la clase [ProcessBuilder](http://api.symfony.com/3.0/Symfony/Component/Process/ProcessBuilder.html) en su lugar:

```
use Symfony\Component\Process\ProcessBuilder;

$builder = new ProcessBuilder(array('ls', '-lsa'));
$builder->getProcess()->run();
```

Si estás construyendo un driver binario, puedes utilizar el método [setPrefix()](http://api.symfony.com/3.0/Symfony/Component/Process/ProcessBuilder.html#method_setPrefix) para prefijar todos los comandos de procesos generados.

El siguiente ejemplo genera dos comandos de proceso para un adaptador tar binario:

```
use Symfony\Component\Process\ProcessBuilder;

$builder = new ProcessBuilder();
$builder->setPrefix('/usr/bin/tar');

// '/usr/bin/tar' '--list' '--file=archive.tar.gz'
echo $builder
    ->setArguments(array('--list', '--file=archive.tar.gz'))
    ->getProcess()
    ->getCommandLine();

// '/usr/bin/tar' '-xzf' 'archive.tar.gz'
echo $builder
    ->setArguments(array('-xzf', 'archive.tar.gz'))
    ->getProcess()
    ->getCommandLine();
```

### Process Timeout

Puedes **limitar la cantidad de tiempo que un proceso tarda en completarse** estableciendo un **timeout** en segundos:

```
use Symfony\Component\Process\Process;

$process = new Process('ls -lsa');
$process->setTimeout(3600);
$process->run();
```

Si se alcanza el timeout, se lanza una excepción [RuntimeException](http://api.symfony.com/3.0/Symfony/Component/Process/Exception/RuntimeException.html).

Para comandos de larga ejecución es tu responsabilidad checkear el timeout de forma regular:

```
$process->setTimeout(3600);
$process->start();

while ($condition) {
    // ...

    // comprobar si se ha llegado al timeout
    $process->checkTimeout();

    usleep(200000);
}
```

### Process Idle Timeout

En contraste con el timeout anterior, el **idle timeout** sólo considera el tiempo desde el último output producido por el proceso:

```
use Symfony\Component\Process\Process;

$process = new Process('something-with-variable-runtime');
$process->setTimeout(3600);
$process->setIdleTimeout(60);
$process->run();
```

En este caso, un proceso es considerado timed out ya sea cuando el tiempo total de ejecución excede los 3600 segundos o el proceso no produce ningún output en 60 segundos.

### Process Signals

Cuando se ejecuta un programa de forma asíncrona, puedes enviarle señales **POSIX** con el método [signal()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_signal):

```
use Symfony\Component\Process\Process;

$process = new Process('find / -name "rabbit"');
$process->start();

// enviará un SIGKILL al proceso
$process->signal(SIGKILL);
```

Debido a algunas limitaciones en PHP, si utilizas señales con el componente Process, podrías tener que prefijar los comandos con [exec](https://en.wikipedia.org/wiki/Exec_(operating_system)). Hay dos issues en Github y php.net que explican el por qué: [#5759](https://github.com/symfony/symfony/issues/5759) y [#39992](https://bugs.php.net/bug.php?id=39992) respectivamente.

### Process Pid

Puedes acceder al [pid](https://en.wikipedia.org/wiki/Process_identifier) de un proceso en marcha con el método [getPid()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_getPid). 

```
use Symfony\Component\Process\Process;

$process = new Process('/usr/bin/php worker.php');
$process->start();

$pid = $process->getPid();
```

Debido a algunas limitaciones en PHP, si quieres obtener el pid de un proceso symfony podrías tener que prefijar tus comandos con [exec](http://en.wikipedia.org/wiki/Exec_(operating_system)). Puedes leer el por qué en el issue [#5759](https://github.com/symfony/symfony/issues/5759).

### Disabling Output

Un output estándar y un error output son siempre recuperados en un proceso determinado, y puede ser conveniente desactivar el output en algunos casos para ahorrar memoria. Puedes emplear los métodos [disableOutput()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_disableOutput) y [enableOutput()](http://api.symfony.com/3.0/Symfony/Component/Process/Process.html#method_enableOutput) para alternar esta característica:

```
use Symfony\Component\Process\Process;

$process = new Process('/usr/bin/php worker.php');
$process->disableOutput();
$process->run();
```

Ten en cuenta que no puedes activar o desactivar el output mientras el proceso está en marcha.

Si desactivas el output, no tienes acceso a _getOutput_, _getIncrementalOutput_, _getErrorOutput_ o _getIncrementalErrorOutput_. Además, no podrías pasar un callback a los métodos _start_, _run_ o _mustRun_ o usar _setIdleTimeout_.