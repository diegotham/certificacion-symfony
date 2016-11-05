### Estructura de un HTTP response

Una vez que el navegador envía el **HTTP request**, el servidor responde con un **HTTP response**, compuesto por:

*   **Protocolo**. Contiene HTTP y su versión, actualmente 1.1.
*   **Status code**. El **código de respuesta**, por ejemplo: **200 OK**, que significa que el GET request ha sido satisfactorio y el servidor devolverá los contenidos del documento solicitado. Otro ejemplo es **404 Not Found**, el servidor no ha encontrado el resource solicitado.
*   **Headers**. Contienen información sobre el **software del servidor**, cuando se modificó por última vez el _resource_ solicitado, el **mime type**, etc. De nuevo la mayoría son opcionales.
*   **Body**. Si el servidor devuelve información que no sean _headers_ ésta va en el _**body**_.

**Ejemplo de HTTP response**:
```
HTTP/1.1 200 OK
**Content-Encoding**: gzip
**Content-language**: en
**Content-Length**: 5107
**Content-Type**: text/html; charset=utf-8
**Date**: Mon, 09 Nov 2015 12:08:48 GMT
**Last-Modified**: Mon, 09 Nov 2015 11:50:11 GMT
**Link**: <http://php.net/index>; rel=shorturl
**Server**: nginx/1.6.2
**Vary**: Accept-Encoding
**X-Frame-Options**: SAMEORIGIN
**X-Powered-By**: PHP/5.6.13-0+deb8u1

<!DOCTYPE html...
...
```

La primera línea es la **barra de estado** (versión HTTP y código de respuesta), seguido de los HTTP headers, hasta una línea vacía. Bajo la línea vacía se encuentra el contenido (en este caso contenido html).

### Headers de un HTTP response

En PHP se pueden establecer los códigos de respuesta con la función _[header()](http://php.net/manual/en/function.header.php)_. PHP ya envía algunos headers automáticamente como para cargar el contenido o establecer cookies. Se pueden ver los headers enviados, o que serán enviados, con la función _[headers_list()](http://www.php.net/manual/en/function.headers-list.php)_. Se puede saber si los headers ya se han enviado con _[headers_sent()](http://www.php.net/manual/en/function.headers-sent.php)_. 

*  Cache-Control

Campo usado para especificar las directivas que se deben cumplir por los mecanismos de caché durante la cadena request/response. Estos mecanismos de caché incluyen gateways y proxies que el ISP puede estar utilizando.
```
Cache-Control: max-age=3600, public
```

"public" significa que la respuesta puede ser cacheada por cualquiera. "max-age" indica por cuantos segundos es válida. Permitir que el sitio web sea cacheado reduce el consumo de memoria en el servidor y reduce los tiempos de carga.

Se puede evitar el cacheo con "no-cache":
```
Cache-Control: no-cache
```

*  Content-Type

Indica el **mime-type** del documento. El navegador decide entonces como interpretar los contenidos. Una página html (o PHP con output html):
```
Content-Type: text/html; charset=UTF-8
```

"text" es el tipo y "html" es el subtipo del documento. Para una imagen tipo gif:
```
Content-Type: image/gif
```

El navegador puede decidir si usar una aplicación externa o una extensión de navegador basándose en el mime-type. Por ejemplo lo siguiente puede cargarse con Adobe Reader:
```
Content-Type: application/pdf
```

Cuando se carga directamente, **Apache** normalmente puede detectar el **mime-type** de un documento y enviar el header apropiado. Los navegadores tienen detección automática de mime-types, por si los headers están mal o no están presentes.

En **PHP** se puede utilizar la función _[finfo_file()](http://www.php.net/manual/en/function.finfo-file.php)_ para detectar el mime-type de un archivo.

*  Content-Disposition

Este header indica al navegador que abra una caja de descarga de archivos, en lugar de analizar el contenido. 
```
Content-Disposition: attachment; filename="descargar.zip"
```

Este header anterior debe ir acompañado de este otro:
```
Content-Type: application/zip
```

*  Content-Length

Cuando el contenido se va a enviar al navegador, el servidor puede indicar el tamaño en bytes:
```
Content-Length: 123245
```

Esto es especialmente útil para la descarga de archivos, así el navegador puede calcular el progreso de la descarga.

*  Etag

Es otro header que se usa para caching.
```
Etag: "pub1212441;gz"
```

El servidor web puede enviar este header con cada documento que envía. El valor puede estar basado en la **última fecha de modificación**, el **tamaño del archivo**, etc. El navegador guarda entonces este valor y cachea el documento. La proxima vez que el navegador solicita el mismo archivo, envía esto en el **HTTP request**:
```
If-None-Match: "pub1212441;gz"
```

Si el valor **eTag** del documento concuerda, el servidor enviará un código 304 en lugar de 200, sin contenido. El navegador cargará los contenidos de su caché.

*  Last-Modified

Indica la última fecha de modificación del documento en formato GMT. 
```
Last-Modified: Mon, 09 Nov 2015 11:50:11 GMT
```

En PHP podemos establecerlo así:

```
$modificarTiempo = filemtime($archivo);
header("Last-Modified: ".gmdate("D, d M Y H:i:s", $modificarTiempo));
```

*  Location

Header utilizado para las redirecciones. Si el código de respuesta es 301 o 302, el servidor debe también enviar este header.

En PHP se puede redirigir a un usuario así:

```
header('Location: http://www.google.com');
```

Esta será una **redirección 302**. Si queremos que sea **301**:

```
header('Location: http://www.google.com', 301);
```

*  Set-Cookie

Cuando un sitio web quiere establecer o actualizar una cookie en tu navegador, utilizará este header.
```
Set-Cookie: skin=noskin; path=/; domain=.amazon.com; expires=Sun, 21-Nov-2015 14:22:22 GMT
Set-Cookie: session-id=820-1736418-8162394; path=/; domain=.amazon.
```

Cada cookie se envía en un header separado. En PHP las cookies se establecen con la función _setcookie()_.

Si no se especifica una fecha de expiración, la cookie se eliminará cuando se cierre la ventana del navegador.

*  WWW-Authenticate

Un sitio web puede enviar este header para **identificar al usuario a través de HTTP**. Cuando el navegador ve este header, abrirá una ventana de login. 
```
WWW-Authenticate: Basic realm="Restricted Area"
```

En PHP se puede utilizar el siguiente script:

```
if(!isset($_SERVER['PHP_AUTH_USER'])){
    header('WWW-Authenticate: Basic realm="My Realm"');
    header('HTTP/1.0 401 Unauthorized');
    echo 'Texto a enviar si el usuario cancela';
    exit;
} else {
    echo "<p>Hola {$_SERVER['PHP_AUTH_USER']}.</p>";
    echo "<p>Has introducido {$_SERVER['PHP_AUTH_PW']} como tu contraseña</p>";
}
```

*  Content-Encoding

Header enviado cuando el contenido está comprimido.
```
Content-Encoding: gzip
```
