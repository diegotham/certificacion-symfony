**Internacionalización y localización**, a menudo abreviado [i18n](https://en.wikipedia.org/wiki/Internationalization_and_localization), se refiere al proceso de abstraer strings y otras piezas específicas de _locales_ de la aplicación en una capa donde puedan ser traducidas y convertidas basándose en el _locale_ del usuario (por ejemplo idioma y país). Para texto, esto significa envolver cada uno con una función capaz de traducir el texto o mensaje en el lenguaje del usuario:

```
// el texto siempre se mostrará en inglés
dump('Hello World');
die();

// el texto puede traducirse en el idioma del usuario final o
// por defecto en inglés
dump($translator->trans('Hello World'));
die();
```

El término _locale_ hace referencia básicamente al **idioma** y **país** del usuario. Puede ser cualquier string que tu aplicación utiliza para manejar las traducciones y otras diferencias de formato (como formato de moneda). El más recomendable y usado es utilizar el código de lenguaje [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes), una barra baja y después el código de país [ISO 3166-1 alpha 2](https://en.wikipedia.org/wiki/ISO_3166-1), por ejemplo para **España** es **es_ES**. 

El proceso de utilizar el [componente Translation de Symfony](http://symfony.com/doc/current/components/translation/usage.html) se puede dividir como sigue:

1.  Activar y configurar el translation service de Symfony
2.  Abstraer strings (como mensajes) envolviéndolos en llamadas al Translator
3.  Crear archivos de traducción para cada locale soportado que traduce cada mensaje en la aplicación
4.  Determinar, establecer y manejar el locale del usuario para el request y opcionalmente en la sesión del usuario entera

**Indice de contenido**

1.  [Configuración](#Configuracion)
2.  [Traducción básica](#TraduccionBasica)
3.  [Traducción en Twig](#TraduccionEnTwig)
4.  [Localización de los archivos de traducción](#LocalizacionDeLosArchivosDeTraduccion)
5.  [Manejar el locale del usuario](#ManejarElLocaleDelUsuario)
6.  [Traducir constraints](#TraducirConstraints)
7.  [Depurar traducciones](#DepurarTraducciones)

### <a id="Configuracion" name="Configuracion"></a>1\. Configuración

Las traducciones se manejan con el _service translator_ que utiliza el _locale_ del usuario para buscar y devolver mensajes traducidos. Antes de utilizarlo, activa el translator en la configuración:

```
# app/config/config.yml
framework:
    translator: { fallbacks: [en] }
```

El _locale_ empleado en las traducciones es el guardado en el _request_. Esto se establece normalmente a través del atributo _locale en las routes.

### <a id="TraduccionBasica" name="TraduccionBasica"></a>2\. Traducción básica

La traducción de textos se hace a través de el translator service ([Translator](http://api.symfony.com/3.0/Symfony/Component/Translation/Translator.html)). Para traducir un bloque de texto (llamado mensaje), utiliza el método [trans()](http://api.symfony.com/3.0/Symfony/Component/Translation/Translator.html#method_trans). Suponemos, por ejemplo, que traducimos un mensaje desde un controller:

```
// ...
use Symfony\Component\HttpFoundation\Response;

public function indexAction()
{
    $translated = $this->get('translator')->trans('Symfony is great');

    return new Response($translated);
}
```

Cuando se ejecuta este código, **Symfony** tratará de traducir el mensaje "Symfony is great" basándose en el locale del usuario. Para que esto funcione, necesitas decirle a Symfony cómo traducir el mensaje a través de una _translation resource_, que normalmente es un archivo que contiene una colección de traducciones para un locale dado. El "diccionario" de traducciones puede crearse en varios formatos, siendo XLIFF el recomendado:

XLIFF:

```
<!-- messages.fr.xlf -->
<?xml version="1.0"?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
    <file source-language="en" datatype="plaintext" original="file.ext">
        <body>
            <trans-unit id="symfony_is_great">
                <source>Symfony is great</source>
                <target>J'aime Symfony</target>
            </trans-unit>
        </body>
    </file>
</xliff>
```

YAML:

```
# messages.fr.yml
Symfony is great: J'aime Symfony
```

Ahora si el lenguaje del locale del usuario es francés (por ejemplo fr_FR o fr_BE), el mensaje se traducirá a J'aime Symfony. 

#### El proceso de traducción

Para traducir el mensaje, symfony emplea un simple proceso:

*   Symfony almacena el locale del usuario actual en el request
*   Un catálogo de mensajes de traducción se carga de las diferentes fuentes de traducción para el locale (como fr_FR). El resultado es un gran diccionario de traducciones.
*   Si el mensaje se encuentra en el catálogo, se devuelve la traducción. Sino, el translator devuelve el mensaje original.

Cuando se emplea el método trans(), Symfony busca el string exacto dentro del catálogo de mensajes y lo devuelve (si existe).

#### Message placeholders

A veces, un mensaje que contiene una variable necesita ser traducido:

```
use Symfony\Component\HttpFoundation\Response;

public function indexAction($name)
{
    $translated = $this->get('translator')->trans('Hello '.$name);

    return new Response($translated);
}
```

Sin embargo, crear una traducción para este string es imposible ya que el translator intentará buscar el mensaje exacto, incluyendo las partes de variable (por ejemplo "Hello Ryan" o "Hello Fabien").

#### Pluralización

Otra complicación es cuando tienes traducciones que pudieran ser o no ser plurales, basándose en alguna variable:

```
There is one apple.
There are 5 apples.
```

Para manejar esto, puedes emplear el método [transChoice()](http://api.symfony.com/3.0/Symfony/Component/Translation/Translator.html#method_transChoice) o el filtro _transchoice_ en tu template. Para más información, puedes leer [Pluralization](http://symfony.com/doc/current/components/translation/usage.html#component-translation-pluralization) en la documentación del componente Translation.

### <a id="TraduccionEnTwig" name="TraduccionEnTwig"></a>3\. Traducción en Twig

La mayoría de las veces las traducciones ocurren en las plantillas. Symfony proporciona soporte tanto para Twig como para PHP.

Symfony proporciona etiquetas especializadas _trans_ y _transchoice_ para ayudar con la traducción del mensaje o de bloques estáticos de texto:

```
{% trans %}Hello %name%{% endtrans %}

{% transchoice count %}
    {0} There are no apples|{1} There is one apple|]1,Inf[ There are %count% apples
{% endtranschoice %}
```

La etiqueta transchoice automáticamente obtiene la variable _%count%_ del contexto actual y lo pasa al translator. Este mecanismo sólo funciona cuando usas un placeholder con el patrón _%var%_. Si necesitas emplear el símbolo de porcentaje (%) en un string, puedes escaparlo poniéndolo dos veces: _{% trans %}Percent: %percent%%%{% endtrans %}_.

También puedes especificar el dominio del mensaje y pasar algunas variables adicionales:

```
{% trans with {'%name%': 'Fabien'} from "app" %}Hello %name%{% endtrans %}

{% trans with {'%name%': 'Fabien'} from "app" into "fr" %}Hello %name%{% endtrans %}

{% transchoice count with {'%name%': 'Fabien'} from "app" %}
    {0} %name%, there are no apples|{1} %name%, there is one apple|]1,Inf[ %name%, there are %count% apples
{% endtranschoice %}
```

Los filtros _trans_ y _transchoice_ pueden emplearse para traducir _variable texts_ y expresiones complejas:

```
{{ message|trans }}

{{ message|transchoice(5) }}

{{ message|trans({'%name%': 'Fabien'}, "app") }}

{{ message|transchoice(5, {'%name%': 'Fabien'}, 'app') }}
```

Utilizar las etiquetas de traducción o los filtros tienen el mismo efecto, pero con una pequeña diferencia: el escapado automático sólo se aplica a traducciones que emplean el filtro. Por lo que si quieres asegurarte de que tu mensaje no es _output escaped_, debes aplicar el filtro _raw_ después del filtro de traducción:

```
{# los textos entre etiquetas nunca se traducen #}
{% trans %}
    <h3>foo</h3>
{% endtrans %}

{% set message = '<h3>foo</h3>' %}

{# strings y variables traducidas a través de un filtro se escapan por defecto #}
{{ message|trans|raw }}
{{ '<h3>bar</h3>'|trans|raw }}
```

Puedes **establecer el dominio de traducción en una template Twig** con una etiqueta:

```
{% trans_default_domain "app" %}
```

esto sólo afecta a la template actual, no a ninguna template incluída (para evitar efectos secundarios).

### <a id="LocalizacionDeLosArchivosDeTraduccion" name="LocalizacionDeLosArchivosDeTraduccion"></a>4\. Localización de los archivos de traducción

Symfony busca archivos de mensajes (como traducciones) en las siguientes localizaciones por defecto:

*   El directorio app/Resources/translations
*   El directorio app/Resources/<nombre del bundle>/translations
*   El directorio Resources/translations/ dentro de cualquier bundle

Las localizaciones anteriores están en orden de prioridad. Puedes sobreescribir los mensajes de traducción de un bundle en cualquiera de los 2 directorios superiores.

El mecanismo de sobreescribir funciona a nivel de keys: sólo las keys sobreescritas han de ser listadas en el archivo de mensajes de prioridad superior. Cuando un key no se encuentra en un archivo de mensajes, el translator automáticamente buscará en los archivos de mensajes de menor prioridad.

El nombre del archivo es también importante: cada archivo debe nombrarse de acuerdo con el siguiente path:_ domain.locale.loader_:

*   _domain_. Una forma opcional de organizar los mensajes en grupos (por ejemplo _admin_, _navigation_, o el _messages_ que es por defecto)
*   _locale_. El locale para el que son las traducciones (_es_ES_, _en_GB_, etc)
*   _loader_. Cómo Symfony debería cargar y analizar el archivo (_xlf_, _php_, _yml_, etc)

El loader puede ser el nombre de cualquier loader registrado. Por defecto, Symfony proporciona varios loaders, incluyendo:

*   xlf: archivo XLIFF
*   php: archivo PHP
*   yml: archivo YAML

La elección de qué loader utilizar depende de ti, es cuestión de gustos. La opción recomendable es emplear _xlf_ para las traducciones.

Puedes añadir otros directorios con la opción _paths_ en la configuración:

```
# app/config/config.yml
framework:
    translator:
        paths:
            - '%kernel.root_dir%/../translations'
```

También puedes guardar las traducciones en una base de datos, o cualquier otro método de almacenamiento, proporcionando una clase implementando la interface [LoaderInterface](http://api.symfony.com/3.0/Symfony/Component/Translation/Loader/LoaderInterface.html). [Aquí](http://symfony.com/doc/current/reference/dic_tags.html#dic-tags-translation-loader) puedes encontrar más información.

Cada vez que creas un nuevo resource de traducción (o instalas un bundle que incluye un translation resource), asegúrate de limpiar la caché de forma que Symfony pueda descubrir las nuevas translation resources:

```
php bin/console cache:clear
```

#### Fallback Translation Locales

Imagina que el locale del usuario es fr_FR y que estás traduciendo la key Symfony is great. Para encontrar la traducción en francés, Symfony comprueba resources de traducción en varios locales:

1.  Primero busca _fr_FR_ en los resources (por ejemplo _messages.fr_FR.xlf_)
2.  Si no lo encuentra, busca la traducción en resources _fr_ (_messages.fr.xlf_)
3.  Si todavía no se encuentra, Symfony emplea el parámetro de configuración _fallbacks_, que por defecto es _en_. 

Si Symfony no encuentra una traducción para el locale dado, añadirá la traducción en el archivo log.

### <a id="ManejarElLocaleDelUsuario" name="ManejarElLocaleDelUsuario"></a>5\. Manejar el locale del usuario

El locale del usuario actual se guarda en el request y es accesible a través del objeto _request_:

```
use Symfony\Component\HttpFoundation\Request;

public function indexAction(Request $request)
{
    $locale = $request->getLocale();
}
```

Para establecer el _locale_ del usuario, podrías querer crear un event listener de forma que se establezca antes que otras partes del sistema (como el translator) lo necesiten:

```
public function onKernelRequest(GetResponseEvent $event)
{
    $request = $event->getRequest();

    // alguna lógica para determinar el $locale
    $request->setLocale($locale);
}
```

Si quieres ver cómo establecer el locale del usuario permanente en la sesión, puedes leer este [artículo](http://symfony.com/doc/current/cookbook/session/locale_sticky_session.html).

Establecer el locale empleando _$request->setLocale()_ en el controller es muy tarde para que afecte al translator. Establece el locale a través de un listener (como antes), [a través de URL](https://diego.com.es/establecer-el-locale-del-usuario-en-symfony) o llama directamente a _setLocale()_ en el translator service.

Puedes garantizar que un locale se establece en cada request de usuario definiendo el parámetro _default_locale_:

```
# app/config/config.yml
framework:
    default_locale: en
```

### <a id="TraducirConstraints" name="TraducirConstraints"></a>6\. Traducir constraints

Si empleas constraints de validación con el **componente Form**, traducir los mensajes de error es fácil: simplemente crea un translation resource para el validators domain. 

Para empezar, supón que has creado un objeto PHP que necesitas usar en algún otro lado en la aplicación:

```
// src/AppBundle/Entity/Author.php
namespace AppBundle\Entity;

class Author
{
    public $name;
}
```

Añade constraints a través de cualquiera de los métodos soportados. Establece la opción message al source text de la traducción. Por ejemplo para garantizar que la propiedad $name no está vacía, añade lo siguiente:

```
// src/AppBundle/Entity/Author.php
use Symfony\Component\Validator\Constraints as Assert;

class Author
{
    /**
     * @Assert\NotBlank(message = "author.name.not_blank")
     */
    public $name;
}
```

Crea un archivo de traducción bajo el catálogo _validators_ para los mensajes de constraint, típicamente en el directorio _Resources/translations_ del bundle:

```
<!-- validators.en.xlf -->
<?xml version="1.0"?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
    <file source-language="en" datatype="plaintext" original="file.ext">
        <body>
            <trans-unit id="author.name.not_blank">
                <source>author.name.not_blank</source>
                <target>Please enter an author name.</target>
            </trans-unit>
        </body>
    </file>
</xliff>
```

### <a id="DepurarTraducciones" name="DepurarTraducciones"></a>7\. Depurar traducciones

Cuando se mantiene un bundle, podrías utilizar o remover el uso de un mensaje de traducción sin actualizar todos los catálogos de mensajes. El comando _debug:translation_ te ayuda a encontrar estos mensajes perdidos o inutilizados para un locale dado. Muestra una tabla con el resultado cuando se traduce el mensaje en el locale dado y el resultado cuando se usaría el _fallback_. Además, también te muestra cuando la traducción es la misma que la traducción del fallback (esto podría indicar que el mensaje no ha sido correctamente traducido).

Gracias a los extractores de mensajes, el comando detectará la etiqueta translation o filtrará el uso en las templates Twig:

```
{% trans %}Symfony2 is great{% endtrans %}

{{ 'Symfony2 is great'|trans }}

{{ 'Symfony2 is great'|transchoice(1) }}

{% transchoice 1 %}Symfony2 is great{% endtranschoice %}
```

Los extractores no pueden inspeccionar los mensajes traducidos fuera de las templates lo que significa que los usos del tranlator en etiquetas form o dentro de los controllers no serán detectados. Las traducciones dinámicas que envuelven variables o expresiones no se detectan en las templates, lo que significa que este ejemplo no será analizado:

```
{% set message = 'Symfony2 is great' %}
{{ message|trans }}
```

Supón que el _default_locale_ de tu aplicación es _fr_ y que has configurado _en_ como fallback, y supón que ya has establecido algunas traducciones para el locale fr dentro de AcmeDemoBundle:

```
<!-- src/Acme/AcmeDemoBundle/Resources/translations/messages.fr.xliff -->
<?xml version="1.0"?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
    <file source-language="en" datatype="plaintext" original="file.ext">
        <body>
            <trans-unit id="1">
                <source>Symfony2 is great</source>
                <target>J'aime Symfony2</target>
            </trans-unit>
        </body>
    </file>
</xliff>
```

y para el locale _en_:

```
<!-- src/Acme/AcmeDemoBundle/Resources/translations/messages.en.xliff -->
<?xml version="1.0"?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
    <file source-language="en" datatype="plaintext" original="file.ext">
        <body>
            <trans-unit id="1">
                <source>Symfony2 is great</source>
                <target>Symfony2 is great</target>
            </trans-unit>
        </body>
    </file>
</xliff>
```

Para inspeccionar todos los mensajes en el locale _fr_ para el AcmeDemoBundle, ejecuta:

```
php bin/console debug:translation fr AcmeDemoBundle
```

Obtendrás un output como el siguiente:

![Translation debugging 1](http://symfony.com/doc/current/_images/debug_1.png)

Indica que el mensaje _Symfony 2 is great_ no es utilizado porque es traducido pero no lo has utilizado en ningún lado todavía.

Ahora, si traduces el mensajes en una de las templates, obtendrás este output:

![Translation debugging 2](http://symfony.com/doc/current/_images/debug_2.png)

El estado es vacío, lo que significa que el mensaje es traducido en el locale _fr_ y utilizado en una o más plantillas.

Si borras el mensaje _Symfony is great_ de tu archivo de traducción para el locale _fr_ y ejecutas el comando, obtendrás:

![Translation debugging 3](http://symfony.com/doc/current/_images/debug_3.png)

El estado indica que el mensaje no se encuentra porque no está traducido en el locale _fr_ pero todavía se usa en la template. Además, el mensaje en el locale _fr_ equivale al mensaje en el locale _en_. 

Si copias el contenido del archivo de traducción en el locale en al archivo de traducción en el locale fr y ejecutas el comando, obtendrás:

![Translation debugging 4](http://symfony.com/doc/current/_images/debug_4.png)

Puedes ver que las traducciones del mensaje son idénticas en los locales fr y en lo que significa que este mensaje probablemente fue copiado del francés al inglés y quizás olvidaste traducirlo. 

Por defecto se inspeccionan todos los dominios, pero puedes especificar sólo un dominio:

```
php bin/console debug:translation en AcmeDemoBundle --domain=messages
```

Cuando los bundles tienen muchos mensaje, es útil mostrar sólo los inutilizados, empleado los siguientes comandos:

```
php bin/console debug:translation en AcmeDemoBundle --only-unused
php bin/console debug:translation en AcmeDemoBundle --only-missing
```