Cada parámetro de **routing** o **valor por defecto** están disponibles como argumentos en el **método del controller**. Además, existen varios parámetros especiales, y cada uno añade una funcionalidad específica en una **aplicación**:

### _controller 

Este parámetro se utiliza para determinar qué controller se ejecuta cuando coincide con una route.

Cada route debe tener un parámetro **_controller**, que dicta qué controller ha de ejecutarse cuando la **dirección URL** coincide con la **route**. Este parámetro utiliza un patrón de _string_ llamado _logical controller name_, que **Symfony** mapea a un método y clase específicos. El patrón tiene tres partes, cada una separada por dos puntos:

**bundle:controller:action**

En el ejemplo que viene después, el valor **_controller** de **AppBundle:Article:show** significa:

Bundle -> **AppBundle**, Clase Controller -> **ArticleController**, Nombre de método -> **showAction**

### _format

Con este parámetro se establece el valor del formato del objeto **Request**. El formato request se emplea para cosas como establecer el Content-Type de la respuesta (por ejemplo un formato **json** se traduce a **Content-Type** como _application/json_). Puede también usarse en el controller para renderizar una template diferente para cada valor de __format_. El parámetro __format_ es es una buena forma de renderizar el mismo contenido en formatos diferentes.

### _locale

Utilizado para establecer el locale en el request (se concreta más en la siguiente sección sobre _locale_).

Vamos a verlos todos en un ejemplo:

```
// src/AppBundle/Controller/ArticleController.php

// ...
class ArticleController extends Controller
{
    /**
     * @Route(
     *     "/articles/{_locale}/{year}/{title}.{_format}",
     *     defaults={"_format": "html"},
     *     requirements={
     *         "_locale": "en|fr",
     *         "_format": "html|rss",
     *         "year": "\d+"
     *     }
     * )
     */
    public function showAction($_locale, $year, $title)
    {
    }
}
```

Para cada argumento del método **Controller**, **Symfony** busca un parámetro de route con ese nombre y asigna su valor a ese argumento. Cualquier combinación (en cualquier orden) de las siguientes variables pueden usarse como argumentos del método showAction(): _$_locale_, _$year_, _$title_, _$_format_, _$_controller_, _$_route_.

Ya que los placeholders y la colección de _defaults_ se unen después, también la variable _$_controller_ está disponible.

La variable especial _$_route_ se establece con el nombre de la route que haya coincidido.

### Pasar información extra al Controller

Los parámetros dentro de la colección _defaults_ no tienen por qué coincidir con un placeholder en la dirección de la route. Se puede emplear el array _defaults_ para especificar parámetros extra que serán accesibles como **argumentos en el Controller**:

```
# app/config/routing.yml
blog:
    path:      /blog/{page}
    defaults:
        _controller: AppBundle:Blog:index
        page:        1
        title:       "Hello world!"
```

Ahora podríamos acceder al parámetro extra en el Controller:

```
public function indexAction($page, $title)
{
    // ...
}
```

### Usar parámetros del Service Container en las routes

En ocaciones puede resultar útil hacer que algunas partes de las routes sean configurables globalmente. Por ejemplo si construyes un sitio internacional, probablemente empezarás con uno o dos locales. Seguro que añadirás un requisito a tus routes para evitar que un usuario acceda a un locale que tu aplicación no soporte. 

Puedes poner el requirement __locale_ en todas las **routes**, pero una solución mejor es emplear un **parámetro service container** configurable detro de la configuración de tu routing:

```
# app/config/routing.yml
contact:
    path:     /{_locale}/contact
    defaults: { _controller: AppBundle:Main:contact }
    requirements:
        _locale: '%app.locales%'
```

Ahora podemos controlar el parámetro _app.locales_ desde alguna parte del container:

```
# app/config/config.yml
parameters:
    app.locales: en|es
```

También puedes usar un parámetro directamente para definir la dirección de tu route (o parte de la dirección):

```
# app/config/routing.yml
some_route:
    path:     /%app.route_prefix%/contact
    defaults: { _controller: AppBundle:Main:contact }
```

Como en los archivos de configuración del **service container**, si necesitas un % en la route, se puede escapar con otro carácter %, por ejemplo _/score-50%%_. Sin embargo, como los caracteres % incluidos en cualquier URL son codificados automáticamente, la URL resultante de este ejemplo sería /score-50%25 (%25 es el resultado de codificar el carácter %).