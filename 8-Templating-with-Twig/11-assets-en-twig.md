Las templates normalmente hacen referencia a **imágenes**, **JavaScript**, **hojas de estilo** y otros **assets**. **Symfony** proporciona una forma dinámica y flexible de cargarlos a través de la función _asset_ de Twig:

```
<img src="{{ asset('images/logo.png') }}" alt="Symfony!" />

    <link href="{{ asset('css/blog.css') }}" rel="stylesheet" />
```

El principal objetivo de la función asset es hacer la aplicación más portable. Si tu aplicación está en el **root** del **host** (_http://ejemplo.com_), entonces los directorios deberían ser _/images/logo.png_. Pero si tu aplicación está en un subdirectorio (_http://example.com/my_app_), cada directorio de los assets deberá renderizarse en el subdirectorio (_/my_app/images/logo.png_). La función _asset_ se hace cargo de esto determinando cómo está construida y generando los directorios correctos.

Además, si empleas la función _asset_, **Symfony** puede añadir automáticamente un **query string** a tu asset, para garantizar que las actualizaciones de assets estáticos no sean cacheados. Por ejemplo, _/images/logo.png_ puede cambiarse a _/images/logo.png?v2_. La versión de todo el conjunto de assets se puede modificar en la opción de configuración [assets_version](http://symfony.com/doc/current/reference/configuration/framework.html#ref-framework-assets-version).

Si necesitas establecer una versión para un asset específico, puedes establecer el argumento _version_.

```
<img src="{{ asset('images/logo.png', version='3.0') }}" alt="Symfony!" />
```

Si no proporcionas una versión o pones _null_, el paquete de versiones por defecto (el de _assets_version_) se empleará. Si pones false, la URL versionizada se desactivará para este asset en concreto.

Si necesitas **URLs absolutas** para **assets**, puedes establecer el argumento _absolute_ si estás usando **Twig** a **true**:

```
<img src="{{ absolute_url(asset('images/logo.png')) }}" alt="Symfony!" />
```

### Incluir stylesheets y JavaScripts en Twig

Hoy en día ningún sitio web estaría completo sin incluir archivos **JavaScript** y **hojas de estilos**. En Symfony la inclusión de estos assets se lleva a cabo de forma elegante aprovechando la **herencia de templates de Symfony**.

Symfony también incluye la posibilidad de emplear la librería [Assetic](http://symfony.com/doc/current/cookbook/assetic/asset_management.html), que incluye mayores posibilidades para el manejo de los assets.

Ponemos un ejemplo en el que insertamos dos bloques a nuestra template base que almacenarán nuestros assets: uno llamado **stylesheets** dentro de la etiqueda _head_ y otro llamado **javascripts** encima del cierre de la etiqueta _body_. Estos bloques contendrán todos los stylesheets y JavaScripts que necesitarás en tu sitio:

```
{# app/Resources/views/base.html.twig #}
    <html>
    <head>
    {# ... #}

    {% block stylesheets %}
    <link href="{{ asset('css/main.css') }}" rel="stylesheet" />
    {% endblock %}
    </head>
    <body>
    {# ... #}

    {% block javascripts %}
    <script src="{{ asset('js/main.js') }}"></script>
    {% endblock %}
    </body>
    </html>
```

Si ahora queremos incluir assets extra en una template hija, podemos emplear la función _parent_. Vamos a incluir una **hoja de estilos específica** para la sección de contacto:

```
{# app/Resources/views/contact/contact.html.twig #}
    {% extends 'base.html.twig' %}

    {% block stylesheets %}
    {{ parent() }}

    <link href="{{ asset('css/contact.css') }}" rel="stylesheet" />
    {% endblock %}

    {# ... #}
```

En la template hija simplemente sobreescribes el bloque stylesheets y pones tu nuevo dentro de ese bloque. La función parent hará que los _assets_ de la madre se incluyan también.

También puedes incluir assets localizados en la carpeta _Resources/public_. Necesitarás ejecutar el comando _php bin/console assets:install target [-- symlink],_ que mueve (o emplea [_symlink_](http://php.net/manual/es/function.symlink.php)) los archivos a la localización correcta. El destino por defecto es "web". 

```
<link href="{{ asset('bundles/acmedemo/css/contact.css') }}" rel="stylesheet" />
```

El resultado final es una página que incluye tanto a _main.css_ como a _contact.css_.