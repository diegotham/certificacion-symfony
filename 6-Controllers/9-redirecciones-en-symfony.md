Para realizar una redirección en Symfony se pueden emplear varios métodos:

### RedirectController

La clase **RedirectController** implementa **ContainerAwareInterface**, y redirige un request a otra URL. Para ello emplea dos métodos diferentes: 

*   _redirectAction()_. Redirige a otra route con el nombre dado.
```
public Response redirectAction (Request $request, string $route, bool $permanent = false, bool|array $ignoreAttributes = false)
```

El **código de estado de respuesta** es **302** y el parámetro _$permanent_ es false, y **301** si la redirección es permanente.

En caso de que el nombre de _$route_ esté vacío el código de estado será **404** cuando _$permanent_ sea falso y **410** si es verdadero. Se produce una [HttpException](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/Exception/HttpException.html) en este caso.

Devuelve una instancia [Response](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Response.html).

```
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\RedirectController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class RedController extends RedirectController
{
    /**
     * @Route("/myred", name="myred" )
     */
    public function myRedirectAction(Request $request)
    {
        return $this->redirectAction($request, 'homepage');
    }
}
```

*   _urlRedirectAction()_. Redirecciona a una URL.
```
public **Response urlRedirectAction (Request $request, string $path, bool $permanent = false, string|null $scheme = null, int|null $httpPort = null, int|null $httpsPort = null)
```

El código de respuesta es **302** si el parámetro _$permanent_ es false (por defecto), y **301** si es true.

En caso de que _$path_ quede vacío, el **código de estado** será **404** cuando _$permanent_ sea false y **410** cuando sea verdadero. Se produce una HttpException en este caso.

Devuelve un objeto **Response**.

Misma situación que la anterior, pero esta vez facilitamos el **absolute path** o la **URL**:

```
/**
 * @Route("/myred", name="myred" )
 */
public function myRedirectAction(Request $request)
{
    return $this->urlRedirectAction($request, '/');
}
```

### RedirectResponse

La clase [RedirectResponse](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/RedirectResponse.html) extiende a la clase **Response**. Representa una respuesta HTTP realizando una redirección.
```
public __construct (string $url, int $status = 302, array $headers = array())
```

Crea una respuesta redirect de forma que respeta las conveniencias definidas para un **código de estado** de redirección.

```
/**
 * @Route("/myred", name="myred" )
 */
public function myRedirectAction()
{
    return new RedirectResponse('/');
}
```

También dispone del método setTargetUrl:
```
public RedirectResponse setTargetUrl (string $url)

```

Ejemplo:

```
/**
 * @Route("/myred/{decision}", name="myred" )
 */
public function myRedirectAction($decision)
{
    // Redirección por defecto ($decision = no)
    $redireccion = new RedirectResponse('/');

    // ¿Redireccionamos a Google?
    if ($decision == "si") {
        $redireccion->setTargetUrl('http://www.google.com');
    }

    return $redireccion;
}
```

También podríamos utilizar el métodos shortcut _redirectToRoute()_, que equivale a un **RedirectResponse**.

```
return new RedirectResponse($this->generateUrl('homepage'));
// es igual que
return $this->redirectToRoute('homepage');
```

O con _redirect()_ :

```
return $this->redirect($this->generateUrl('homepage'));
```

### Redirección interna

No es muy frecuente, pero también puedes reenviar a otro controller internamente con el método _forward()_. En lugar de redireccionar el navegador del usuario, realiza un subrequest interno y llama al controller. El método _forward()_ devuelve el objeto **Response** devuelto por ese controller:

```
public function indexAction($name)
{
    $response = $this->forward('AppBundle:Something:fancy', array(
        'name'  => $name,
        'color' => 'green',
    ));

    // ... podemos modificar la respuesta o devolverla directamente

    return $response;
}
```

El método utiliza una representación especial del controller (el patrón de nombrado de controllers). En este caso, la función del controller objetivo es UnController::fancyAction() dentro del bundle AppBundle. El array que se pasa al método se convierte en los argumentos para el controller. Es la misma idea que se usa cuando se embeben controllers en templates. El método del controller objetivo sería algo así:

```
public function fancyAction($name, $color)
{
    // ... crea y devuelve un objeto Response
}
```

Igual que cuando se crea un controller para una route, el orden de los argumentos de _fancyAction_ no importa. Symfony hace coincidir los keys (_name_), con los nombres de los argumentos de los métodos (_$name_). Si cambias el orden de los argumentos, Symfony pasará igualmente el valor correcto de cada variable.