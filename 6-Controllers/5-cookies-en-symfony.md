Para **establecer cookies en Symfony** simplemente se ha de utilizar el atributo público _headers_:

```
use Symfony\Component\HttpFoundation\Cookie;

$response->headers->setCookie(new Cookie('foo', 'bar'));
```

El método _setCookie()_ toma una instancia de [Cookie](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Cookie.html) como argumento:
```
public __construct (string $name, string $value = null, int|string|DateTime $expire, string $path = '/', string $domain = null, bool $secure = false, bool $httpOnly = true)
```

Sus parámetros son:

| | |
| -------- | -------- |
| _string_ $name | Nombre de la cookie |
| _string_ $value | Valor de la cookie |
| _int_|_string_|_DateTime_ $expire | La hora a la que expira la cookie |
| _string_ $path | El directorio en el servidor en el cual estará disponible la cookie |
| _string_ $domain | El dominio en el que la cookie estará disponible |
| _bool_ $secure | Si la cookie se ha de transmitir vía HTTPS |
| _bool_ $httpOnly | Si la cookie será accesible sólo a través del protocolo HTTP |

Ejemplo sencillo de **establecer dos cookies y luego mostrarlas con Twig:**

```
// src/AppBundle/Controller/DefaultController.php
class DefaultController extends Controller
{
    /**
     * @Route("/set", name="setcookies")
     */
    public function setCookiesAction ()
    {
        $response = $this->render('setcookies.html.twig');

        $response->headers->setCookie(new Cookie('Peter', 'Griffin', time() + 3600));
        $response->headers->setCookie(new Cookie('Homer', 'Simpson', time() + 3600));

        return $response;
    }
```

El controlador devuelve una respuesta en una **template twig**, que en este caso es _setcookies.html.twig_ y puede estar vacía, o mostrar un mensaje como "Las cookies se han establecido".

```
/**
 * @Route("/get", name="getcookies")
 */
public function getCookiesAction (Request $request)
{
    $cookies = $request->cookies;
    $response = $this->render('getcookies.html.twig', array(
        'cookies' => $cookies
    ));

    return $response;
}
```

Ahora hemos empleado el [type hinting](http://diego.com.es/type-hinting-en-php) con la clase **Request** para obtener las cookies de los headers. Después simplemente renderizamos la template _getcookies.html.twig_, y le pasamos el array de cookies _$cookies_. En la template ahora podemos poner un listado de las cookies:

```
<ul>
    {% for key, value in cookies %}
        <li>{{ key }} - {{ value }}</li>
    {% endfor %}
</ul>
```

Otras **funciones disponibles para cookies**:

*   _clearCookie(). _Limpia una cookie del navegador (_$response->headers->clearCookie('Homer')_).
*   _getCookies()_. Devuelve un array con todas las cookies.
*   _removeCookie()_. Elimina una cookie del array, pero no la quita del navegador (_$response->headers->removeCookie('Peter')_).

Lo que ocurre en _clearCookie()_ es que **Symfony** envía un header _set-Cookie_ con el valor "deleted" y con fecha de expiración en 1970 (ya caducada):
```
Homer=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; Max-Age=0; path=/; httponly
```

En cambio _removeCookie()_ no envía ningún header _set-Cookie_ al navegador. En nuestro ejemplo esta función no evitaría que se siguiera mostrando la cookie Homer, ya que sigue estando en el navegador.