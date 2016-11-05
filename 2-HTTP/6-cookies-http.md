Las cookies son como **archivos de texto que se guardan en el ordenador**. A petición de un **servidor web**, el navegador crea un archivo de este tipo. Una vez que ocurre esto el servidor puede leer y escribir contenido en este archivo. Puede parecer peligroso, pero existen una serie de restricciones para hacerlo lo más seguro posible:

*   **Los servidores web sólo pueden acceder a cookies establecidas a su propio dominio**. Este dominio es establecido por el navegador cuando el servidor crea una nueva cookie, y sólo puede ser un dominio o subdominio del servidor.
*   Según el **protocolo HTTP**, las cookies no pueden ser más grandes de **4096 Bytes (4KB)**.
*   Hay un **límite de cookies por dominio**. Depende del navegador, pero suelen ser 20 cookies.
*   También hay un **límite en el número total de cookies en el disco duro del cliente**. Suele ser de unas 300 cookies. Cuando se llega a este número, una cookie antigua se elimina antes de crear la nueva.

Las cookies tienen una **fecha de expiración**. Esta fecha permite al navegador eliminar cookies antiguas cuando ya no son requeridas por el servidor web. Si la fecha de expiración está vacía, la cookie se eliminará cuando finalice la conexión con el servidor. Esto ocurrirá cuando el usuario cierre la ventana o pestaña del sitio web, o directamente cierre el navegador. Estas cookies, normalmente denominadas _**session cookies**_, son usadas principalmente para guardar ajustes temporales.

#### **Protocolo HTTP**

Las cookies se transmiten a través del **protocolo HTTP**. Este es el protocolo que utilizan los navegadores para **enviar y recibir archivos de un servidor**. Una vez que se ha solicitado una cookie, el servidor la envía cada vez que un nuevo objeto en el sitio web es empleado por el navegador. Un ejemplo de un servidor solicitando una nueva cookie:
```
Set-Cookie: Name=content data; expires=Fri, 31-Dec-2010 23:59:59 GMT; path=/; domain=.ejemplo.com
```

*   **Set-Cookie**. Le hace saber al navegador que el servidor quiere crear una nueva cookie.
*   **Name**. Nombre de la cookie. Cada cookie de un dominio debe tener un nombre diferente, para que el navegador pueda mantener todas las cookies separadas. Después del nombre viene el propio contenido de la cookie (_content data_). Estos datos sólo pueden ser strings o números, y como máximo de 4KB.
*   **expires**. Indica la fecha de expiración. El formato es "**Wdy, DD-Mon-YYYY HH:MM:SS GMT**". El navegador elimina cookies con una fecha de expiración antigua.
*   **domain**. Es el dominio donde será activada la **cookie**. Si el dominio es _sub.ejemplo.com_, la cookie sólo será enviada al servidor de ese dominio, y si el dominio es _ejemplo.com_, la cookie se enviará a cualquier servidor de cualquiera de los subdominios de _ejemplo.com_ (incluyendo a él mismo).
*   **path**. Es la ruta del dominio a donde se envía la cookie. Si la ruta es /imagenes/, y el dominio es ejemplo.com, la cookie sólo se enviará al servidor si el navegador solicita un archivo de ejemplo.com/imagenes/. Si path es /, la cookie se enviará al servidor independientemente de la localización del archivo solicitado en el servidor.

### Crear y leer cookies

Las cookies **se pueden crear de varias formas** y con **diferentes lenguajes**.

Lo más importante a la hora de **crear una cookie en PHP** es que se debe establecer antes de enviar cualquier tipo de dato al navegador. Esto incluye cualquier tipo de etiqueta html, echo, print, etc. 

```
$nombre = 'usuario';
$valor = 'Homer';
// El tiempo de expiración es en 30 minutos. PHP traduce la fecha al formato adecuado
$expiracion = time() + 60 * 30;
// Ruta y dominio
$ruta = '/cookies/';
$dominio = "prueba.dev";
// Sólo envía la cookie con conexión HTTPs
$seguridad = false;
// Cookie disponible sólo para el protocolo HTTP
$solohttp = true;
setcookie($nombre, $valor, $expiracion, $ruta, $dominio, $seguridad, $solohttp);
echo "Cookie establecida";
```

La opción de _$seguridad_ fuerza a que la cookie sólo sea enviada si se ha establecido una conexión segura HTTPs. _$solohttp_ hace que la cookie sólo esté disponible a través de **http**, por lo que lenguajes del lado del servidor como JavaScript no pueden acceder a la cookie. Esto ayuda a prevenir [ataques XSS](http://diego.com.es/ataques-xss-cross-site-scripting-en-php), aunque si se quiere acceso a las cookies desde JavaScript se tiene que activar. Esto no significa que no se puedan enviar cookies a través de HTTPs.

Las cookies pueden verse con Firebug de Firefox o con las Developer Tools de Google Chrome.

Se observa que la cookie sólo aparece cuando se accede al directorio _/cookies/_. Si se va a un directorio padre la cookie no existe.

La cookie anterior puede ser escrita de forma mucho más reducida:

```
setcookie('usuario', 'Homer', time()+60*30, '/cookies/', 'prueba.dev', false, true);
```

Para leer las cookies existe un _array_ llamado **$_COOKIE**, que se utiliza para **extraer los valores de la cookie**. Su uso es muy sencillo:

```
$valorCookie = $_COOKIE['usuario'];
echo $valorCookie; // Devuelve: Homer
```

Esta variable de entorno puede utilizarse como cualquier otra, como [$_GET y $_POST](http://diego.com.es/get-y-post-en-php), y puede tratarse como una variable normal (una vez comprobada su existencia).

Si se quiere modificar la **fecha de expiración**, **path** o **dominio**, se ha de sobreescribir la cookie con _**setcookie()**_ con el mismo nombre que la cookie original. Si se establece una fecha de expiración en el pasado (por ejemplo _time()-30*60_), la cookie se eliminará.