HTTP (**Hypertext Transfer Protocol** o [Protocolo de Transferencia de Hipertexto](http://es.wikipedia.org/wiki/Hypertext_Transfer_Protocol)) es un **lenguaje de texto** o **protocolo** que permite a dos ordenadores comunicarse entre ellos siguiendo un esquema **petición-respuesta** entre un **cliente** (_**user agent**_) y un **servidor**. 

El **cliente** lanza una petición (_**request**_) al **servidor** donde esté alojada una **página web** o **app** (normalmente mediante **navegador web** o **smartphone app**), el **servidor** prepara una respuesta (_**response**_) y la devuelve al cliente.

**Indice de contenido**

1.  [La petición del cliente](#LaPeticionDelCliente)
2.  [La respuesta del servidor](#LaRespuestaDelServidor)
3.  [Peticiones y respuestas en PHP](#PeticionesYRespuestasEnPHP)
4.  [Peticiones y respuestas en Symfony](#PeticionesYRespuestasEnSymfony)
5.  [De la petición a la respuesta](#DeLaPeticionALaRespuesta)
6.  [Construye tu aplicación, no tus herramientas](#ConstruyeTuAplicacionNoTusHerramientas)

### <a id="LaPeticionDelCliente" name="LaPeticionDelCliente"></a>1. La petición del cliente

El _**request**_ es un mensaje de texto creado por el **cliente** en un formato especial conocido como **HTTP**. El cliente envía ese request al servidor y espera una respuesta:

```
GET / HTTP/1.1
Host: diego.com.es
Accept: text/html
User-Agent: Mozilla/5.0 (Macintosh)
```

Este mensaje comunica lo necesario sobre los recursos que está solicitando el cliente. La primera línea es la más importante, que contiene la **URI** y el **método HTTP**. La **URI** es la **dirección única** o **localización** que identifica al recurso que está solicitando el cliente. El **método GET** define que quieres hacer con ese recurso. Los métodos HTTP son los verbos del _**request**_ y definen las cosas que puedes hacer con los recursos:

*   GET - Devuelve recursos del servidor
*   POST - Crea un recurso en el servidor
*   PUT - Actualiza el recurso en el servidor
*   DELETE - Elimina un recurso del servidor

Por ejemplo para borrar una entrada de blog, la primera línea del _**request**_ es:

```
DELETE /blog/{id} HTTP/1.1
```

Además de la primera línea, la **petición HTTP** contiene otras línas denominadas **request headers**. Los _**headers**_ contienen información como el **Host**, los formatos de respuesta que acepta el cliente **Accept**, o la aplicación que el cliente usa para hacer la petición, **User-Agent**. 

### <a id="LaRespuestaDelServidor" name="LaRespuestaDelServidor"></a>2. La respuesta del servidor

El servidor recibe la petición, e inmediatamente sabe que recurso está solicitando el cliente por la URI, además que qué quiere hacer con ese recurso (mediante el método HTTP). La respuesta es algo así:

```
HTTP/1.1 200 OK
Date: Sat, 02 Apr 2011 21:05:05 GMT
Server: lighttpd/1.4.19
Content-Type: text/html

<html>
  <!-- ... HTML de diego.com.es -->
</html>
```

De la respuesta HTTP, el recurso solicitado es el contenido HTML.

La primera línea contiene el [código de estado](http://es.wikipedia.org/wiki/Anexo:C%C3%B3digos_de_estado_HTTP), en este caso **200**, que indica que la respuesta ha sido satisfactoria. Este código indica si ha habido algún error en la respuesta, si ha sido satisfactoria o si el cliente ha de hacer algo (como una redirección).

Al igual que en la **petición**, la **respuesta** también contiene piezas adicionales de información en **HTTP headers**, como **Content-Type**. El recurso solicitado se puede devolver en diferentes formatos: **HTML**, **XML**, o **JSON**, y el header Content-Type usa [Internet Media Types](http://en.wikipedia.org/wiki/Internet_media_type#List_of_common_media_types) como _text/html_ para informar al cliente del formato devuelto.

Independientemente del lenguaje de programación a usar o el tipo de aplicación que se vaya a construir (web, móvil, JSON API) el objetivo final de una aplicación es entender la petición y crear una respuesta adecuada. 

### <a id="PeticionesYRespuestasEnPHP" name="PeticionesYRespuestasEnPHP"></a>3. Peticiones y respuestas en PHP

```
$uri = $_SERVER['REQUEST_URI'];
$foo = $_GET['foo'];

header('Content-Type: text/html');
echo 'La URI solicitada es: '.$uri;
echo 'El valor del parámetro "foo" es: '.$foo;
```

Esta pequeña aplicación coge información de la **petición HTTP** y la usa para crear una **respuesta HTTP**. En lugar de analizar directamente el mensaje de petición, **PHP** tiene variables **superglobals** como _$_SERVER_ y _$_GET_ que contienen toda la **información de la petición**. Igualmente, en lugar de devolver directamente la respuesta HTTP, mediante la función _header()_ se crean headers de respuesta y se devuelve un contenido de mensaje de respuesta. PHP creará una respuesta HTTP y la devolverá al cliente:

```
HTTP/1.1 200 OK
Date: Sat, 03 Apr 2011 02:14:33 GMT
Server: Apache/2.2.17 (Unix)
Content-Type: text/html

La URI solicitada es: /testing?foo=symfony
El valor del parámetro "foo" es: symfony
```

### <a id="PeticionesYRespuestasEnSymfony" name="PeticionesYRespuestasEnSymfony"></a>4. Peticiones y respuestas en Symfony

**Symfony** proporciona dos clases que permiten interactuar con la petición y la respuesta de forma sencilla. La clase **Request** es una representación orientada a objetos del mensaje **HTTP request**:

```
use Symfony\Component\HttpFoundation\Request;

$request = Request::createFromGlobals();

// La URI solitada (ej : /contacto)
$request->getPathInfo();

// Devuelve las variables GET y POST respectivamente
$request->query->get('foo');
$request->request->get('bar', 'valor por defecto si bar no existe');

// Devuelve las variables SERVER
$request->server->get('HTTP_HOST');

// Devuelve una instancia de UploadedFile identificada como foo
$request->files->get('foo');

// Devuelte un valor COOKIE
$request->cookies->get('PHPSESSID');

// Devuelte un header request
$request->headers->get('host');
$request->headers->get('content_type');

$request->getMethod();    // GET, POST, PUT, DELETE, HEAD
$request->getLanguages(); // Un array con lenguajes que acepta el cliente
```

Además, la clase **Request** realiza otro tipo de acciones, por ejemplo mediante el método **_isSecure()_** comprueba los tres diferentes valores en PHP que indican si el usuario se está conectando mediante conexión segura (como **HTTPS**).

Las variables $_GET y $_POST son accesibles a través de las propiedades _**query**_ y _**request**_, respectivamente. Cada uno de estos objetos es un objeto ParameterBag, que tiene métodos como _get()_, _has()_, _all()_ y más. Las propiedades usadas en el ejemplo anterior son instancias de ParameterBag.

La clase **Request** también tiene una **propiedad public** llamada _**attributes**_, que guarda información relacionada con cómo la aplicación funciona internamente. Para el **Framework Symfony**, _**attributes**_ guarda la información devuelta por la ruta, como _**_controller**_, el _**id**_ (si se ha insertado un _wildcard_), o el nombre de la ruta _**_route**_. La propiedad _**attributes**_ sirve para preparar y guardar información sobre la petición. 

Symfony también proporciona la clase **Response**, una representación **PHP** de un **mensaje de respuesta HTTP**. Esto permite a una aplicación usar una interfaz orientada a objetos para construir la respuesta que ha de ser devuelta al cliente:

```
use Symfony\Component\HttpFoundation\Response;

$response = new Response();

$response->setContent('<html><body><h1>Hello world!</h1></body></html>');
$response->setStatusCode(200);
$response->headers->set('Content-Type', 'text/html');

// Imprime los headers HTTP seguidos del contenido
$response->send();
```

Las clases **Request** y **Response** forman parte de un componente de **Symfony** llamado **HttpFoundation**, puede usarse independientemente del framework y proporciona clases para manejar sesiones y subidas de archivos.

### <a id="DeLaPeticionALaRespuesta" name="DeLaPeticionALaRespuesta"></a>5. De la petición a la respuesta

Al igual que el **HTTP**, los objetos **Request** y **Response** no son muy complejos. La parte complicada va entre medio: enviar emails, crear formularios, guardar datos en una base de datos, seguridad... Symfony permite facilitar todas las tareas.

Para una explicación más detallada del proceso, accede a este artículo: [de la petición a la respuesta en Symfony](http://diego.com.es/de-la-peticion-a-la-respuesta-en-symfony).

#### El Front Controller

Anteriormente las aplicaciones se construían de forma que cada página tenía su propio archivo: index.php, blog.php, contacto.php... Lo que tiene ciertas limitaciones.

Una solución más eficaz es usar un **controlador frontal**: un archivo **PHP** que maneja cada **request** que se le solicita a una aplicación:

*   /index.php ejecuta _index.php_
*   /index.php/contacto ejecuta _index.php_
*   /index.php/blog ejecuta _index.php_

Con **Apache mod_rewrite** o su equivalente en otros servidores web, las **URLs** pueden ser amigables.

Ahora en lugar de que cada URL ejecute archivos PHP diferentes, el **front controller** se ejecuta siempre, y las rutas a las diferentes **URLs** se hacen internamente. Casi todas las aplicaciones modernas lo hacen de esta forma, incluido **Wordpress**. 

Dentro del **front controller** se ha de averiguar que código ha de ejecutarse y que contenido se ha de devolver. Para averiguar esto se necesita comprobar la **URI** de la petición y ejecutar diferentes partes del código dependiendo de ese valor. Solucionar esto puede resultar bastante complejo, y es ahí donde entra **Symfony**.

#### El flujo de las aplicaciones Symfony

![Funcionamiento interno de Symfony](http://symfony.com/doc/current/_images/request-flow.png)

Las **peticiones** se interpretan por el **routing** y se pasan a **funciones de los controladores** que devuelven **objetos respuesta**.

Cada **página** de un sitio se define en un archivo **routing** de configuración que asigna diferentes URLs a diferentes funciones PHP. El trabajo de cada **función PHP en un controlador** es utilizar la información del **request** para crear el objeto **respuesta**. En otras palabras, el **controlador** es donde va tu código: interpretas la petición y creas la respuesta. 

#### Una petición Symfony en acción

Sin entrar en muchos detalles, este sería el proceso en una página de /contacto. Primero se añade la ruta en el archivo de configuración routing:

```
# app/config/routing.yml
contact:
    path:     /contacto
    defaults: { _controller: AppBundle:Main:contacto }
```

Cuando alguien visita la página _/contacto_, se activa la ruta y se ejecuta el controlador. **AppBundle:Main:contacto** es una forma abreviada de apuntar al método **contactoAction** dentro de la clase llamada **MainController**:

```
// src/AppBundle/Controller/MainController.php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;

class MainController
{
    public function contactAction()
    {
        return new Response('<h1>Contact us!</h1>');
    }
}
```

Este ejemplo tan sólo crea un **objeto respuesta** con un **header html**, aunque lo normal es separar el código de presentación en html en otros archivos, lo que se verá en el capítulo de **controladores**.

### <a id="ConstruyeTuAplicacionNoTusHerramientas" name="ConstruyeTuAplicacionNoTusHerramientas"></a>6. Construye tu aplicación, no tus herramientas:

Cuando las aplicaciones crecen se hace más difícil mantener el código bien organizado. Complejas tareas se repiten, como añadir datos a la base de datos, mostrar y reusar plantillas, manejar formularios, enviar emails, validaciones y seguridad.

**Symfony** proporciona un **framework** con **herramientas** que permiten construir la aplicación, no las herramientas. Es una colección de más de 20 librerías independientes que **se pueden usar dentro de cualquier proyecto PHP**. Estas librerías, llamadas **Symfony Components**, contienen utilidades para muchas situaciones diferentes. Las más importantes:

*   [HttpFoundation](http://symfony.com/doc/current/components/http_foundation/introduction.html): contiene las clases Request y Response, así como otras para manejar sesiones y subidas de archivos.
*   [Routing](http://symfony.com/doc/current/components/routing/introduction.html): Sistema de enrutamiento que permite asociar URIs con comportamientos. 
*   [Form](http://symfony.com/doc/current/components/form/introduction.html): framework para crear formularios y manejarlos.
*   [Validator](https://github.com/symfony/Validator): Sistema para crear reglas de validación en los formularios.
*   [Templating](http://symfony.com/doc/current/components/templating/introduction.html): herramienta para mostrar plantillas, herencia entre plantillas, y otras tareas comunes.
*   [Security](http://symfony.com/doc/current/components/security/introduction.html): librería para manejar la seguridad de una aplicación.
*   [Translation](http://symfony.com/doc/current/components/translation/introduction.html): framework para traducir _strings_ en una aplicación.