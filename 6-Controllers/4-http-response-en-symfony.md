Un objeto **Response** guarda toda la información que ha de ser devuelta al cliente a partir de un **request** solicitado. El constructor acepta hasta tres argumentos: el **contenido de la respuesta**, el **código de estado** y un **array de HTTP headers**:

```
use Symfony\Component\HttpFoundation\Response;

$response = new Response(
    'Content',
    Response::HTTP_OK,
    array('content-type' => 'text/html')
);
```

Esta información también puede ser manipulada después de la creación del **objeto Response**:

```
$response->setContent('Hello World');

// el atributo público headers es un ResponseHeaderBag
$response->headers->set('Content-Type', 'text/plain');

$response->setStatusCode(Response::HTTP_NOT_FOUND);
```

Cuando se establece el **Content-Type** del **Response**, puedes establecer el **charset**, pero es mejor establecerlo mediante el método _setCharset()_:

```
$response->setCharset('ISO-8859-1');
```

Por defecto, Symfony asume que tus respuestas irán en **UTF-8**. 

**Indice de contenido**

1.  Enviar la respuesta
2.  Establecer cookies
3.  Manejar la cache
4.  Redireccionar al usuario
5.  Respuesta streaming
6.  Servir archivos
7.  Crear una respuesta JSON
8.  JSONP callback

### 1. Enviar la respuesta

Antes de enviar la respuesta, puedes asegurarte de que obedece las **especificaciones HTTP** llamando al método _prepare()_:

```
$response->prepare($request);
```

Enviar la respuesta al cliente es tan simple como llamar al método _send()_:

```
$response->send();
```

### 2. Establecer cookies

Las cookies de respuesta pueden ser manipuladas con el **atributo** público _**headers**_:

```
use Symfony\Component\HttpFoundation\Cookie;

$response->headers->setCookie(new Cookie('foo', 'bar'));
```

El método [_setCookie()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/ResponseHeaderBag.html#method_setCookie) utiliza una instancia de [Cookie](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Cookie.html) como argumento.

Puedes limpiar una cookie a través del método [_clearCookie()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/ResponseHeaderBag.html#method_clearCookie).

### 3. Manejar la caché

La clase **Response** tiene un conjunto de métodos para manipular los **headers HTTP** relacionados con la caché:

*   _setPublic()_
*   _setPrivate()_
*   _expire()_
*   _setExpires()_
*   _setMaxAge();_
*   _setSharedMacAge();_
*   _setTtl();_
*   _setClientTtl();_
*   _setLastModified();_
*   _setEtag();_
*   _setVary();_

El método _setCache()_ puede utilizarse para establecer **la información de caché más usada** en sólo un método:

```
$response->setCache(array(
    'etag' => 'abcdef',
    'last_modified' => new \DateTime(),
    'max_age' => 600,
    's_maxage' => 600,
    'private' => false,
    'public' => true
    ));
```

Para comprobar si los validadores **Response** (_ETag_, _Last-Modified_) concuerdan con el **valor condicional especificado** en el **Request** del cliente, se usa el método _isNotModified()_:

```
if ($response->isNotModified($request)) {
    $response->send;
}
```

Si el **Response** no está modificado, establece el **código de estado** en **304 Not Modified** (la petición a la URL no ha sido modificada desde que fue requerida por última vez) y quita el contenido de la respuesta.

### 4. Redireccionar al usuario

Para **redireccionar al usuario a otra URL**, puedes usar la clase **RedirectResponse**:

```
use Symfony\Component\HttpFoundation\RedirectResponse:

$response = new RedirectResponse('http://example.com/');
```

### 5. Respuesta streaming

La clase **StreamedResponse** te permite transmitir de nuevo el **Response** al cliente. El contenido de respuesta se representa por un **PHP callable** en lugar de un _string_:

```
use Symfony\Component\HttpFoundation\StreamedResponse;

$response = new StreamedResponse();
$response->setCallback(function() {
    echo 'Hello world';
    flush();
    sleep(2);
    echo 'Hello world again!';
    flush();
});
$response->send();
```

La función _flush()_ no vacía el búfer. Si _ob_start()_ ha sido llamada antes o la opción **output_buffering** del _**php.ini**_ está activada, debes llamar a _ob_flush()_ antes que _flush()_.

Además, PHP no es la única forma de **almacenar datos de salida en el búfer**. Tu **servidor web** puede también basándose en su configuración. Si usa _**FastCGI**_, buffering ni siquiera se puede desactivar.

### 6. Servir archivos

Cuando se envía un archivo debes añadir el header **Content-Disposition** a tu respuesta. Crear este header para descargar archivos básicos es fácil, pero con nombres de archivo **non-ASCII** es más complejo. El método _makeDisposition()_ abstrae esta tarea complicada con una simple API:

```
use Symfony\Component\HttpFoundation\ResponseHeaderBag;

$d = $response->headers->makeDisposition(
    ResponseHeaderBag::DISPOSITION_ATTACHMENT,
    'foo.pdf'
    );

$response->headers->set('Content-Disposition', $d);
```

Alternativamente, con un archivo estático se puede usar una respuesta **BinaryFileResponse**:

```
use Symfony\Component\HttpFoundation\BinaryFileResponse;

$file = 'path/to/file.txt';
$response = new BinaryFileResponse($file);
```

**BinaryFileResponse** manejará automáticamente los headers **Range** y **If-Range** de la petición. También admite X-Sendfile (para [Nginx](http://wiki.nginx.org/XSendfile) o [Apache](https://tn123.org/mod_xsendfile/)). Para usarlo, necesitas determinar si el header **x-Sendfile-Type** debe aceptarse y llamar al método [_trusttXSendfileTypeHeader()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/BinaryFileResponse.html#method_trustXSendfileTypeHeader) si es así:

```
BinaryFileResponse::trustXSendfileTypeHeader();
```

Puedes todavía establecer el **Content-Type** del archivo enviado, o cambiar su **Content-Disposition**:

```
$response->headers->set('Content-Type', 'text/plain');
$response->setContentDisposition(
    ResponseHeaderBag::DISPOSITION_ATTACHMENT,
    'filename.txt'
);
```

Se puede **borrar el archivo** después de que el **request** es enviado con el método [_deleteFileAfterSend()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/BinaryFileResponse.html#method_deleteFileAfterSend), pero no funciona cuando el header _X-Sendfile_ está establecido.

Si se ha creado recientemente el archivo en el request actual, el archivo podría enviarse sin ningún contenido. Esto puede deberse a que el archivo cacheado devuelve cero por el tamaño del archivo. Para solucionar esto se puede llamar a _clearstatcache(false, $file)_ con la ruta al archivo binario.

### 7. Crear una respuesta JSON

Cualquier tipo de respuesta se puede crear con la clase **Response** estableciendo el contenido y headers deseados. Una respuesta JSON puede ser así:

```
use Symfony\Component\HttpFoundation\Response;

$response = new Response();
$response->setContent(json_encode(array(
    'data' => 123,
)));
$response->headers->set('Content-Type', 'application/json');
```

También existe una clase **JsonResponse** que lo hace más fácil:

```
use Symfony\Component\HttpFoundation\JsonResponse;

$response = new JsonResponse();
$response->setData(array(
    'data' => 123
));
```

Esto codifica tu array de datos a **JSON** y establece el header **Content-Type** a _application/json_. 

Para evitar el [JSON Hijacking XSSI](http://haacked.com/archive/2009/06/25/json-hijacking.aspx/), se debe pasar un array asociativo al **JsonResponse** y no un array indexado para que el resultado final sea un **objeto** (ej: {"object": "not inside an array"}) en lugar de un **array** (ej: [{"object": "inside an array"}]). Sólo los métodos que responden a **GET request** son vulnerables a estos ataques.

### 8. JSONP callback

Si usas **JSONP**, puedes establecer la función **callback** a la que se le pasarán los datos:

```
$response->setCallback('handleResponse');
```

En este caso, el header **Content-Type** será _text/javascript_ y el contenido de la respuesta será del tipo:

```
handleResponse({'data': 123});
```