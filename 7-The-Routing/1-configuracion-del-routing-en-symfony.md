Definir una route es fácil, y una aplicación normalmente tendrá muchas de ellas. Una **route** básica consiste en dos partes: el _path_ y el array _defaults_. Vamos a ver la misma configuración de un route en los cuatro formatos posibles:

#### XML

```
<!-- app/config/routing.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<routes xmlns="http://symfony.com/schema/routing"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/routing
        http://symfony.com/schema/routing/routing-1.0.xsd">

    <route id="blog_show" path="/blog/{slug}">
        <default key="_controller">AppBundle:Blog:show</default>
    </route>
</routes>
```

#### Anotaciones

```
// src/AppBundle/Controller/MainController.php

// ...
class MainController extends Controller
{
    /**
     * @Route("/")
     */
    public function homepageAction()
    {
        // ...
    }
}
```

#### YAML

```
# app/config/routing.yml
_welcome:
    path:      /
    defaults:  { _controller: AppBundle:Main:homepage }
```

#### PHP

```
// app/config/routing.php
use Symfony\Component\Routing\RouteCollection;
use Symfony\Component\Routing\Route;

$collection = new RouteCollection();
$collection->add('_welcome', new Route('/', array(
    '_controller' => 'AppBundle:Main:homepage',
)));

return $collection;
```

Esta route llama al controller _homepageAction_ cuando se solicita la URL /. El string de **_controller** se traduce en Symfony como una función PHP y se ejecuta. 

El fomato recomendado por **Symfony** para el routing son las **anotaciones**.

Muchas de las routes en las aplicaciones pueden contener **wildcard placeholders**:

```
// src/AppBundle/Controller/BlogController.php

// ...
class BlogController extends Controller
{
    /**
     * @Route("/blog/{slug}")
     */
    public function showAction($slug)
    {
        // ...
    }
}
```

La dirección URL podrá coincidir con cualquier valor después de blog: _/blog/{slug}_. Si la URL es _/blog/hello-world_, una variable _$slug_, con un valor de hello-world estará disponible en el controller. Esto puede usarse, por ejemplo, para cargar la entrada de blog _hello-world_.

La dirección no coincidirá con _/blog_, ya que por defecto todos los placeholders son requeridos, pero puede añadirse un valor por defecto.

### Incluir fuentes externas de routing

Todas las routes se cargan desde un mismo **archivo de configuración**, generalmente _app/config/routing.yml_, pero si empleas **routing annotations**, tendrás que apuntar el router a los controllers con las anotaciones. Esto se puede hacer importando los directorios en la **configuración del routing**:

```
# app/config/routing.yml
app:
    resource: '@AppBundle/Controller/'
    type:     annotation # necesario para activar el Annotation reader para este resource
```

El key _resource_ carga el routing resource. En este ejemplo el resource es un directorio, donde el shortcut **@AppBundle** resuelve el directorio entero del bundle **AppBundle**. Cuando se apunta a un directorio, todos los archivos de ese directorio de analizan y se pasan al routing.

Puedes también incluir otros archivos de configuración del routing, lo que es frecuente cuando se importa el routing de bundles de terceros:

```
# app/config/routing.yml
app:
    resource: '@AcmeOtherBundle/Resources/config/routing.yml'
```

### Prefijar routes importadas

Puedes también establecer un prefijo _prefix_ a las routes importadas. Por ejemplo, si queremos prefijar todas las routes en el **AppBundle** con _/site_ (_/site/blog/{slug}_) en lugar de _/blog/{slug}_:

```
# app/config/routing.yml
app:
    resource: '@AppBundle/Controller/'
    type:     annotation
    prefix:   /site
```

La dirección de cada route cargada desde la nueva **routing resource** será ahora prejijada con el string _/site_.

Si utilizas anotaciones, también puedes prefijar varias routes definiendo la anotación _@Route_ en la clase del controlador:

```
// src/AppBundle/Controller/BlogController.php

/**
 * @Route("/blog")
 */
class BlogController extends Controller
{
    /**
     * @Route("/{slug}")
     */
    public function showAction($slug)
    {
        // la URL de este controlador es: /blog/{slug}
        // ...
    }
}
```
