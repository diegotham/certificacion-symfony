Cuando añades cualquier línea de código, puedes añadir también sin quererlo algún bug. Para construir mejores aplicaciones y más estables, lo ideal es testear el código con unit y functional tests.

En **Symfony** viene incorporado **PHPUnit**, una librería independiente para testear ([documentación](https://phpunit.de/manual/current/en/)).

Cada test es una **clase PHP** que ha de ir localizada en el directorio _/tests_, de esta forma basta con ejecutar el siguiente comando:

```
phpunit
```

La **configuración de PHPUnit** se encuentra en el archivo _phpunit.xml.dist_, en el directorio root de tu aplicación Symfony.

### Unit Tests

Los unit tests comprueban una clase específica, también llamada _unit_. Para crear **Symfony unit tests** se hace de la misma forma que para crear **PHPUnit tests**. Si por ejemplo tenemos una clase **Calculator** en el directorio _/Util_ del bundle **AppBundle**:

```
// src/AppBundle/Util/Calculator.php
namespace AppBundle\Util;

class Calculator
{
    public function add($a, $b)
    {
        return $a + $b;
    }
}
```

Para testear al código anterior, creamos un archivo **CalculatorTest** en el directorio _tests/AppBundle/Util_:

```
// tests/AppBundle/Util/CalculatorTest.php
namespace Tests\AppBundle\Util;

use AppBundle\Util\Calculator;

class CalculatorTest extends \PHPUnit_Framework_TestCase
{
    public function testAdd()
    {
        $calc = new Calculator();
        $result = $calc->add(30, 12);

        // nos aseguramos que la calculadora ha sumado los números correctamente
        $this->assertEquals(42, $result);
    }
}
```

Por convención, el directorio _Tests/AppBundle_ debe replicar el directorio de tu bundle para unit tests, por lo que si testeamos una clase en el directorio _AppBunde/Util/_, añadiremos el test en el directorio _tests/AppBundle/Util/_. 

Al igual que en una aplicación real, autoloading se activa automáticamente a través del archivo _autoload.php_ (tal y como está configurado por defecto en  el archivo _phpunit.xml.dist_).

Ejecutar tests para un determinado archivo o directorio se hace como sigue:

```
# ejecutar todos los tests de la aplicación
$ phpunit

# ejecutar todos los tests del directorio /Util
$ phpunit tests/AppBundle/Util

# ejecutar los tests de la clase Calculator
$ phpunit tests/AppBundle/Util/CalculatorTest.php

# ejecutar los tests de todo el bundle
$ phpunit tests/AppBundle/
```