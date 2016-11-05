CSRF, o [Cross-Site request forgery](http://diego.com.es/ataques-csrf-cross-site-request-forgery-en-php), es un método mediante el cual un atacante intenta hacer que los usuarios proporcionen datos sin que ellos tengan la intención de hacerlo. Los **ataques CSRF** pueden evitarse con un **CSRF token** en los formularios.

Por defecto, **Symfony** incrusta y valida CSRF tokens automáticamente. La **protección CSRF** lo que hace es añadir un campo hidden en el formulario, llamado __token_ por defecto, que contiene un valor que sólo el servidor y el usuario conocen. Esto asegura que el usuario (no otra entidad) está enviando los datos. Symfony automáticamente valida la presencia y validez del token.

El campo __token_ es un campo **hidden** que se mostrará automáticamente si incluyes la función _form_end_ en la template, lo que asegura que todos los campos no mostrados todavía se muestren.

Ya que **el token se guarda en la sesión**, una sesión se inicia automáticamente tan pronto como muestres un formulario con protección CSRF.

El token CSRF puede customizarse para algún formulario en concreto:

```
use Symfony\Component\OptionsResolver\OptionsResolver;

class TaskType extends AbstractType
{
    // ...

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class'      => 'AppBundle\Entity\Task',
            'csrf_protection' => true,
            'csrf_field_name' => '_token',
            // a unique key to help generate the secret token
            'csrf_token_id'   => 'task_item',
        ));
    }

    // ...
}
```

Para desactivar la protección CSRF, puedes poner la opción _csrf_protection_ en **false**. Las customizaciones pueden hacerse globalmente en tu proyecto. 

La opción _csrf_token_id_ es opcional pero **aumenta la seguridad del token generado** haciéndolo diferente para cada formulario.

### Configurar la protección CSRF en un formulario de login

Puedes configurar manualmente el componente de seguridad, y modificar el token provider:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        secured_area:
            # ...
            form_login:
                # ...
                csrf_token_generator: security.csrf.token_manager
```

Para emplear el provider del **componente de seguridad de Symfony** se pone **security.csrf.token_manager**.

Si ahora quieres mostrar manualmente el campo hidden en el formulario de login con el CSRF token, pueden renderizarlo. Hay que emplea la función _csrf_token_ y pasarle un **token ID**, que se pasa con _authenticate_ para el formulario de login:

```
{# src/AppBundle/Resources/views/Security/login.html.twig #}

{# ... #}
<form action="{{ path('login_check') }}" method="post">
    {# ... the login fields #}

    <input type="hidden" name="_csrf_token"
        value="{{ csrf_token('authenticate') }}"
    >

    <button type="submit">login</button>
</form>
```

Desde el archivo de configuración de seguridad también puedes cambiar algunas opciones generales:

```
# app/config/security.yml
security:
    # ...

    firewalls:
        secured_area:
            # ...
            form_login:
                # ...
                csrf_parameter: _csrf_security_token
                csrf_token_id: a_private_string
```

### Cachear páginas con protección CSRF

Los tokens CSRF han de ser diferentes para cada usuario. Es por ello que hay que tener cuidado al **cachear páginas con fomularios**, incluyendo este tipo de protección.

Normalmente a cada usuario se le asigna un **token CSRF**, que se guarda en la sesión para su validación. Esto significa que si cacheas la página con un formulario que contiene un CSRF token, cachearas el CSRF token sólo del primer usuario. Cuando un usuario envíe el formulario, el token no coincidirá con el token guardado en la sesión y los usuarios (excepto el primero) fallarán en la validación CSRF cuando envíen el formulario.

Muchas _reverse proxies_ (como **Varnish**) rechazan el cacheo de páginas con tokens CSRF. Esto es porque se envía una cookie para preservar la sesión PHP abierta y el comportamiento por defecto de Varnish es no cachear HTTP requests con cookies.

Para **cachear una página que contiene un token CSRF**, puedes usar técnicas más avanzadas de cacheo como [fragmentos ESI](http://symfony.com/doc/current/book/http_cache.html#edge-side-includes), donde cacheas la página entera e incrustas el formulario en una **etiqueta ESI sin cachear**.

Otra opción es cargar el formulario con un AJAX request sin cachear, pero cachear el resto de la respuesta HTML.

También podrías **cargar sólo el token CSRF con un AJAX request** y reemplazar el valor en el campo del formulario.