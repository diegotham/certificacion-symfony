**Indice de contenido**

| | |
| -------- | -------- |
| 1. Introducción | 4. Establecer una URL a un controller |
| 2. Ciclo request-response | 5. Parámetros de routes como argumentos de controllers |
| 3. Ejemplo de controller | 6. El request como argumento del controller |

### 1. Introducción

Un **controller** es un [callable PHP](http://php.net/manual/es/language.types.callable.php) creado que recibe la información del **HTTP request** y crea y devuelve una **HTTP response** (devuelto como un **objeto Response**). La respuesta podría ser una **página HTML**, un **documento XML**, un **array serializado JSON**, una imagen, una redirección, un **error 404** o cualquier otra cosa. El controller contiene cualquier lógica arbitraria que tu aplicación necesita para renderizar el contenido de una página.

Un ejemplo sencillo de controller que imprime "Hola mundo":

```
use Symfony\Component\HttpFoundation\Response;

public function helloAction()
{
    return new Response('Hola mundo');
}
```

El **objetivo** de un controller es siempre el mismo: **crear y devolver un objeto Response**. En el proceso puedes leer información del request, cargar una base de datos, enviar un email, o establecer información de la sesión de usuario. Pero en todos los casos el controller eventualmente devolverá un objeto Response, que se enviará al cliente. 

Ejemplo de un diferentes procesos utilizando un controller:

*   El controller A prepara un **objeto Response** representando el contenido de la página principal del sitio.
*   El controller B lee el parámetro _slug_ del request para cargar una entrada de blog de la base de datos y crea un objeto Response mostrando la entrada. Si el _slug_ no se encuetnra en la base de datos, crea y devuelve un objeto Response con un [código de estado](http://diego.com.es/codigos-de-estado-http) **404**.
*   El controller C administra la sumisión de un **formulario de contacto**. Lee la información del formulario del request, guarda la información de contacto en la base de datos y envía un email al administrador con la información. Finalmente, crea un objeto Response y redirige al navegador a la página de "Gracias por contactarnos".

### 2\. Ciclo request-response

Cada request administrado por un proyecto **Symfony** pasa por el mismo ciclo:

1.  Cada request es manejado por un sólo **front controller** (app.php o app_dev.php) que arranca la aplicación.
2.  El **Router** lee la información del request (por ejemplo la URI), encuentra una route que coincide con la información y lee el parámetro **_controller** del route.
3.  El controller de la route se ejecuta y el código de dentro del controlador crea y devuelve un objeto **Response**.
4.  Los **headers HTTP** y el contenido del objeto Response de devuelven al cliente.

Crear una página es tan fácil como crear un controller y hacer que la route especifique una URL a ese controller. 

### 3\. Ejemplo de controller

Un controller puede ser cualquier **PHP callable** (una función, un método de un objeto o un Closure), pero normalmente se crea un método dentro de una clase controller. A los controllers también se les llama actions. 

```
// src/AppBundle/Controller/HelloController.php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;

class HelloController
{
    public function indexAction($name)
    {
        return new Response('<html><body>Hola'.$name.'</body></html>')
    }
}
```

Conviene aclarar que el controller es el método indexAction, que está dentro de una _controller class_ (**HelloController**). Una clase controller es sólo una forma recomendable de agrupar una serie de controllers/actions. Normalmente la clase controller agrupará varios controllers (por ejemplo: updateAction, deleteAction, etc).

El esquema del código anterior es:

1.  **línea 2**. Symfony utiliza los namespaces de PHP y añade un namespace a la clase controller.
2.  **línea 4**. De nuevo emplea los namespaces: la palabra _use_ importa la clase Response, que el controller debe devolver.
3.  **línea 6**. El nombre de la clase es la concatenación de un nombre para la clase controller (_Hello_) y la palabra **Controller**. Es una convención que proporciona consistencia a los controllers y les permite ser referenciados sólo con la primera parte del nombre (_Hello_) en la configuración routing.
4.  **línea 8**. Cada _action_ de una clase controller tiene el sufijo **Action** y es referenciada en la configuración routing por el nombre de la acción (_index_). En la siguiente sección vamos a crear una route que establece una URI a esta _action_. Aprenderás como los placeholders ({_name_}) de las routes se convierten en argumentos en el método action (_$name_).
5.  **línea 10**. El controller crea un devuelve un objeto **Response**.

### 4. Establecer una URL a un controller

El controller devuelve una simple **página HTML**. Para ver esta página en el navegador, necesitas crear una route, que establece una URL específica para el controller. En este ejemplo vamos a emplear las anotaciones, que es la forma que recomienda **Symfony** para mapear las routes:

```
use Symfony\Component\HttpFoundation\Response;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class HelloController
{
    /**
     * @Route("/hello/{name}", name="hello")
     */
    public function indexAction($name)
    {
        return new Response('<html><body>Hello '.$name.'!</body></html>');
    }
}
```

Ahora puedes ir a _/hello/marta_ (por ejemplo, _http://localhost:8000/hello/marta_ si empleas el [built-in server](http://symfony.com/doc/current/cookbook/web_server/built_in.html)) y **Symfony** ejecutará el controller _HelloController:indexAction()_ y pasará marta como la variable _$name_. Crear una página simplemente significa crear un método controller y asociar una ruta.

### 5\. Parámetros de routes como argumentos de controllers

Ya sabemos que la route apunta al método _HelloController::indexAction()_ que está dentro del bundle AppBundle, pero hay que destacar el argumento que se le pasa a ese método:

```
// src/AppBundle/Controller/HelloController.php
// ...
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

/**
 * @Route("/hello/{name}", name="hello")
 */
public function indexAction($name)
{
    // ...
}
```

El controller tiene un argumento, $name, que corresponde con el parámetro {_name_} de la route (marta, si vas a _/hello/marta_). Cuando ejecutas el controller, **Symfony** enlaza cada argumento con un parámetro de la route. Así el valor de {_name_} se pasa a $name. El siguiente ejemplo muestra dos argumentos:

```
// src/AppBundle/Controller/HelloController.php
// ...

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class HelloController
{
    /**
     * @Route("/hello/{firstName}/{lastName}", name="hello")
     */
    public function indexAction($firstName, $lastName)
    {
        // ...
    }
}
```

Establecer parámetros de routes a argumentos de controllers es fácil y flexible, pero hay que tener en cuentas las siguientes pautas:

*   **El orden de los argumentos del controller no importa**. Symfony enlaza el nombre de los parámetros de la route con los nombres de variables del controller. Los argumentos del controller podrían reordenarse y funcionar de la misma forma.
*   **Cada argumento controller requerido debe coincidir con un parámetro de routing**, si no existe, devolverá un **RuntimeException**. Puedes en cambio marcar un argumento como opcional  y no lanzará ningún error:

   
```
    public function indexAction($firstName, $lastName, $foo = 'bar')
    {
        // ...
    }
   
```

*   **No todos los parámetros routing necesitan ser argumentos de tu controller**. Si por ejemplo el lastName no es importante en tu controller, puedes omitirlo:

   
```
    public function indexAction($firstName)
    {
        // ...
    }
   
```

Cada route tiene un parámetro especial **_route**, el cual es igual al nombre de la route con la que ha coincidido (en este caso "_hello_"). Aunque no suele ser de mucha utilidad, también está disponible como argumento de controller. Puedes también [pasar otras variables de tu route a los argumentos de tu controller](http://symfony.com/doc/current/cookbook/routing/extra_information.html). 

### 6\. El request como argumento del controller

¿Cómo podemos leer parámetros query, leer un request header o tener acceso a un archivo subido? Toda esta información se guarda en el **objeto Request de Symfony**. Para meterlo en tu controller, simplemente añádelo como argumento y haz type hinting con la clase Request:

```
use Symfony\Component\HttpFoundation\Request;

public function indexAction($firstName, $lastName, Request $request)
{
    $page = $request->query->get('page', 1);

    // ...
}
```