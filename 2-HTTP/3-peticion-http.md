### Estructura de un HTTP request

Un HTTP request se compone de:

*   **Método**: GET, POST, PUT, etc. Indica que tipo de request es.
*   **Path**: la URL que se solicita, donde se encuentra el resource.
*   **Protocolo**: contiene HTTP y su versión, actualmente 1.1.
*   **Headers**. Son esquemas de _**key**_: _**value**_ que contienen **información sobre el HTTP request y el navegador**. Aquí también se encuentran los datos de las cookies. La mayoría de los headers son opcionales.
*   **Body**. Si se envía información al servidor a través de POST o PUT, ésta va en el body.

**Ejemplo de HTTP request**:
```
GET php.net HTTP/1.1 **Accept**: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
**Accept-Encoding**: gzip, deflate, sdch
**Accept-Language**: es-ES,es;q=0.8,en;q=0.6
**Cache-Control**: max-age=0
**Connection**: keep-alive
**Cookie**: COUNTRY=NA%2C122.16.430.651; LAST_LANG=es; LAST_NEWS=3847110839
**Host**: php.net
**If-Modified-Since**: Mon, 09 Nov 2015 11:50:11 GMT
**Upgrade-Insecure-Requests**: 1
**User-Agent**: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36
```

La primera línea es la **línea del request**, que contiene su información básica (método HTTP, URL y versión). Lo demás son headers HTTP.

### Headers de un HTTP request

Los headers pueden obtenerse con el array **$_SERVER** en PHP. La función _getallheaders() _devuelve todos los headers de vez.

Los headers más comunes en los HTTP requests son los siguientes:

*  Host
```
Host: php.net
```

Es el nombre del host, incluyendo dominio y subdominio si existe. En PHP se obtiene con **$_SERVER['HTTP_HOST']** o **$_SERVER['SERVER_NAME']**.

*  User-Agent
```
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.80 Safari/537.36
```

El User-Agent contiene información como el **nombre y versión del navegador** y del **sistema operativo** y el **idioma por defecto**. De esta forma los sitios web pueden saber información acerca de los sistemas de los visitantes. Pueden detectar si el usuario está visitando desde un móvil y redireccionarlo a una versión móvil más adecuada para bajas resoluciones. En PHP se encuentra bajo **$_SERVER['HTTP_USER_AGENT']**.

*  Accept-Language
```
Accept-Language: en-us, en;q=0.5
```

Este header muestra el lenguaje por defecto del usuario. Si el sitio web tiene diferentes versiones por idiomas, puede redireccionar al usuario. En PHP: **$_SERVER['HTTP_ACCEPT_LANGUAGE']**.

```
if(substr($_SERVER['HTTP_ACCEPT_LANGUAGE'], 0, 2) == 'es'){
    header('Location: http://es.ejemplo.com');
}
```

*  Accept-Encoding
```
Accept-Encoding: gzip, deflate, sdch
```

Formatos de codificación que soporta el navegador. El servidor web puede enviar el **HTML** resultante en un **formato comprimido**, lo que ahorra hasta un 80% de bandwidth y tiempo de carga. En PHP: **$_SERVER['HTTP_ACCEPT_ENCODING']**.

*  If-Modified-Since
```
If-Modified-Since: Mon, 9 Nov 2015 08:32:10 GMT
```

Si no se ha modificado el resource desde esa fecha, el servidor devolverá un código de respuesta 304 Not Modified, sin contenido, ya que el navegador cargará el contenido de la caché. En PHP: **$_SERVER['HTTP_IF_MODIFIED_SINCE']**.

```
// Comprobar si el explorador envía el header If-Modified-Since
if(isset($_SERVER['HTTP_IF_MODIFIED_SINCE'])){
    // Comprobar si el cache del navegador coincide con el modify time
    if($last_modify_time == strtotime($_SERVER['HTTP_IF_MODIFIED_SINCE'])){
        // Enviar un header 304, sin contenido
        header("HTTP/1.1 304 Not Modified");
        exit;
    }
}
```

También está el header Etag, que se asegura de que la caché es actual.

*  Cookie
```
Cookie: PHPSESSID=rqwe8f1ew8f1341fiu; usuario=homer
```

Envía las cookies guardadas en el navegador para ese dominio. En PHP: array **$_COOKIE**.

*  Referer

**Referer**. Contiene la url de referencia. Si un usuario hace click en un enlace, en la página de destino aparecerá como referer la anterior. En PHP: **$_SERVER['HTTP_REFERER']**.

```
if(isset($_SERVER['HTTP_REFERER'])){
    $url = parse_url($_SERVER['HTTP_REFERER']);
    // Comprobar si el visitante viene de google
    if($url['host'] == 'www.google.com'){
        parse_str($url['query'], $vars);
        echo "Has buscado estas palabras: " . $vars['q'];
    }
}
```

*  Authorization

Cuando un sitio web solicita autorización, el navegador abre una ventana de login. Cuando insertas los datos de entrada, el navegador envía otro request, pero esta vez contiene:
```
Authorization: Basic bXl1c2VyOm15cGFzcw==
```

El dato incluído está codificado en **base 64**. La función base64_decode('bXl1aefi128djGFzcw==') devuelve 'myuser:mypass'.

En PHP: **$_SERVER['PHP_AUTH_USER']** y **$_SERVER['PHP_AUTH_PW']**.