En cada **request**, **Symfony** establece una **variable template global** _app_ por defecto. La variable **app** es una instancia de [GlobalVariables](http://api.symfony.com/3.0/Symfony/Bundle/FrameworkBundle/Templating/GlobalVariables.html) que da acceso a algunas variables específicas de la aplicación automáticamente:

*   _app.user_ - El objeto **user** actual.
*   _app.request_ - El objeto **request**.
*   _app.session_ - El objeto **session**.
*   _app.environment_ - El entorno actual.
*   _app.debug_ - True si está en **debug mode**. Sino es false.

```
<p>Nombre de usuario: {{ app.user.username }}</p>
{% if app.debug %}
    <p>Método request: {{ app.request.method }}</p>
    <p>Entorno de la aplicación: {{ app.environment }}</p>
{% endif %}
```

Además puedes incluir variables globales propias, desde el archivo _app/config/config.yml:_

```
# app/config/config.yml
twig:
    # ...
    globals:
        ga_tracking: UA-xxxxx-x
```

Ahora la variable **ga_tracking** está disponible en todas las **templates Twig**.

```
<p>El código de Google de tracking es: {{ ga_tracking }}</p>
```

### Utilizar parámetros del Service Container

Puedes aprovechar las ventajas del sistema [service parameters](http://symfony.com/doc/current/book/service_container.html#book-service-container-parameters) del contenedor de servicios que permite aislar o reusar el valor:

```
# app/config/parameters.yml
parameters:
    ga_tracking: UA-xxxxx-x
```

En el **archivo de configuración** pondríamos entonces:

```
# app/config/config.yml
twig:
    globals:
        ga_tracking: '%ga_tracking%'
```

La misma variable está disponible de la misma forma que antes.

### Referenciar _services_

En lugar de utilizar **valores estáticos** también puedes guardar el valor en un _service_. Siempre que se acceda a la variable global desde la **template**, se solicitará el service del **service container** y dará acceso a ese objeto.

El _service_ no se carga en modo **lazy**. Es decir, tan pronto como se cargue **Twig**, el service se instancia, incluso si ni siquiera se utiliza esa variable.

Para definir un service como una variable Twig global, se prefija el string con @ (igual que como se hace en la configuración de services).

```
# app/config/config.yml
twig:
    # ...
    globals:
        user_management: '@app.user_management'

```

### Utilizar una extensión Twig

Si la **variable global** que quieres establecer es más complicada (como un **objeto**), no podrás emplear el método anterior. Para ello tendrás que crear una [extensión Twig](http://symfony.com/doc/current/reference/dic_tags.html#reference-dic-tags-twig-extension) y devolver la variable global como una de las entradas en el método _getGlobals_.