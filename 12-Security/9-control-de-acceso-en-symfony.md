Por cada request, Symfony comprueba cada _access_control_ para encontrar una que coincida con el request. En cuanto encuentra un _access_control_ que coincide, para, y sólo se emplea el que ha encontrado primero.

Cada _access_control_ tiene varias opciones que configuran dos aspectos distintos:

### 1\. Opciones de coincidencia

**Symfony** crea una instancia de [RequestMatcher](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/RequestMatcher.html) para cada _access_control_, que determina si un control de acceso debería usarse en este request. Se emplean las siguientes opciones access_control para la comprobación: _path_, _ip_ o _ips_, _host_ y _methods_. Tomamos la siguiente configuración como ejemplo:

```
# app/config/security.yml
security:
    # ...
    access_control:
        - { path: ^/admin, roles: ROLE_USER_IP, ip: 127.0.0.1 }
        - { path: ^/admin, roles: ROLE_USER_HOST, host: symfony\.com$ }
        - { path: ^/admin, roles: ROLE_USER_METHOD, methods: [POST, PUT] }
        - { path: ^/admin, roles: ROLE_USER }
```

Por cada request, **Symfony** decidirá que _access_control_ usar basándose en el **URI**, en la **dirección IP** del cliente, en el **host** y en el **método request**. La primera regla que coincide se utiliza, y si no se especifican ni ip, ni host ni método para _/admin_, el _access_control_ permitirá cualquier ip, host, o método.

### 2\. Ejecución de acceso

Una vez que Symfony ha decidido que _access_control_ coincide (si es que coincide alguno), ejecuta las restricciones de acceso basándose en las opciones _roles_, _allow_if_ y _requires_channel_:

*   **role**. Si el usuario no tiene los roles necesarios, no se le permite el acceso (internamente se lanza una [AccessDeniedException](http://api.symfony.com/3.0/Symfony/Component/Security/Core/Exception/AccessDeniedException.html)).
*   **allow_if**. Si la expresión devuelve false, no se permite el acceso.
*   **requires_channel**. Si el channel del request (por ejemplo http) no coincide con el valor especificado (por ejemplo https), el usuario será redirigido de http a https y viceversa.

Si el acceso es denegado, el sistema intentará autenticar al usuario si no lo está ya (por ejemplo redirigiendo al usuario a la página de login). Si el usuario ya está logeado, se mostrará una **página de error 403**.

### Control de acceso por IP

Podemos controlar el acceso por IP, lo que puede ser útil si por ejemplo queremos denegar el acceso en un patrón URL a todos los requests salvo a los provenientes de un servidor interno. 

La restricción no se produce en una IP en cocreto. Mediante la key _ip_, _access_control_ sólo coincidirá con la IP proporcionada, y los usuarios que accedan desde una IP diferente continuarán a las siguientes reglas del access_control.

Un ejemplo para un patrón _/internal_:

```
# app/config/security.yml
security:
    # ...
    access_control:
        #
        - { path: ^/internal, roles: IS_AUTHENTICATED_ANONYMOUSLY, ips: [127.0.0.1, fe80::1, ::1] }
        - { path: ^/internal, roles: ROLE_NO_ACCESS }
```

Cuando el path es _/internal/something_ proveniente de una dirección IP externa 10.0.0.1:

*   La primera regla de _access_control_ se ignora ya que el **path** coincide pero la **dirección IP** no coincide con ninguna de la IPs.
*   La segunda regla _access_control_ se activa (la única restricción es el path) ya que coincide. Si ningún usuario tiene el role ROLE_NO_ACCESS, entonces el acceso será denegado. ROLE_NO_ACCESS es cualquier cosa que no coincida con un role existente, sirve como método para denegar siempre el acceso.

Pero si el mismo request viene de 127.0.0.1, ::1 o fe80::1 (ips IPv6):

*   Ahora la primera regla de control de acceso se activa ya que tanto el path como la ip coinciden. El acceso se permite ya que el usuario siempre tendrá el role IS_AUTHENTICATED_ANONYMOUSLY.
*   La segunda regla no se examina ya que la primera ha coincidido.

### Seguridad mediante una expresión

Una vez que ha coincidido _access_control_, puedes denegar el acceso a través del key _roles_ o emplear lógica más compleja con una expresión en la key _allow_if_:

```
# app/config/security.yml
security:
    # ...
    access_control:
        -
            path: ^/_internal/secure
            allow_if: "'127.0.0.1' == request.getClientIp() or has_role('ROLE_ADMIN')"
```

En este caso cuando el usuario intenta acceder a cualquier URL que comienza con _/_internal/secure_, sólo se le permitirá el acceso si la dirección IP es 127.0.0.1 o si el usuario tiene el role ROLE_ADMIN.

Dentro de la expresión tienes acceso a diferentes variables y funciones, incluyento request (el objeto [Request](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Request.html) de Symfony). Puedes ver más expresiones [aquí](http://symfony.com/doc/current/cookbook/expression/expressions.html#book-security-expression-variables).

### Forzar un channel (http, https)

Puedes también requerir que un usuario acceda a una **URL** a través de **SSL**: simplemente emplea el argumento _requires_channel_ en cualquier _access_control_. Si el _access_control_ coincide y el request emplea el **channel http**, el usuario será redirigido a **https**:

```
# app/config/security.yml
security:
    # ...
    access_control:
        - { path: ^/cart/checkout, roles: IS_AUTHENTICATED_ANONYMOUSLY, requires_channel: https }
```