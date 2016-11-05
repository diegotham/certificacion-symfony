No existe realmente una **convención de nombres para controllers**, aunque se puede tender a lo siguiente:

*   Nombrarlo con la principal entidad con la que trata. Por ejemplo si en un PostBundle hay dos entidades Posts y Comments, puedo crear un controller para posts y otro para comments de forma PostController y CommentController.
*   Si sólo hay un Controller en el bundle, llamarlo con el nombre del bundle: AppBundle -> AppController.
*   Si existen varios módulos en el bundle, puedes llamar a cada controller como cada módulo. Por ejemplo FOSUserBundle tienen los módulos Security, Resetting, Registration, etc. Cada uno tiene su respectivo controller.

Todo esto es muy relativo, ya que depende de la extensión del controller. Hay que tener en cuenta que hay que tender a tener **controllers** reducidos y a agrandar los **models**. 

### Patrón de nombres de controllers en routes

Cada route debe tener un parámetro **_controller**, que dicta que controller ha de ejecutarse cuando se solicita esa route. Este parámetro utiliza un patrón sencillo de string llamado _logical controller name_, que **Symfony** mapea a un método y clase específicos de PHP. El patrón tiene tres partes, cada parte separada por dos puntos:

_**bundle:controller:action**_

Por ejemplo un valor **_controller** de _AppBundle:Blog:show_ significa:

Bundle -> AppBundle, Clase del Controller -> BlogController, Nombre del método -> showAction

El controller puede resultar algo así:

```
// src/AppBundle/Controller/BlogController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class BlogController extends Controller
{
    public function showAction($slug)
    {
        // ...
    }
}
```

Symfony añade el _string_ **Controller** al nombre de la clase (Blog -> **BlogController**) y Action al nombre del método (show -> **showAction**).

Puedes también hacer referencia a este controller con su nombre de clase y método fully-qualified: _AppBundle\Controller\BlogController::showAction_. Pero si sigues algunas convenciones simples, el nombre lógico es más conciso y permite más flexibilidad.

Además de poder usar el nombre lógico o el nombre de clase fully-qualified, **Symfony** soporta una tercera forma de referirse al controller. Este método utiliza sólo un separador de dos puntos (por ejemplo: _service_name:indexAction_) y se refiere al controller como _service_. [Aquí](http://symfony.com/doc/current/cookbook/controller/service.html) puedes ver cómo definir controllers como services.