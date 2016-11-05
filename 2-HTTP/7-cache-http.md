Una de las formas más efectivas de mejorar el rendimiento de una aplicación es mediante un buen sistema de cache. **Caching**, o temporalmente guardar contenidos de requests previos, es parte del núcleo del sistema de envío de contenidos del **protocolo HTTP**. 

**Indice de contenido**

| | |
| -------- | -------- |
| 1. ¿Qué es caching? | 4. Headers de cache |
| 2. ¿Qué se puede cachear? | 5. Desarrollar una estrategia de cache |
| 3. ¿Dónde se guarda la cache? | |

### 1\. ¿Qué es caching?

Caching es guardar respuestas reusables para después poder cargar requests sucesivos de forma más rápida. Existen diferentes tipos de caching, y cada uno tiene sus propias características. Application caches y memory caches son populares por sus habilidades para mejorar la velocidad de respuestas específicas.

En este caso nos centramos en **Web caching**, característica del protocolo HTTP cuyo objetivo es minimizar el tráfico de red así como mejorar las respuestas del sistema como un todo. Las caches se encuentran en diferentes niveles en el recorrido del contenido desde el servidor hasta el navegador.

El sistema de Web caching cachea las **respuestas HTTP** para requests en base a ciertas reglas. Los requests sucesivos para contenidos cacheados pueden ser devueltos por una cache más cercana al usuario en lugar de enviar el request directamente al servidor web.

#### Beneficios del caching

Los beneficios del uso de cache son los siguientes:

*   **Reducción del uso de red**. Cuando el contenido es cacheado cerca del usuario, los requests aumentarán el uso de red más allá de la cache.
*   **Mejora de la respuesta**. El contenido cacheado se devuelve más rápido. La cache guardada en el navegador permite que la respuesta sea casi instantánea.
*   **Mejora del rendimiento en el mismo hardware**. Para el servidor desde donde se originó el contenido, mayor rendimiento puede obtenerse desde el mismo hardware con la cache. El administrador puede equilibrar el esfuerzo de los servidores en el trayecto del contenido.
*   **Disponibilidad del contenido en interrupciones de red**. La cache también puede emplearse para proporcionar contenidos al usuario incluso cuando pueden no estar disponibles por cortos periodos de tiempo desde el servidor.

#### Terminología del caching

En el caching existen diferentes términos que es conveniente conocer:

*   **Origin server**. Es la localización original del contenido en un web server.
*   **Cache hit ratio**. La efectividad de la cache se mide en términos de su **hit ratio** o **hit rate**. Este es un ratio de los requests devueltos por la cache respecto al total de requests. Un alto ratio indica que la cache devuelve gran parte del contenido, y es lo que se desea.
*   **Freshness**. Describe si un objeto en una cache es todavía válido para devolver al cliente con una medida temporal.
*   **Stale content**. Los objetos en la cache pueden expirar de acuerdo a la configuración en freshness. El contenido expirado es "stale". Generalmente el contenido expirado no puede emplearse para responder a los requests de los clientes.
*   **Validation**. Los objetos stale pueden ser validados para refrescar su tiempo de expiración. Esto implica comprobar en el origin server si el contenido cacheado todavía representa su versión más reciente.
*   **Invalidation**. Es el proceso de eliminación de contenido de la cache antes de la fecha de expiración especificada. Esto es necesario si el objeto ha cambiado en el origin server para evitar tener un objeto antiguo en la cache.

### 2\. ¿Qué se puede cachear?

Existen unos contenidos más sencillos de cachear. Los más comunes y **sencillos de cachear**: logos e imágenes de marca, imágenes no rotativas (como iconos de navegación), hojas de estilo, archivos generales de Javascript, contenido descargable y archivos multimedia.

Los contenidos anteriores tienden a cambiar cada mucho tiempo, por lo que pueden **cachearse por largos periodos**.

Otros **contenidos con los que habría que tener más cuidado a la hora de cachear**: páginas HTML, imágenes rotativas, CSS y JS modificados con frecuencia y contenidos solicitados con cookies de autenticación.

Y algunos contenidos **no deberían cachearse**: contenidos relacionados con datos sensibles (como datos bancarios) o contenido específico del usuario o que puede cambiar con frecuencia.

Se pueden establecer políticas que permiten cachear diferentes tipos de contenido de forma apropiada. Por ejemplo si los usuarios autenticados ven la misma vista de un sitio web puede ser posible cachear esa vista para todos. Si usuarios autenticados ven una vista con datos sensibles que será válida por un tiempo, puedes decir al navegador del usuario que cachee esos contenidos pero decirle a cualquier cache intermediaria que no los almacenen.

### 3\. ¿Dónde se guarda la cache?

El contenido cacheado puede guardarse en diferentes entidades por donde pasa:

*   **Cache del navegador**. Los navegadores web mantienen una cache pequeña. Normalmente el navegador establece una política que dicta los objetos más importantes para cachear.
*   **Proxies intermediarias de caching**. Cualquier infraestructura entre el cliente y tu servidor puede cachear ciertos contenidos. Estas caches pueden mantenerse por ISPs u otros individuos.
*   **Reverse cache**. Tu propio servidor puede implementar su propia cache. De esta forma el contenido puede servirse desde el punto de contacto en lugar de solicitarlo a diferentes servidores backend en cada request.

### 4\. Headers de cache

La política de cache hace que la entidad de cacheo decida si cachear o no ciertos contenidos. Puede decidir si cachear menos de lo que tiene permitido cachear, pero nunca más.

La mayoría del comportamiento de caching se determina por la política de caching, establecida por el administrador del contenido mediante HTTP headers. Los headers empleados para la cache son los siguientes:

*   **Expires**. Establece un tiempo en el futuro en el que el contenido expirará. Después de esa fecha cualquier contenido solicitado se pedirá directamente al servidor. 
*   **Cache-control**. Más moderno y flexible que Expires.
*   **Etag**. Utilizado con cache validation. El origin server puede proporcionar un Etag único para un objeto cuando proporciona inicialmente el contenido. Cuando una cache necesita validar si el contenido es actual, comprueba en el servidor si el Etag es el mismo. Si es así, la cache devolverá el contenido, sino, el contenido será devuelto por el servidor, que establecerá un nuevo Etag para el nuevo contenido.
*   **Last-Modified**. Establece la última vez que el contenido fue modificado. Puede emplearse como parte de la estrategia de validación para asegurar que el contenido es fresco.
*   **Content-Length**. No es específicamente para cache, pero se emplea a la hora de establecer políticas de cache.
*   **Vary**. Una cache normalmente utiliza el host solicitado y el directorio al resource como llave para guardar un objeto de cache. El header Vary puede usarse para decir a las caches que presten atención a un header adicional cuando deciden si un request es para el mismo objeto, lo que permite guardar diferentes versiones del mismo contenido.
    Esto es empleado normalmente para decir a las caches que observen también el header Accept-Encoding de forma que la cache sabrá diferenciar entre contenido comprimido y no comprimido. No conviene usarlo con headers sin estándares, como User-Agent, ya que puede generar muchas versiones del mismo contenido y producir un cache hit ratio muy bajo.

#### El impacto de Cache-Control en caching

Diferentes políticas pueden establecerse con este header con múltiples instrucciones separadas por comas:

*   **no-cache**. Cualquier contenido cacheado debe ser revalidado en cada request antes de ser devuelto al cliente. Esto marca como "stale" el contenido inmediatamente, aunque se pueden usar técnicas de revalidación para evitar descargar el mismo contenido continuamente.
*   **no-store**. El contenido no puede cachearse de ninguna forma (utilizado para contenido sensible).
*   **public**. El contenido puede ser cacheado por el navegador y por cualquier cache intermediaria. Para requests que utilizan autenticación HTTP, las respuestas se marcan como _private_ por defecto. Este header sobreescribe esa configuración.
*   **private**. El contenido puede ser guardado por el navegador del usuario, pero no por intermediarios. Esto se emplea para contenido específico de usuario.
*   **max-age**. Tiempo máximo que el contenido puede ser cacheado antes de ser revalidado o refrescado desde el origin server. Sustituye al header Expires en navegadores modernos y es la base para determinar el freshness. Toma su valor en segundos con un valor máximo de un año (31.536.000 segundos).
*   **s-maxage**. Similar a max-age, indica la cantidad de tiempo que el contenido puede ser cacheado. La diferencia es que sólo se aplica a caches intermediarias.
*   **must-revalidate**. Indica que la información de freshness indicada por max-age, s-maxage o Expires deben de ser obedecidas estrictamente. Contenido "stale" no puede servirse bajo ninguna circunstancia. Esto previene de usar contenido cacheado en cado de interrupciones de red o escenarios similares.
*   **proxy-revalidate**. Funciona igual que la anterior pero sólo se aplica a proxies intermediarias.
*   **no-transform**. Le dice a las caches que no pueden modificar el contenido recibido por motivos de rendimiento bajo ninguna circunstancia (por ejemplo para comprimir el contenido).

### 5\. Desarrollar una estrategia de cache

En un mundo perfecto todo podría cachearse y los servidores sólo serían contactados para proporcionar información de validaciones de forma ocasional. Esto normalmente no ocurre en la práctica, por lo que se deben precisar las políticas de cache para equilibrar las diferentes situaciones.

#### Problemas comunes

Hay muchas ocasiones en las que caching no debería emplearse debido a **cómo se produce el contenido** (generado dinámicamente por usuario) o la **naturaleza del contenido** (información bancaria sensible). Otras veces versiones antiguas del contenido están cacheadas sin ser stale, incluso cuando nuevas versones ya se han publicado.

Estas situaciones son comunes y pueden tener impactos en el rendimiento y la precisión del contenido que se sirve, pero con políticas adecuadas es posible anticiparse a estos problemas.

#### Recomendaciones

Estas son medidas a tener en cuenta a la hora de desarrollar una política de cache para mejorar el cache hit ratio:

*   **Establecer directorios específicos para imágenes, css y contenido compartido**. Colocar el contenido en directorios dedicados permite referenciarlos fácilmente desde cualquier página.
*   **Utilizar la misma URL para los mismos objetos**. Ya que las caches identifican los contenidos por el host y la ruta, conviene asegurarse de que siempre es la misma para el mismo contenido.
*   **Utilizar image sprites siempre que sea posible**. Para iconos y navegación es muy recomendable cachearlos agrupados en sprites.
*   **Almacenar scripts y fuentes externas localmente siempre que sea posible**. Si se utilizan scripts de fuentes externas u otros resources externos considera almacenarlos en tu propio servidor. 
*   **Identificadores en los objetos de cache**. Para contenidos estáticos como archivos CSS y Javascript, puede ser apropiado añadirles un identificador único, de forma que si se modifica el resource, un nuevo resource puede ser solicitado, causando que los requests eviten la cache. En Symfony esto es muy sencillo.

A la hora de seleccionar los headers correctos para diferentes objetos, lo siguiente puede servir como referencia general:

*   **Permite a todos los caches guardar assets generales**. 
*   **Permite a los navegadores cachear assets específicos de usuarios**.
*   **Realiza excepciones para contenido esencial sensible al tiempo**. Realiza excepciones en las dos medidas anteriores de forma que los contenidos caducados no se sirvan en situaciones críticas.
*   **Siempre utiliza validaciones**. Esto permite refrescar el contenido stale sin tener que descargar de nuevo todo el contenido del servidor.
*   **Utiliza largos periodos de freshness para contenidos de apoyo**. Elementos que sirven de apoyo al contenido deberían tener una configuración de freshness más larga. Normalmente imágenes y CSS cargados en un HTML.
*   **Utiliza cortos periodos de freshness para contenidos principales**. Para que el esquema funcione correctamente, el objeto contenedor debe tener un largo periodo de freshness o ni siquiera ser cacheado. Esto es normalmente el archivo HTML que llama a los diferentes resources.

Lo ideal es poder configurar una cache agresiva y realizar invalidaciones para los cambios, resultando una división de objetos de la siguiente forma:

*   Objetos cacheados de forma agresiva.
*   Objetos cacheados con periodos cortos de freshness y la habilidad para revalidarse.
*   Objetos que no deben de ser cacheados nunca.

El objetivo es intentar mover los contenidos al primer punto siempre que sea posible manteniendo el nivel de precisión.