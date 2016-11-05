Según el API de Symfony, existen varios controllers incorporados en el framework:

### FrameworkBundle

#### Controller

Es el controller básico que se suele extender por defecto. Es una [clase abstract](http://diego.com.es/clases-abstractas-en-php) que implementa [ContainerAwareInterface](http://api.symfony.com/3.0/Symfony/Component/DependencyInjection/ContainerAwareInterface.html). Proporciona métodos para funcionalidades comunes que se necesitan en controllers.

**Controller** es el [base controller](http://diego.com.es/la-clase-base-controller-de-symfony) que se ha tratado en una sección anterior, que facilita redirecciones, renderizar templates, acceder a services, etc.

```
// src/AppBundle/Controller/DefaultController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class DefaultController extends Controller
{
    // ...
}
```

#### ControllerNameParser

Convierte un controlador desde la notación corta a:b:c (_BlogBundle:Post:index_) a una class::method fully-qualified (_Bundle\BlogBundle\Controller\PostController::indexAction_).

#### ControllerResolver

Extiende al ControllerResolver del HttpKernel.

#### RedirectController

RedirectController implementa **ContainerAwareInterface**, y añade dos métodos: _redirectAction_ y _urlRedirectAction_.

#### TemplateController

También implementa a **ContainerAwareInterface** y añade el método _templateAction_, que renderiza una template.

### TwigBundle

#### ExceptionController

Es el controlador que renderiza las páginas de error o excepción para una FlattenExtension dada. El método _showAction_ convierte una **Exception** en una **Response**.

### WebProfilerBundle

#### ExceptionController

Maneja las excepciones en el Web Profiler.

#### ProfilerController

Controller principal del Web Profiler.

#### RouterController

Maneja las routes del Web Profiler.

### HttpKernel

#### ControllerReference

Actúa como un marcador y almacén de datos para un Controller. Algunos métodos en Symfony aceptan una URI (como string) o un controller como argumentos. En este segundo caso, en lugar de pasar un array representando el controller, puedes usar una instancia de esta clase.

#### ControllerResolver

Esta implementación utiliza el atributo del request '__controller_' para determinar el controller a ejecutar y utiliza los atributos del request para determinar los argumentos del método del controller.

#### TraceableControllerResolver

Incorpora un objeto [Stopwatch](http://symfony.com/doc/current/components/stopwatch.html) en su constructor, clase que permite perfilar el código.

#### ControllerResolverInterface

Una implementación de ControllerResolverInterface permite determinar el controller a ejecutar basándose en un objeto Request.