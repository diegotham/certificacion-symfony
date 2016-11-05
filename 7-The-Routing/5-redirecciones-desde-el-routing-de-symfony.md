**Redireccionar a otra URL** se puede hacer de diferentes maneras. Desde el routing podemos emplear el [RedirectController](http://api.symfony.com/3.0/Symfony/Bundle/FrameworkBundle/Controller/RedirectController.html) del **FrameworkBundle**, utilizando una configuración en formato YAML, XML o PHP.

Podemos redireccionar a una **dirección** específica (_/about_) o a una **route** específica (_homepage_).

### Redireccionar usando una dirección URL

Suponemos que no hay un **controller** por defecto para el directorio _/_ de la aplicación y queremos redireccionar esos requests a _/app_. Necesitaremos utilizar la action _urlRedirectAction()_ para redireccionar a esta nueva URL:

```
# app/config/routing.yml

# Cargamos algunas routes
AppBundle:
    resource: '@AppBundle/Controller/'
    type:     annotation
    prefix:   /app

# Redireccionar el root
root:
    path: /
    defaults:
        _controller: FrameworkBundle:Redirect:urlRedirect
        path: /app
        permanent: true
```

En este ejemplo se configura la route para la dirección _/_ y dejamos al **RedirectController** que redireccione a _/app_. El parámetro _permanent_ le dice a la _action_ que emplee un [código de estado HTTP](http://diego.com.es/codigos-de-estado-http) **301** en lugar del código de estado por defecto **302**.

### Redireccionar usando una Route

Si por ejemplo estamos **migrando un sitio web de Wordpress a Symfony**, queremos redireccionar la dirección _/wp-admin_ a la route **sonata_admin_dashboard**. No conocemos la dirección, sólo el nombre de _route_. Esto se puede hacer con la action _redirectAction()_:

```
# app/config/routing.yml

# ...

# redirecting the admin home
root:
    path: /wp-admin
    defaults:
        _controller: FrameworkBundle:Redirect:redirect
        route: sonata_admin_dashboard
        permanent: true
```

### Redireccionar una URL con Trailing Slash

Ahora queremos redireccionar una URL con **trailing slash** (/) a una sin ella, por ejemplo de _/en/blog/_ a _/en/blog_.

Podemos hacerlo creando un controller que coincida con cualquier URL que tenga la trailing slash, removerla (manteniendo los parámetros query si hay) y redireccionar a la nueva URL con un **código de estado 301**:

```
// src/AppBundle/Controller/RedirectingController.php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;

class RedirectingController extends Controller
{
    public function removeTrailingSlashAction(Request $request)
    {
        $pathInfo = $request->getPathInfo();
        $requestUri = $request->getRequestUri();

        $url = str_replace($pathInfo, rtrim($pathInfo, ' /'), $requestUri);

        return $this->redirect($url, 301);
    }
}
```

Después, creamos una **route** en este **controller** que coincidirá cuando una URL con una **trailing slash** sea solicitada. **Asegúrate de poner esta route la última en el sistema**.

```
remove_trailing_slash:
    path: /{url}
    defaults: { _controller: AppBundle:Redirecting:removeTrailingSlash }
    requirements:
        url: .*/$
    methods: [GET]
```

Redireccionar un POST request no funciona bien en navegadores antiguos. Un 302 en un POST request enviaría un GET request después de la redirección. Por esta razón, la route sólo está configurada para GET requests.