Un [patrón bridge](https://en.wikipedia.org/wiki/Bridge_pattern) sirve para "_desacoplar una abstracción de su implementación, de forma que ambas puedan variar independientemente_". El bridge utiliza **encapsulación**, **agregación** y puede usar **herencia** para separar responsabilidades en clases diferentes.

El [componente PHPUnit bridge](http://symfony.com/doc/current/components/phpunit_bridge.html) de Symfony proporciona utilidades para informar de legacy tests y el uso de código obsoleto y un helper para tests sensibles al tiempo.

Viene con las siguientes características: 

*   Fuerza a los tests a utilizar un **locale** consistente (C)
*   Auto registra _class_exists_ para cargar anotaciones **Doctrine** (cuando se usen)
*   Muestra la lista de **código obsoleto** utilizado en la aplicación
*   Muestra el **stack trace** de una obsolescencia bajo demanda
*   Proporciona una clase helper **ClockMock** para tests sensibles al tiempo

Al ejecutar los tests, puedes obtener una ventana como la siguiente:

![Ejemplo de PHPUnit test](http://symfony.com/doc/current/_images/report.png)

El sumario incluye: 

*   **Unsilenced**. Informa de deprecation notices lanzados sin el operador recomendable [@-silencing operator](http://php.net/manual/en/language.operators.errorcontrol.php).
*   **Legacy**. Deprecation notices que muestran tests que prueban algunas características legacy.
*   **Remaining/Other**. Todos los demás deprecation notices (non-legacy), agrupados por mensaje, y método y clase del test. 

### Lanzar deprecation notices

Los depretacion notices pueden ser lanzados utilizando:

```
@trigger_error('Your deprecation message', E_USER_DEPRECATED);
```

Sin el [@-silencing operator](http://php.net/manual/en/language.operators.errorcontrol.php), los usuarios tendrían que dejar los deprecation notices. Silenciar por defecto intercambia este comportamiento y permite a los usuarios que los activen cuando estén preparados para manejarlos (añadiendo un error handler customizado como el proporcionado por este bridge). Cuando no estén silenciados, los deprecation notices aparecerán en la sección **Unsilenced** del reporte de obsolescencia.

### Marcar tests como legacy

Hay 4 formas de marcar un test como legacy:

*   La recomendada es añadir la anotación _@group legacy_ a su clase o método
*   Que el nombre de su clase empiece por _Legacy_
*   Que el nombre del método empiece por _testLegacy_ en lugar de test
*   Que el data provider empiece con _provideLegacy_ o _getLegacy_

### Configuración

En caso de que necesites inspeccionar el stack trace de una obsolescencia en particular lanzada por tus unit tests, puedes establecer la [variable de entorno](https://phpunit.de/manual/current/en/appendixes.configuration.html#appendixes.configuration.php-ini-constants-variables) **SYMFONY_DEPRECATIONS_HELPER** a una expresión regular que coincida con este mensaje de obsolescencia, junto con _/_. Por ejemplo con:

```
<!-- http://phpunit.de/manual/4.1/en/appendixes.configuration.html -->
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://schema.phpunit.de/4.1/phpunit.xsd"
>

    <!-- ... -->

    <php>
        <server name="KERNEL_DIR" value="app/" />
        <env name="SYMFONY_DEPRECATIONS_HELPER" value="/foobar/" />
    </php>
</phpunit>
```

**PHPUnit** parará los tests una vez que se ha lanzado un deprecation notice cuyo mensaje contiene el string "foobar".

### Hacer que un test falle

Por defecto, cualquier deprecation notice non-legacy-tagged o non-@-silenced harán que el test falle. De forma alternativa, estableciendo SYMFONY_DEPRECATIONS_HELPER a un valor arbitrario (por ejemplo 320) hará que el test falle sólo si se alcanza un número mayor de deprecation notices (por defecto es 0). Puedes también establecer el valor "weak" que hará que el bridge ignore cualquier **deprecation notice**. Esto es útil en proyectos que deben utilizar interfaces obsoletas por razones de compatibilidad con versiones anteriores.

### Tests sensibles al tiempo

Si tienes este tipo de tests:

```
use Symfony\Component\Stopwatch\Stopwatch;

class MyTest extends \PHPUnit_Framework_TestCase
{
    public function testSomething()
    {
        $stopwatch = new Stopwatch();

        $stopwatch->start();
        sleep(10);
        $duration = $stopwatch->stop();

        $this->assertEquals(10, $duration);
    }
}
```

Has utilizado el [componente StopWatch](http://symfony.com/doc/current/components/stopwatch.html) para calcular el tiempo de duración del proceso, aquí 10 segundos. Sin embargo, dependiendo en la carga del servidor, _$duration_ podría ser por ejemplo 10.00012 en lugar de 10s.

Este tipo de tests se llaman tests transitorios (_transient tests_): fallan de forma aleatoria dependiendo de circunstancias externas. Provocan continuos problemas cuando se emplean services de integración contínua como [Travis CI](https://travis-ci.com/).

#### Clock Mocking

La clase **ClockMock** proporcionada por este bridge permite imitar las **funciones de tiempo de PHP** _time()_, _microtime()_, _sleep()_, _usleep()_. 

Para utilizar la clase ClockMock en tu test, puedes: 

*   (Recomendable) Añadir la anotación @group time-sensitive a su clase o método
*   Registrarla manualmente llamando a _ClockMock::register(__CLASS__)_ y _ClockMock::withClockMock(true)_ antes del test y _ClockMock::withClockMock(false)_ después del test.

Como resultado, lo siguiente funciona seguro y ya no es un 

#### Solución de problemas

La anotación _@group time-sensitive_ funciona "por convención" y asume que el namespace de la clase testeadda puede obtenerse removiendo la parte _\Tests\_ del namespace del test. Por ejemplo si el **fully qualified class name** de tu test es _App\Tests\Watch\DummyWatchTest_, asume que el [fully qualified class name](https://diego.com.es/namespaces-en-php#ReglasDeResolucionDeLosNamespaces) de la clase testeada es _App\Watch\DummyWatch_.

Si esta convención no funciona en tu aplicación, puedes también configurar imitaciones de namespaces en el archivo _phpunit.xml_, tal como se hace por ejempo en el [HttpKernel Component](http://symfony.com/doc/current/components/http_kernel/introduction.html):

```
<!-- http://phpunit.de/manual/4.1/en/appendixes.configuration.html -->
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://schema.phpunit.de/4.1/phpunit.xsd"
>

    <!-- ... -->

    <listeners>
        <listener class="Symfony\Bridge\PhpUnit\SymfonyTestsListener">
            <arguments>
                <array>
                    <element><string>Symfony\Component\HttpFoundation</string></element>
                </array>
            </arguments>
        </listener>
    </listeners>
</phpunit>
```