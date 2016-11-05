La estructura básica de directorios de un **bundle reusable** (por ejemplo **AcmeBlogBundle**) en **Symfony** es como sigue:
```
<your-bundle>/
├─ AcmeBlogBundle.php
├─ Controller/
├─ README.md
├─ LICENSE
├─ Resources/
│   ├─ config/
│   ├─ doc/
│   │  └─ index.rst
│   ├─ translations/
│   ├─ views/
│   └─ public/
└─ Tests/
```

Los siguientes archivos son obligatorios, ya que aseguran una estructura sobre la que algunas herramientas automáticas pueden actuar:

*   **AcmeBlogBundle.php**. Esta es la clase que transforma un directorio normal en un bundle Symfony (con el nombre de bundle que se haya especificado).
*   **README.md**. Este archivo contiene la descripción básica del bundle y normalmente muestra ejemplos básicos y enlaza a su documentación completa (puede emplear cualquiera de los formatos soportados por GitHub, como README.rst).
*   **LICENSE**. Los contenidos de la licencia utilizada por el código. La mayoría de bundles de terceros se publican bajo licencia MIT, pero puedes [elegir cualquier licencia](http://choosealicense.com/).
*   **Resources/doc/index.rst**. El archivo root para la documentación del bundle. 

La **profundidad de subdirectorios** se debería mantener al mínimo para la mayoría de clases y archivos (dos niveles como máximo).

El **directorio bundle** es read-only. Si necesitas escribir archivos temporales, guárdalos en el directorio _cache/_ o _log/_ de la aplicación. Hay herramientas que pueden generar archivos en el directorio bundle, pero sólo si los archivos generados van a formar parte del repositorio.

Las siguientes clases y archivos tienen emplazamientos específicos:

| | |
| -------- | -------- |
| **Tipo** | **Directorio** |
| Commands | Command/ |
| Controllers | Controller/ |
| Service Container Extensions | DependencyInjection/ |
| Event Listeners | EventListener/ |
| Model classes | Model |
| Configuration | Resources/config/ |
| Web Resources (CSS, JS, images) | Resources/public/ |
| Translation files | Resources/translations/ |
| Templates | Resources/views/ |
| Unit and Functional Tests | Tests/ |

### Classes

La **estructura de directorios del bundle** se emplea como **jerarquía de namespaces**. Por ejemplo **ContentController** se guarda en _Acme/BlogBundle/Controller/ContentController.php_ y el nombre de clase fully qualified es _Acme\BlogBundle\Controller\ContentController_.

Todas las clases y archivos deben respetar los [estándares de código de Symfony](http://diego.com.es/estandares-de-codigo-en-symfony).

Algunas clases deberían verse como _facades_ y deberían ser tan cortas como fuera posible, como **Commands**, **Helpers**, **Listeners** y **Controllers**.

Las clases que conectan con el **event dispatcher** deberían tener el **sufijo Listener**.

Las clases **Exception** deben ir guardadas en un subnamespace Exception.

### Vendors

Un bundle no debe embeber librerías PHP de terceros. Debe dejar el trabajo al autoloader de Symfony. Tampoco ha de embeber librerías de terceros escritas en cualquier otro lenguaje como JavaScript o CSS.

### Tests

Un bundle debería ir con tests escritos en PHPUnit y guardarlos en el directorio Tests/. Los tests han de seguir los siguientes principios:

*   Los tests han de ser ejecutables con un simple comando _phpunit_.
*   Los functional tests deben usarse sólo para probar el output de respuesta y algula información de perfil si existe.
*   Los tests deberían cubrir al menos el 95% del código base.

Los tests no deben contener scripts AllTests.php, pero deben confiar en la existencia de un archivo phpunit.xml.dist.

### Documentation

Todas las clases y funciones deben ir con **PHPDoc**.

Documentación extensiva debería proporcionarse también en formato [reStructuredText](http://symfony.com/doc/current/contributing/documentation/format.html), bajo el directorio _Resources/doc_. El archivo _Resources/doc/index.srt_ es el único archivo obligatorio y debe ser el punto de entrada a la documentación.

Para facilitar la instalación de bundles de terceros es conveniente emplear un [formato estandarizado en el archivo README.md](http://symfony.com/doc/current/cookbook/bundles/best_practices.html#installation-instructions).

### Routing

Si el bundle proporciona **routes**, deben ir prefijadas con el alias del bundle. Por ejemplo si tu bundle se llama **AcmeBlogBundle**, todas sus routes deben ir prefijadas con _acme_blog__.

### Templates

Si un bundle proporciona templates, deben usar **Twig**. Un bundle no debe proporcionar un layout principal, a no ser que proporcione una aplicación completa.

### Translation files

Si un bundle proporciona mensajes de traducción, deben definirse en el formato XLIFF. El dominio debe definirse después del nombre del bundle (_acme_blog_). Un bundle no debe sobreescribir mensajes existentes de otro bundle.

### Configuration

Para proporcionar más flexibilidad. un bundle puede proporcionar ajustes de configuración usando mecanismos por defecto de Symfony.

Para ajustes de configuración simples, se puede emplear la entrada por defecto _parameters_ de la **configuración Symfony**. Los parámetros sin simples parejas _key_/_value_, siendo _value_ cualquier **valor válido de PHP**. Cada nombre de parámetro debería empezar con el alias del bundle, aunque esto sólo es una sugerencia de buena práctica. El resto del nombre del parámetro debe emplear un punto (.) para separar partes diferentes (por ejemplo: **acme_blog.author.email**).

El usuario final del bundle podrá proporcional los valores en cualquier archivo de configuración (YAML, XML, PHP).

```
# app/config/config.yml
parameters:
    acme_blog.author.email: 'ejemplo@ejemplo.com'
```

Accede a los parámetros de configuración en tu código desde el container:

```
$container->getParameter('acme_blog.author.email');
```

Aun cuando este mecanismo es bastante simple, deberías considerar emplear un [método semántico de configuración del bundle](http://symfony.com/doc/current/cookbook/bundles/configuration.html).

### Versioning

Los bundles deben ir en versiones al estilo de [Semantic Versioning](http://diego.com.es/versiones-de-aplicaciones).

### Services

Si el bundle define _services_, deben ir prefijados con el alias del bundle. Por ejemplo un service de **AcmeBlogBundle** debe ir prefijado con **acme_blog**. Además, los services que no sean aplicados directamente en la aplicación, deberían [definirse como private](http://symfony.com/doc/current/components/dependency_injection/advanced.html#container-private-services).

### Composer Metadata

El archivo composer.json debe incluir los siguientes metadatos:

*   **name**. Vendor y nombre corto del bundle. Si lanzas el bundle en tu nombre en lugar de en nombre de una compañía, emplea tu propio nombre (_johnsmith/blog-bundle_). El nombre corto del bundle excluye el nombre del vendor y separa cada palabra con un guión. Por ejemplo AcmeBlogBundle se transforma en blog-bundle y AcmeSocialConnectBundle es social-connect-bundle.
*   **description**. Explicación breve del **objetivo del bundle**.
*   **type**. Emplea el valor **symfony-bundle**.
*   **license**. MIT es la licencia preferida para bundles de Symfony, pero puedes usar cualquier otra licencia.
*   **autoload**. Esta información es empleada por Symfony para cargar las clases del bundle. El estándar autoload PSR-4 es recomendable para bundles modernos, aunque también soporta el PSR-0.

Para que sea más fácil encontrar el bundle por terceros, es recomendable registrarlo en [Packagist](https://packagist.org/).