Es muy recomendable que un [functional test](https://diego.com.es/functional-tests-en-symfony) sólo compruebe la respuesta. Pero si escribes **tests funcionales** que monitorizan tus servidores de producción, podrías querer escribir tests con los datos del profiling ya que te proporciona una forma de **comprobar** diferentes cosas y realizar ciertas **métricas**.

El [Profiler de Symfony](http://symfony.com/doc/current/cookbook/profiler/index.html) guarda muchos datos en cada request. Utiliza estos datos para comprobar el número de llamadas a la base de datos, el tiempo empleado en el framework, etc. Pero antes de escribir _assertions_, activa el profiler y comprueba si el profiler está realmente disponible (está disponible por defecto en el entorno _test_):

```
class LuckyControllerTest extends WebTestCase
{
    public function testNumberAction()
    {
        $client = static::createClient();

        // Activa el profiler para el próximo request
        // (no hace nada si el profiler no está disponible)
        $client->enableProfiler();

        $crawler = $client->request('GET', '/lucky/number');

        // ... escribe algunas assertions sobre la respuesta

        // Comprueba que el profiler está activado
        if ($profile = $client->getProfile()) {
            // comprueba el número de requests
            $this->assertLessThan(
                10,
                $profile->getCollector('db')->getQueryCount()
            );

            // comprueba el tiempo empleado en el framework
            $this->assertLessThan(
                500,
                $profile->getCollector('time')->getDuration()
            );
        }
    }
}
```

Si un test falla por los datos del **profiling** (por ejemplo demasiadas consultas a la base de datos), podrías querer utilizar el **Web Profiler** para analizar el request después de que finalice el test. Es fácil de hacer si incrustas el token en el mensaje de error:

```
$this->assertLessThan(
    30,
    $profile->getCollector('db')->getQueryCount(),
    sprintf(
        'Comprobamos que el número de consultas es menos de 30 (token %s)',
        $profile->getToken()
    )
);
```

El **profiler store** puede ser diferente dependiendo del entorno (especialmente si utilizas **SQLite store**, que es el configurado por defecto).

La información del profiler está disponible incluso si aislas al cliente o si utilizas una capa HTTP para tus tests.

Puedes leer la API de [data collectors](http://symfony.com/doc/current/cookbook/profiler/data_collector.html) incorporados para aprender más sobre sus interfaces.

Para evitar coleccionar datos en cada test puedes establecer el parámetro _collect_ a false:

```
# app/config/config_test.yml

# ...
framework:
    profiler:
        enabled: true
        collect: false
```

De esta forma sólo comprueba que la llamada _$client->enableProfiler()_ coleccionará datos.