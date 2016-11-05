Los **mensajes flash** son mensajes especiales en la **sesión de usuario**. Los mensajes flash están diseñados para mostrarse sólo una vez, y desaparecer de la sesión automáticamente tan pronto como los quieras sacar. Esta funcionalidad hace a los mensajes flash particularmente útiles para **guardar notificaciones de usuarios**.

Por ejemplo, si estamos procesando un formulario:

```
use Symfony\Component\HttpFoundation\Request;

public function updateAction(Request $request)
{
    $form = $this->createForm(...);

    $form->handleRequest($request);

    if ($form->isValid()) {
        // hacer algún tipo de procesamiento

        $this->addFlash(
            'notice',
            'Tus cambios se han guardado!'
        );

        // $this->addFlash es equivalente a $this->get('session')->getFlashBag()->add

        return $this->redirectToRoute(...);
    }

    return $this->render(...);
}
```

Después de procesar el **request**, el controller establece un **mensaje flash en la sesión** y después redirige. El _message key_ (en este ejemplo es **notice**) puede ser cualquier cosa, se emplea el _key_ para hacer referencia al mensaje.

En la plantilla de la página siguiente (o incluso mejor en la **base template**), puedes leer cualquiera de los mensajes de la sesión:

```
{% for flash_message in app.session.flashbag.get('notice') %}
    <div class="flash-notice">
        {{ flash_message }}
    </div>
{% endfor %}
```

Es frecuente utilizar **notice**, **warning** y **error** como los keys para los diferentes tipos de mensajes flash, pero puedes utilizar cualquier key que quieras.

Puedes emplear el método [_peek()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_peek) también para obtener el mensaje. Éste continuará en la bolsa.

### FlashBagInterface

El propósito de [FlashBagInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html) es proporcionar una forma de establecer y devolver mensajes en las sesiones. El **workflow** normalmente sería establecer los mensajes flash en un request y mostrarlos después de una redirección de página. Como en el ejemplo anterior: un usuario envía un formulario que pasa por un update controller, y después de procesar el controller redirige la página a la página actualizada o a una página de error. Los **mensajes flash** establecidos en el request de la página anterior de mostrarán subsecuentemente en la página siguiente de esa sesión. 

*   [AutoExpireFlashBag](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/AutoExpireFlashBag.html). En esta implementación, los mensajes establecidos en una carga de página estarán disponibles para mostrarse sólo en la siguiente carga de página. Estos mensajes autoexpirarán independientemente de que se hayan devuelto o no.
*   [FlashBag](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBag.html). En esta implementación, los mensajes se mantendrán en la sesión hasta que son explícitamente obtenidos o limpiados. Esto hace posible el uso del ESI caching. 

[FlashBagInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html) tiene una API simple:

*   [add()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_add). Añade un mensaje flash a la pila del tipo específico.
*   [set()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_set). Establece flashes por tipo. Este método toma mensajes simples como _string_ y múltiples mensajes en un _array_.
*   [get()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_get). Obtiene los flashes por tipo y los limpia de la bolsa.
*   [setAll()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_setAll). Establece todos los flashes, acepta un array asociativo de arrays _type => array($messages)_.
*   [all()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_all). Obtiene todos los flashes (como un array asociativo de arrays) y limpia los flashes de la bolsa.
*   [peek()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_peek). Obtiene los flashes por tipo (sólo lectura).
*   [peekAll()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_peekAll). Obtiene todos los flashes (sólo lectura) como un array asociativo de arrays.
*   [has()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_has). Devuelve true si el tipo existe, false si no.
*   [keys()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_keys). Devuelve un array de los tipos flash guardados.
*   [clear()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html#method_clear). Limpia la bolsa.

Para aplicaciones simples es normalmente suficiente tener un **mensaje flash** por tipo, por ejemplo un **notice de confirmación** después de enviar un formulario. Sin embargo, los mensajes flash se guardan en un array asociativo por flash _$type_ lo que significa que tu aplicación puede tener varios mensajes para un mismo tipo. Esto permite a la API usarse para mensajes más complejos en la aplicación.

Ejemplo de uso con **múltiples flashes**:

```
use Symfony\Component\HttpFoundation\Session\Session;

$session = $request->getSession();

// Añadir mensajes flash
$session->getFlashBag()->add(
    'warning',
    'Cuidado! Un warning!'
);
$session->getFlashBag()->add('error', 'No se ha podido actualizar el usuario');
$session->getFlashBag()->add('error', 'Otro error');
```

Ahora podemos **mostrar los tipos de mensajes individualmente**:

```
// mostrar warnings
foreach ($session->getFlashBag()->get('warning', array()) as $message) {
    echo '<div class="flash-warning">'.$message.'</div>';
}

// mostrar errores
foreach ($session->getFlashBag()->get('error', array()) as $message) {
    echo '<div class="flash-error">'.$message.'</div>';
}
```

O utiliza un método compacto para procesar el **mostrar todos los flashes de vez**:

```
foreach ($session->getFlashBag()->all() as $type => $messages) {
    foreach ($messages as $message) {
        echo '<div class="flash-'.$type.'">'.$message.'</div>';
    }
}
```