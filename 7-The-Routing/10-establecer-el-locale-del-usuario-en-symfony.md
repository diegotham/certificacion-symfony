El **locale del usuario actual** se guarda en el request y es accesible a través del objeto **Request**:

```
use Symfony\Component\HttpFoundation\Request;

public function indexAction(Request $request)
{
    $locale = $request->getLocale();
}
```

Para establecer el _locale_ del usuario puedes crear un **event listener** customizado de forma que quede establecido antes que otras partes del sistema (por ejemplo el _translator_) lo necesiten.

```
public function onKernelRequest(GetResponseEvent $event)
{
    $request = $event->getRequest();

    // lógica para determinar el $locale
    $request->getSession()->set('_locale', $locale);
}
```

Establecer el local con _$request->setLocale()_ en el controller es demasiado tarde para afectar al translator. Se establece el locale a través de un **listener** (como en el ejemplo anterior), a través de la **URL** (como se verá después) o llamando a _setLocale()_ directamente en el _translator_ **service**.

### El Locale y la URL

Ya que puedes almacenar el _locale_ del usuario en la sesión, puedes emplear la misma **URL** para mostrar un _resource_ en diferentes idiomas basándote en el locale del usuario. Por ejemplo _www.ejemplo.com/contact_ podría mostrar contenido en inglés para un usuario y en francés para otro. Desafortunadamente viola una norma fundamental de la Web: que una URL particular devuelva el mismo resource independientemente del usuario. Además, ¿Qué versión sería indexada en los motores de búsqueda?.

Una mejor forma de hacerlo es incluir el locale en la URL. Esto es soportado por el sistema de routing con el parámetro especial __locale_. 

```
# app/config/routing.yml
contact:
    path:     /{_locale}/contact
    defaults: { _controller: AppBundle:Contact:index }
    requirements:
        _locale: en|fr|de
```

Cuando se emplea el parámetro especial **_locale** en una route, el locale en concreto se establecerá en el **Request** y puede obtenerse a través del método _getLocale()_. Si por ejemplo un usuario visita el URI _/fr/contact_, el locale _fr_ se establecerá automáticamente como el locale para el request actual. 

Ahora puedes utilizar el locale para crear routes para otras páginas traducidas en tu aplicación.

Si el locale del usuario no se ha determinado se puede **establecer un locale por defecto** definiendo el parámetro **default_locale** en el _key_ framework:

```
# app/config/config.yml
framework:
    default_locale: en
```