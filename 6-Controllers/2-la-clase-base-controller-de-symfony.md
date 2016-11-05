**Symfony** viene con una clase base opcional **Controller**. Si la extiendes, tendrás acceso a un número de **métodos helper** y a todos los objetos _service_ a través del container.

Para usarla se añade la sentencia use con _Symfony\Bundle\FrameworkBundle\Controller\Controller_ y se extiende la clase:

```
// src/AppBundle/Controller/HelloController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;

class HelloController extends Controller
{
    // ...
}
```

Esto realmente no cambia nada sobre el comportamiento del controller, simplemente proporciona acceso a **métodos helper** que la **base controller class** pone a disposición. Esto son simplemente shorcuts para usar **funcionalidad core de Symfony** que está disponible sin el uso de esta clase base Controller. Para ver la funcionalidad core en acción puedes mirar en la [Controller Class](https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/Controller.php).

Para ver cómo funciona un controller si no extiende esta clase base, puedes [emplear los controllers como services](http://symfony.com/doc/current/cookbook/controller/service.html). Esto es opcional pero te da control sobre los objetos y dependencias exactos que serán inyectados en tu controller.

La **base controller class** facilita las siguientes funcionalidades:

### Redirecciones

Si quieres redirigir al usuario a otra página, usa el método _redirectToRoute()_:

```
public function indexAction()
{
    return $this->redirectToRoute('homepage');

    // redirectToRoute es equivalente a usar redirect() y generateUrl() juntos:
    // return $this->redirect($this->generateUrl('homepage'), 301);
}
```

Si quieres redirigir externamente, emplea _redirect()_ y pasa la URL:

```
public function indexAction()
{
    return $this->redirect('http://symfony.com/doc');
}
```

Por defecto el método _redirectToRoute()_ realiza una **redirección temporal 302**. Para realizar una **redirección permanente 301**, modifica el tercer argumento:

```
public function indexAction()
{
    return $this->redirectToRoute('homepage', array(), 301);
}
```

El método _redirectToRoute()_ es simplemente un shortcut que crea un objeto **Response** que se especializa en redireccionar al usuario. Es equivalente a:

```
use Symfony\Component\HttpFoundation\RedirectResponse;

public function indexAction()
{
    return new RedirectResponse($this->generateUrl('homepage'));
}
```

### Renderizar templates

Si quieres **devolver HTML** tendrás que renderizar una template. El método _render()_ renderiza una template y pone el contenido en un objeto Response:

```
// renders app/Resources/views/hello/index.html.twig
return $this->render('hello/index.html.twig', array('name' => $name));

```

Puedes también poner las templates en subdirectorios más profundos. Simplemente intenta evitar las estructuras demasiado profundas:

```
// renders app/Resources/views/hello/greetings/index.html.twig
return $this->render('hello/greetings/index.html.twig', array(
    'name' => $name
));
```

También puedes poner las templates en el directorio _Resources/views_ de tu bundle y referenciarlas de la forma: **BundleName:DirectoryName:FileName**. Por ejemplo **AppBundle:Hello:index.html.twig** se referiría a la template localizada en _src/AppBundle/Resources/views/Hello/index.html.twig_. Symfony recomienda poner las templates en el directorio _/app_ y no en los bundles.

### Acceder a otros services

Symfony viene equipado con un bastantes objetos, llamados _services_. Estos se emplean para renderizar templates, enviar emails, consultar la base de datos y otras tareas. Cuando instalas un bundle, probablemente lleva consigo todavía más _services_.

Cuando se extiende la **base controller class**, puedes acceder cualquier _service_ de **Symfony** con el método _get()_, como los siguientes:

```
$templating = $this->get('templating');

$router = $this->get('router');

$mailer = $this->get('mailer');
```

Para ver que otros services existen utiliza el siguiente comando:
```
php bin/console debug:container
```
