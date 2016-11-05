**Content negotiation** es un mecanismo definido en el protocolo HTTP que le permite servir diferentes versiones de un _resource_ en un mismo URI, de forma que los user agents pueden especificar qué versión les va mejor. Un uso común de esta funcionalidad es servir una imagen en formatos **GIF** o **PNG**, por lo que si un navegador no puede mostrar imágenes PNG, servirá la versión GIF.

Cuando un user agent envía un request a un servidor, el user agent informa al servidor qué media types entiende con notas acerca de cómo de bien los entiende. Es decir, el user agent proporciona un **header HTTP Accept** que lista los **media types** y los factores de calidad asociados. El servidor entonces puede proporcionar la versión del _resource_ que concuerda más con las necesidades del user agent:
```
Accept: text/html; q=1.0, text/*; q=0.8, image/gif; q=0.6, image/jpeg; q=0.6, image/*; q=0.5, */*; q=0.1
```

En este ejemplo anterior el navegador acepta varios media types, prefiriendo HTML frente a texto plano u otros tipos, y prefiriendo GIF o JPEG sobre otros metia types, pero también permitiendo otros media types como última opción.

**Cualquier response conteniendo un body puede estar sujeto a negociación, incluyendo error responses**.

Hay dos tipos de content negotiation posibles en HTTP: server-driven y agent-driven negotiation. Estos dos tipos pueden usarse por separado o combinados.

### Server-driven Negotiation

Si la selección de la mejor representación para un response es realizada por un logaritmo localizado en el servidor, se trata de **server-driven negotiation**. La selección se basa en las representaciones disponibles del response (las dimensiones sobre las que puede variar, como el lenguaje, la codificación de contenido, etc) y los contenidos de campos de headers particulares en el mensaje request u otra información perteneciente al request (como la dirección de red del cliente).

Server-driven negotiation es útil cuando el algoritmo para seleccionar entre las diferentes representaciones es difícil de describir al user agent, o cuando el servidor desea enviar su "mejor conjetura" al cliente junto con la **primera respuesta** (confiando en evitar requests subsecuentes si la "mejor conjetura" es suficientemente buena para el usuario). Para **mejorar la conjetura del servidor**, el user agent puede incluir diferentes request headers (Accept, Accept-Languaje, Accept-Encoding, etc) que describen las preferencias para la respuesta.

#### Desventajas del Server-driven Negotiation

*   Es imposible para el servidor establecer de forma precisa lo que sería mejor para cada usuario.
*   Hacer que el user agent describa sus capacidades en cada request puede ser **ineficiente** (teniendo en cuenta que sólo un pequeño porcentaje de las respuestas tienen múltiples implementaciones) y una potencial **violación de la privacidad del usuario**.
*   Puede limitar las habilidades de **public caches** a la hora de usar la misma respuesta para múltiples requests de usuarios.

**HTTP 1.1** incluye los siguientes **request headers para facilitar el server-driven negotiation**:

*   Accept
*   Accept-Charset
*   Accept-Encoding
*   Accept-Language
*   User-Agent

El **header Vary** puede usarse para expresar los parámetros que el servidor utiliza para seleccionar una representación para el server-driven negotiation.

### Agent-driven Negotiation

En este caso la selección de la mejor representación para una respuesta se realiza por el user agent después de recibir una respuesta inicial del origin server. La selección se basa en una lista de representaciones disponibles de la respuesta incluída en los campos header o en el body de la respuesta inicial, donde cada representación se identifica por su propia URI. La selección entre las diferentes representaciones podría realizarse automáticamente (si el user agent puede hacerlo) o manualmente por el usuario seleccionando las opciones de un menú.

Agent-driven negotiation es útil cuando las respuesta puede variar entre opciones comunes (como lenguaje o codificación), cuando el origin server no es capaz de determinar las capacidades del user agent al examinar el request, y generalmente cuando se usan public caches para distribuir la carga del servidor y reducir el uso de red.

#### Desventajas del Agent-Driven Negotiation

*   Necesita un segundo request para obtener la mejor representación. Esta segunda representación es sólo eficiente cuando se usa caching.
*   No define ningún mecanismo concreto para la selección automática (aunque tampoco previene de ningún mecanismo posible de ser desarrollado como extensión y utilizarla dentro de HTTP 1.1).

HTTP 1.1 define los códigos de estado 300 (Múltiple choice) y 406 (Not Acceptable) para activar el agent-driven negotiation cuando el servidor no es capaz de porporcionar una respuesta múltiple con server-driven negotiation.

### Transparent Negotiation

Transparent Negotiation es una combinación de server-driven y agent-driven negotiation. Cuando una cache es proporcionada con un formulario de la lista de representaciones disponibles de la respuesta (como en agent-driven negotiation) y las variaciones se entienden por completo en la cache, la cache es entonces capaz de realizar server-driven negotiation en nombre del origin server para requests subsecuentes en ese _resource_.

Transparent Negotiation tiene la ventaja de distribuir el trabajo de negociación que de otra forma sería necesario que fuera realizado por el origin server y también eliminar el retraso del segundo request del agent-driven negotiation cuando la cache puede correctamente adivinar la respuesta correcta.

La especificación no define ningún **mecanismo para transparent negotiation**, aunque tampoco previene de cualquier mecanismo de ser desarrollado como extensión que podría usarse dentro de HTTP 1.1.