El cliente del test simula un **cliente HTTP** (como un **navegador**) y hace _requests_ en tu aplicación Symfony:

```
$crawler = $client->request('GET', '/post/hello-world');
```

El método _request()_ toma el **método HTTP** y la **URL** como argumentos y devuelve una instancia de **Crawler**.

Lo mejor es **escribir las URLs directamente** en lugar de generarlas con el **Symfony router**, ya que sino no detectará los cambios de URLs que pueden afectar al usuario final.

La composición del método _request()_ es la siguiente: 

```
request(
    $method,
    $uri,
    array $parameters = array(),
    array $files = array(),
    array $server = array(),
    $content = null,
    $changeHistory = true
)
```

El array _server_ son los valores que normalmente encontrarías en la superglobal $**_SERVER** de PHP. Por ejemplo para establecer los headers HTTP _Content-Type_, _Referer_ y _X-Requested-With_, pasarías lo siguiente:

```
$client->request(
    'GET',
    '/post/hello-world',
    array(),
    array(),
    array(
        'CONTENT_TYPE'          => 'application/json',
        'HTTP_REFERER'          => '/foo/bar',
        'HTTP_X-Requested-With' => 'XMLHttpRequest',
    )
);
```

 Utilizamos el crawler para encontrar elementos del DOM en la respuesta. Estos elementos pueden utilizarse para hacer click en enlaces y enviar formularios:

```

$link = $crawler->selectLink('Go elsewhere...')->link();
$crawler = $client->click($link);

$form = $crawler->selectButton('validate')->form();
$crawler = $client->submit($form, array('name' => 'Fabien'));
```

Los métodos _click()_ y _submit()_ devuelven ambos un objeto **Crawler**. Estos métodos son la mejor forma de navegar por la aplicación ya que tienen en cuenta bastantes cosas, como detectar el método HTTP de un formulario y ofrecer una buena API para subir archivos. 

El método _request_ puede también utilizarse para simular envíos de formularios directamente o realizar requests más complejos. Algunos ejemplos útiles:

```
// Enviar directamente un formulario (utilizar el Crawler es más fácil)
$client->request('POST', '/submit', array('name' => 'Fabien'));

// Enviar un string JSON en el cuerpo del request
$client->request(
    'POST',
    '/submit',
    array(),
    array(),
    array('CONTENT_TYPE' => 'application/json'),
    '{"name":"Fabien"}'
);

// Envío de formulario con archivo
use Symfony\Component\HttpFoundation\File\UploadedFile;

$photo = new UploadedFile(
    '/path/to/photo.jpg',
    'photo.jpg',
    'image/jpeg',
    123
);
$client->request(
    'POST',
    '/submit',
    array('name' => 'Fabien'),
    array('photo' => $photo)
);

// Establecer un request DELETE y pasar headers HTTP
$client->request(
    'DELETE',
    '/post/12',
    array(),
    array(),
    array('PHP_AUTH_USER' => 'username', 'PHP_AUTH_PW' => 'pa$$word')
);
```

Puedes forzar a cada request a ser ejecutado en su propio proceso PHP para evitar efectos secundarios cuando se trabaja con varios clientes en el mismo script:

```
$client->insulate();
```

#### Navegar

El cliente soporta muchas operaciones que pueden hacerse en un navegador real:

```
$client->back();
$client->forward();
$client->reload();

// Limpia todas las cookies y el historial
$client->restart();
```

#### Acceder a objetos internos

Si utilizas el cliente para testear tu aplicación, puede que quieras acceso a los objetos internos del cliente:

```
$history = $client->getHistory();
$cookieJar = $client->getCookieJar();
```

Puedes también obtener los objetos relacionados con el último request:

```
// la instancia HttpKernel request
$request = $client->getRequest();

// la instancia BrowserKit request
$request = $client->getInternalRequest();

// la instancia HttpKernel response
$response = $client->getResponse();

// la instancia BrowserKit response
$response = $client->getInternalResponse();

$crawler = $client->getCrawler();
```

Si tus requests no están aislados (con _$client->insulate()_), puedes también acceder al **Container** y al **Kernel**:

```
$container = $client->getContainer();
$kernel = $client->getKernel();
```

#### Acceder al Container

Es muy **recomendable que un functional test sólo compruebe la respuesta**. Pero en algunos casos muy poco frecuentes, podrías querer acceder a algunos objetos internos para escribir _assertions_. En estos casos, puedes acceder al **Dependency Injection Container**:

```
$container = $client->getContainer();
```

Ten en cuenta que esto no funciona si aislas el cliente o si utilizas una capa HTTP. Para una lista de services disponibles en tu aplicación, puedes emplear el comando _debug:container_.

Si la información que deseas compribar está disponible desde el profiler, utilízalo en su lugar.

#### Acceder los datos del Profiler

En cada request puedes activar el **Symfony profiler** para coleccionar datos sobre el manejo interno de dicho request. Por ejemplo, el profiler podría utilizarse para verificar que una página ejecuta menos de un número de consultas a la base de datos cuando se carga.

Para obtener el profiler para el último request:

```
// activar el profiler para el request sucesivo
$client->enableProfiler();

$crawler = $client->request('GET', '/profiler');

// obtener el profile
$profile = $client->getProfile();
```

#### Redireccionar

Cuando un request devuelve una redirección como respuesta, el cliente no lo sigue de forma automática. Puedes examinar la respuesta y forzar una redirección después con el método _followRedirect()_:

```
$crawler = $client->followRedirect();
```

Si quieres que el cliente siga automáticamente todas las redirecciones, puedes forzarlo con el método _followRedirects()_:

```
$client->followRedirects();
```

Si estableces el argumento de _followRedirects()_ como _false_, las redirecciones no serán seguidas:

```
$client->followRedirects(false);
```