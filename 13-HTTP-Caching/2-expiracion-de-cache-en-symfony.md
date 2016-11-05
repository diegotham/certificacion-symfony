El **modelo de expiración** conviene usarlo siempre que sea posible. Cuando una _response_ es cacheada con expiración, la cache guardará la _response_ y la devolverá directamente sin que llegue a la aplicación, hasta que expire. Esto modelo se emplea con los headers _Expires_ y _Cache-Control_. 

### Expiración con el headers Expires

Según la **especificación HTTP**: el campo header _Expires_ indica la fecha/hora después de que la cual la _response_ es considerada _stale_. El header _Expires_ puede establecerse con el método **Response** _setExpires()_. Toma una instancia de **DateTime** como argumento:

```
$date = new DateTime();
$date->modify('+600 seconds');

$response->setExpires($date);
```

El **header HTTP** resultante se mostrará como sigue:

```
Expires: Thu, 01 Mar 2011 16:00:00 GMT
```

El método _setExpires()_ convierte automáticamente la fecha a la **GMT timezone** tal y como se indica en la especificación.

En versiones anteriores a **HTTP 1.1** no era necesario que el servidor de origen enviara el header **Date**. Consecuentemente, la cache (del navegador por ejemplo) podría necesitar basarse en la hora local del reloj para evaluad el header _Expires_. Otra limitación del header _Expires_ es que la especificación establece que "Los servidores HTTP/1.1 no deberían enviar fechas _Expires_ de más de un año en adelante".

### Expiración con el Cache-Control Header

Debido a las limitaciones del header _Expires_, la mayoría de las veces se debería utilizar el header _Cache-Control_ en su lugar. Este header permite establecer varias directivas de cache. Para expiración existen dos: _max-age_ y _x-maxage_. La primera se utiliza por todas las caches, mientras que la segunda sólo la tienen en cuenta las caches compartidas:

```
// Establece el número de segundos después de los cuales la response
// no debería considerarse fresh
$response->setMaxAge(600);

// Igual que la anterior pero sólo para caches compartidas
$response->setSharedMaxAge(600);
```

El header Cache-Control tomaría el siguiente formato (podría tener directivas adicionales):

```
Cache-Control: max-age=600, s-maxage=600
```