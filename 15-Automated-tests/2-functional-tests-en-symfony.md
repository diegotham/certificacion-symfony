Los **functional tests** o tests funcionales comprueban la integración de diferentes capas de una aplicación (desde el enrutamiento a las vistas). No son diferentes a los [unit tests](https://diego.com.es/unit-tests-en-symfony) desde el punto de vista de **PHPUnit,** pero tienen un **workflow** específico:

*   Realiza un request
*   Comprueba la respuesta
*   Hace click en un link o rellena un formulario
*   Comprueba la respuesta
*   Resfresca y repite

Los functional tests son simplemente archivos PHP que normalmente se encuentran en el directorio _tests/AppBundle/Controller_ de tu bundle. Si quieres comprobar las páginas administradas por tu clase **PostController**, comienza creando un nuevo archivo _PostControllerTest.php_ que extiende una clase especial **WebTestCase**. Como ejemplo:

```
// tests/AppBundle/Controller/PostControllerTest.php
namespace Tests\AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class PostControllerTest extends WebTestCase
{
    public function testShowPost()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/post/hello-world');

        $this->assertGreaterThan(
            0,
            $crawler->filter('html:contains("Hello World")')->count()
        );
    }
}
```

Para ejecutar functional tests, la clase **WebTestCase** arranca el kernel de la aplicación. En la mayoría de los casos esto ocurre de forma automática, pero si tu kernel está en un directorio que no es el estándar, tendrás que modificar el archivo _phpunit.xml.dist_ para establecer la variable de entorno **KERNEL_DIR** al directorio de tu kernel:

```
<?xml version="1.0" charset="utf-8" ?>
<phpunit>
    <php>
        <server name="KERNEL_DIR" value="/path/to/your/app/" />
    </php>
    <!-- ... -->
</phpunit>
```

El método _createClient()_ devuelve un **cliente**, que es como un **navegador** que utilizarás para rastrear el sitio:

```
$crawler = $client->request('GET', '/post/hello-world');
```

El método _request()_ devuelve un objeto **Crawler** que puede usarse para seleccionar elementos en la respuesta, hacer click en enlaces o rellenar formularios.

El Crawler sólo funciona cuando la respuesta es un XML o un documento HTML. Para obtener la respuesta sin formato, puedes llamar a _$client->getResponse()->getContent()_. 

Hacemos click en un enlace seleccionándolo primero con el crawler empleando **expresiones XPath** o un **selector CSS**, y utilizando el cliente para hacer click en él. Por ejemplo:

```
$link = $crawler
    ->filter('a:contains("Greet")') // seleccionamos todos los enlaces con el texto "Greet"
    ->eq(1) // seleccionamos el segundo enlace de la lista
    ->link()
;

// y hacemos click en él
$crawler = $client->click($link);
```

**Enviar un formulario** es muy similar: seleccionamos un botón del formulario, opcionalmente sobreescribimos algunos de sus valores y lo enviamos:

```
$form = $crawler->selectButton('submit')->form();

// establecemos algunos valores
$form['name'] = 'Lucas';
$form['form_name[subject]'] = 'Hey there!';

// enviamos el formulario
$crawler = $client->submit($form);
```

El formulario puede también manejar subidas de archivos y contiene métodos para rellenar diferentes tipos de campos de formularios (como _select()_ y _tick()_). 

Ahora que ya sabemos navegar por la aplicación, utilizamos _assertions_ para testear que realmente hace lo que queramos que haga. Utilizamos el **Crawler** para hacer _assertions_ en el **DOM**:

```
// Aseguramos que la respuesta coincide con el selector CSS seleccionado.
$this->assertGreaterThan(0, $crawler->filter('h1')->count());
```

O testearel contenido de la respuesta si simplemente quieres asegurarte de que el contenido contiene un texto en concreto o en caso de que la respuesta no está en los formatos XML/HTML:

```
$this->assertContains(
    'Hello World',
    $client->getResponse()->getContent()
);
```

### Lista de assertions más comunes

Este es un resumen de las aserciones que más se utilizan:

```
use Symfony\Component\HttpFoundation\Response;

// ...

// Asegurarse que hay por lo menos una etiqueta h2
// con la clase "subtitle"
$this->assertGreaterThan(
    0,
    $crawler->filter('h2.subtitle')->count()
);

// Asegurar que hay exactamente 4 etiquetas h2 en la página
$this->assertCount(4, $crawler->filter('h2'));

// Asegurar que el header "Content-Type" es "application/json"
$this->assertTrue(
    $client->getResponse()->headers->contains(
        'Content-Type',
        'application/json'
    )
);

// Asegurar que el contenido de la respuesta contiene un string
$this->assertContains('foo', $client->getResponse()->getContent());
// ... o que coincide con una expresión regular
$this->assertRegExp('/foo(bar)?/', $client->getResponse()->getContent());

// Asegurar que el código de respuesta es 2xx
$this->assertTrue($client->getResponse()->isSuccessful());
// Asegurar que el código de respuesta es 404
$this->assertTrue($client->getResponse()->isNotFound());
// Asegurar un código de estado 200
$this->assertEquals(
    200, // or Symfony\Component\HttpFoundation\Response::HTTP_OK
    $client->getResponse()->getStatusCode()
);

// Asegurar que la respuesta es una redirección a /demo/contact
$this->assertTrue(
    $client->getResponse()->isRedirect('/demo/contact')
);
// ...o simplemente comprobar que la respuesta es una redirección a cualquier URL
$this->assertTrue($client->getResponse()->isRedirect());
```