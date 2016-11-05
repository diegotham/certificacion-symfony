Una de las partes más interesantes de los comandos son los **argumentos y las opciones** que puedes crear.

### Argumentos de comandos

Los argumentos son _strings_ (separadas por espacios) que van detrás del nombre de comando. Están ordenados y pueden ser opcionales o requeridos. Por ejemplo añadimos un argumento _last_name_ opcional a un comando y hacemos que el argumento _name_ sea requerido: 

```
$this
    // ...
    ->addArgument(
        'name',
        InputArgument::REQUIRED,
        '¿A quién quieres saludar?'
    )
    ->addArgument(
        'last_name',
        InputArgument::OPTIONAL,
        '¿Y tu apellido es...?'
    );
```

Ahora tienes acceso el argumento last_name en el comando:

```
if ($lastName = $input->getArgument('last_name')) {
    $text .= ' '.$lastName;
}
```

El comando puede usarse en cualquiera de las siguientes formas:

php application.php demo:greet Fabien
php application.php demo:greet Fabien Potencier

También se puede dejar que un argumento tenga una lista de valores (imagina que quieres saludar a toda una lista de amigos). Esto debe especificarse al final de la lista de argumentos:

```
$this
    // ...
    ->addArgument(
        'names',
        InputArgument::IS_ARRAY,
        '¿A quién quieres saludar (separa los nombres con un espacio)?'
    );
```

Ahora podemos especificar tantos nombres como queramos:

```
php application.php demo:greet Fabien Ryan Bernhard
```

Puedes acceder al argumento _names_ como un array:

```
if ($names = $input->getArgument('names')) {
    $text .= ' '.implode(', ', $names);
}
```

Existen tres variantes de argumentos que pueden emplearse:

| | |
| -------- | -------- |
| **Modo** | **Valor** |
| _InputArgument::REQUIRED_ | El argumento es requerido |
| _InputArgument::OPTIONAL_ | El argumento es opcional y por lo tanto puede omitirse |
| _InputArgument::IS_ARRAY_ | El argumento puede contener un número indefinido de argumentos y debe emplearse al final de la lista de argumentos |

Puedes combinar _IS_ARRAY_ con _REQUIRED_ y _OPTIONAL_:

```
$this
    // ...
    ->addArgument(
        'names',
        InputArgument::IS_ARRAY | InputArgument::REQUIRED,
        '¿A quién quieres saludar (separa los nombres con un espacio)?'
    );
```

### Opciones de comandos

Al contrario que los argumentos, las **opciones** no están ordenadas (puedes especificarlas en cualquier orden) y se especifican con 2 guiones (por ejemplo _--yell_, aunque también puedes especificar un shortcut de una letra de forma que quede así: _-y_). Las opciones son siempre opcionales, y pueden establecerse para aceptar un valor (por ejemplo: -_-dir=src_).

Es posible **crear un comando con una opción que acepta opcionalmente un valor**. Sin embargo, no hay forma de distinguir cuando la opción se ha empleado sin un valor (_command --yell_) o cuando no se ha empleado (_command_). En ambos casos, el valor devuelto para la opción será _null_. 

Por ejemplo, añadimos una nueva opción al comando que puede usarse para especificar cuántas veces ha de imprimirse el mensaje:

```
$this
    // ...
    ->addOption(
        'iterations',
        null,
        InputOption::VALUE_REQUIRED,
        '¿Cuántas veces ha de imprimirse el mensaje?',
        1
    );
```

Después, empleamos esto en el comando para imprimir el mensaje repetidas veces:

```
for ($i = 0; $i < $input->getOption('iterations'); $i++) {
    $output->writeln($text);
}
```

Ahora cuando ejecutemos la tarea puedemos especificar opcionalmente _--iterations_:

```
php application.php demo:greet Fabien
php application.php demo:greet Fabien --iterations=5
```

El primer ejemplo se imprimirá sólo una vez, ya que _iterations_ está vacío y por defecto es _1_ (el último argumento de _addOption_). El segundo ejemplo se imprimirá cinco veces.

Recuerda que no importa el orden de las opciones, por lo que cualquiera de las siguientes vale:

```
php application.php demo:greet Fabien --iterations=5 --yell
php application.php demo:greet Fabien --yell --iterations=5
```

Hay cuatro variantes de opciones que pueden usarse:

| | |
| -------- | -------- |
| **Opción** | **Valor** |
| _InputOption::VALUE_IS_ARRAY_ | Esta opción acepta múltiples valores (por ejemplo _--dir=/foo --dir=/bar_) |
| _InputOption::VALUE_NONE_ | No acepta input para esta opción (por ejemplo: _--yell_) |
| _InputOption::VALUE_REQUIRED_ | Este valor es requerido (por ejemplo: _--iterations=5_), la opción en sí todavía es opcional |
| _InputOption::VALUE_OPTIONAL_ | Esta opción puede o no tener un valor (por ejemplo _--yell_ o _--yell=loud_) |

Puedes combinar _VALUE_IS_ARRAY_ con _VALUE_REQUIRED_ o _VALUE_OPTIONAL_:

```
$this
    // ...
    ->addOption(
        'colores',
        null,
        InputOption::VALUE_REQUIRED | InputOption::VALUE_IS_ARRAY,
        '¿Qué colores te gustan?',
        array('azul', 'rojo')
    );
```