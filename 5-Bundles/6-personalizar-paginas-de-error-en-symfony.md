En las **aplicaciones Symfony**, todos los errores se tratan como excepciones, no importa que sea un **error 404 Not Found** o un **error fatal** debido al lanzamiento de alguna excepción en el código.

En el **entorno de desarrollo**, Symfony captura todas las excepciones y muestra una página especial de excepciones con abundante información de depuración para ayudar a resolver fácilmente un problema. 

Ya que estas páginas ocntienen abundante información interna sensible, Symfony no los mostrará en el **entorno de producción**. A cambio, muestra una **página de error genérica**.

Las páginas de error para el entorno de producción pueden customizarse de diferentes formas en función de tus necesidades:

1.  Si sólo quieres cambiar los contenidos y estilos de las páginas de error para tu aplicación, basta con sobreescribir las templates de error por defecto.
2.  Si también quieres indagar en la lógica empleada por Symfony para generar las páginas de error, puedes sobreescribir el exception controller por defecto.
3.  Si necesitas un total control de las excepciones para ejecutar tu propia lógica utiliza el evento kernel.exception.

### 1\. Sobreescribir las templates de error por defecto

Cuando carga una página de error, un **ExceptionController** interno se emplea para renderinzar una template **Twig** que mostrar al usuario.

Esto controller emplea el [código de estado HTTP](http://diego.com.es/codigos-de-estado-http), el formato request y la siguiente lógica para determinar el nombre de archivo de la template:

1.  Busca una template para el formato dado y el código de estado (como _error404.json.twig_ o _error500.html.twig_).
2.  Si la template anterior no existe, descarta el código de estado y busca una template genérica para el formato dado (como _error.json.twig_ o _error.xml.twig_).
3.  Si ninguna de las templates anteriores existe, devuelve la template de error genérica de HTML (_error.html.twig_).

Para sobreescribir estas templates se emplea el método estándar de Symfony para sobreescribir templates que están dentro de un bundle: ponerlas en el directorio _app/Resources/TwigBundle/views/Exception/_.

Un proyecto típico que devuelve páginas HTML y JSON:
```
app/
└─ Resources/
   └─ TwigBundle/
      └─ views/
         └─ Exception/
            ├─ error404.html.twig
            ├─ error403.html.twig
            ├─ error.html.twig      # All other HTML errors (including 500)
            ├─ error404.json.twig
            ├─ error403.json.twig
            └─ error.json.twig      # All other JSON errors (including 500)

```

#### Ejemplo de página de error 404

Para sobreescribir la **template de error 404 para páginas HTML**, crea una nueva template error404.html.twig en app/Resources/TwigBundle/views/Exception/:

```
{# app/Resources/TwigBundle/views/Exception/error404.html.twig #}
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Página no encontrada</h1>

    {# ejemplo de parte de la página que sólo se muestra a usuario identificado #}
    {% if is_granted('IS_AUTHENTICATED_FULLY') %}
        {# ... #}
    {% endif %}

    <p>
        No se ha podido localizar la página solicitada. Comprueba cualquier error
        escribiendo la URL o <a href="{{ path('homepage') }}">vuelve al inicio</a>.
    </p>
{% endblock %}
```

En caso de que la necesites, el **ExceptionController** pasa información a la template de error a través de las variables **status_code** y **status_text** que guardan el código de estado HTTP y el mensaje respectivamente.

Puedes customizar el código de estado implementando [HttpExceptionInterface](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/Exception/HttpExceptionInterface.html) y su método requerido _getStatusCode()_. Sino, el código de estado por defecto es 500.

Las páginas de excepción mostradas en el entorno de desarrollo pueden personalizarse de la misma forma que las páginas de error. Crea una nueva template _exception.html.twig_ para la **excepción estándar de HTML** o _exception.json.twig_ para la **página de excepción de JSON**.

#### Testear páginas de error durante el desarrollo

Cuando estás en el entorno de desarrollo, Symfony muestra la página de excepción en lugar de tu página de error personalizada. Así que, ¿Cómo puedes ver cómo se ve y depurarla?.

El **ExcepcionController** por defecto permite ver tus páginas de error durante el desarrollo.

Para utilizar esta característica tienes que tener una definición en el archivo _routing_dev.yml_ de la siguiente forma:

```
# app/config/routing_dev.yml
_errors:
    resource: "@TwigBundle/Resources/config/routing/errors.xml"
    prefix:   /_error
```

Con esta route definida, para ver las templates puedes dirigirte a las siguientes URLs:
```
http://localhost/app_dev.php/_error/{statusCode}
http://localhost/app_dev.php/_error/{statusCode}.{format}
```

para ver la página de error de un código de estado dado como HTML o para un código de estado y formato dados.

### 2\. Sobreescribir el ExceptionController por defecto

Si necesitas un poco más de flexibilidad más allá de sobreescribir una template, puedes cambiar el controller que renderiza la página de error. Por ejemplo, podrías necesitar pasar algunas variables adicionales en tu template.

Para hacerlo, simplemente crea un nuevo controller en cualquier parte de tu aplicación y establece la opción de configuración **twig.excepcion_controller** para señalarle:

```
# app/config/config.yml
twig:
    exception_controller:  AppBundle:Exception:showException
```

La clase **ExceptionListener** utilizada por el **TwigBundle** como listener del evento kernel.exception crea el request que será lanzado a tu controller. Además, a tu controller se le pasarán dos parámetros:

*   **exception**. Una instancia de **FlattenException** creada por la excepción que se esté manejando.
*   **logger**. Una instancia de **DebugLoggerInterface** que puede ser **null** en algunas circunstancias.

En lugar de crear un nuevo exception controller desde cero puedes también extender el ExceptionController por defecto. En ese caso, podrías querer sobreescribir uno o ambos métodos _showAction()_ y _findTemplate()_. El segundo localiza la template a usarse.

La **vista previa de la página de error** también funciona para la propia configuración de tus controllers.

### 3\. Empleando el evento kernel.exception

Cuando se lanza una excepción, la clase **HttpKernel** la captura y lanza un evento **kernel.exception**. Esto te da el poder de convertir la excepción en una **Response** de diferentes fomas. Trabajar con este evento es bastante más poderoso que los dos casos anteriores, pero también requiere un **entendimiento de las partes internas de Symfony**.

Escribiendo tu propio **event listener** para el evento _kernel.exception_ te permite tener una vista más cercana de la excepción y tomar diferentes acciones dependiendo de la misma. Esas acciones pueden ser: añadir logs por la excepción, redireccionar al usuario a otra página o renderizar páginas de error especiales.

Si tu listener llama al método _setResponse()_ en el evento **GetResponseForExceptionEvent**, la propagación se parará y la respuesta se enviará al cliente.

Este enfoque te permite manejar los errores centralizados y por capas: en lugar de cachear y manejar las mismas excepciones en varios controllers siempre, puedes tener uno o varios listeners que se hagan cargo de ellas.

Puedes ver la clase [ExcepcionListener](http://api.symfony.com/3.0/Symfony/Component/Security/Http/Firewall/ExceptionListener.html) para un ejemplo real de un listener avanzado de este tipo. Este listener maneja varias excepciones relacionadas con la seguridad que se lanzan en tu aplicación (como [AccessDeniedExceptcion](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Exception/AccessDeniedException.html)) y toma medidas como redireccionar al usuario a la página de login, deslogearlos y otras cosas.