La consola tiene cinco niveles de verbosidad, definidos en la [OutputInterface](http://api.symfony.com/3.0/Symfony/Component/Console/Output/OutputInterface.html):

| | | |
| -------- | -------- |
| **Valor** | **Significado** | **Opción de consola** |
| _OutputInterface::VERBOSITY_QUIET_ | No muestra ningún mensaje | _-q_ o _--quiet_ |
| _OutputInterface::VERBOSITY_NORMAL_ | El valor por defecto de verbosidad | (ninguna) |
| _OutputInterface::VERBOSITY_VERBOSE_ | Incrementa la verbosidad de los mensajes | _-v_ |
| _OutputInterface::VERBOSITY_VERY_VERBOSE_ | Mensajes informativos no esenciales | _-vv_ |
| _OutputInterface::VERBOSITY_DEBUG_ | Mensajes de debug | _-vvv_ |

El **stacktrace** de excepciones entero se muestra si el nivel de verbosidad es **VERBOSITY_VERBOSE** o superior.

Es posible imprimir un mensaje en un comando sólo para un nivel de verbosidad especificado. Por ejemplo:

```
if ($output->getVerbosity() >= OutputInterface::VERBOSITY_VERBOSE) {
    $output->writeln(...);
}
```

Hay también métodos más semánticos que puedes utilizar para testear cada uno de los niveles de verbosidad:

```
if ($output->isQuiet()) {
    // ...
}

if ($output->isVerbose()) {
    // ...
}

if ($output->isVeryVerbose()) {
    // ...
}

if ($output->isDebug()) {
    // ...
}
```

Cuando se utiliza el nivel _quiet_ todo el output se suprime por lo que el método [write()](http://api.symfony.com/3.0/Symfony/Component/Console/Output/Output.html#method_write) se devuelve sin contenido.

El **MonologBridge** proporciona una clase [ConsoleHandler](http://api.symfony.com/3.0/Symfony/Bridge/Monolog/Handler/ConsoleHandler.html) que te permite mostrar mensahes en la consola. Esta forma es más limpia que agrupar los outputs con condiciones. Puedes leer el artículo [Cómo configurar Monolog para mostrar mensajes de consola](http://symfony.com/doc/current/cookbook/logging/monolog_console.html).