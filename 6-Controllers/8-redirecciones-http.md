En el **protocolo HTTP** una **redirección** es una respuesta con un código de estado que comienza en 3 que provoca que el navegador muestre una página diferente. Si un cliente recibe un redirect, necesita tomar una serie de decisiones para manejar la redirección. Diferentes códigos de estado se utilizan por los clientes para: entender el objetivo de la redirección, saber cómo administrar el caching y saber qué método request usar para el request subsecuente.

HTTP/1.1 define varios [códigos de estado HTTP](http://diego.com.es/codigos-de-estado-http) para **redirecciones** (RFC 7231):

*   **300 multiple choices**. Ofrece múltiples opciones para el resource que el cliente podría seguir, como diferentes formatos de vídeo, lista de archivos con diferentes extensiones, etc.
*   **301 moved permanently**. Este y todos los requests futuros deberán redirigirse a la **URI** dada.
*   **302 found**. La especificación HTTP/1.0 requería que el cliente ejecutara una redirección temporal, pero los navegadores populares implementaron la 302 con la funcionalidad de 303 See Other. Por esta razón HTTP/1.1 añadió los **códigos de estado 303 y 307** para diferencias entre los dos comportamientos. Sin embargo, algunas **aplicaciones web** y **frameworks** utilizan el código 302 como si fuera el 303.
*   **303 see other**. La respuesta al request puede encontrarse en otra URI utilizando un **método GET**. Cuando se recibe en respuesta a un POST (o PUT/DELETE) debería asumirse que el servidor ha recibido los datos y la redirección debería tratarse con un mensaje GET separado.
*   **307 temporary redirect**. El request debería repetirse con otro URI, pero futuros requests deberán utilizar el **URI original**. En contraste a como 302 se implementó históricamente, el método request no está permitido que se cambie cuando se trata el request original. Por ejemplo, un POST request debería repetirse utilizando otro POST request.
*   **308 permanent redirect**. El request, y todos los requests futuros deberán repetirse utilizando otro URI. 307 y 308 son paralelos a los comportamientos de 302 y 301, pero no permiten que cambie el **método HTTP**. 

Tabla resumen de los códigos anteriores:

| | | | | |
| -------- | -------- |
| **Código de estado HTTP** | **Versión HTTP** | **Temporal / Permanente** | **Cacheable** | **Método request subsecuente** |
| 301 | HTTP/1.0 | Permanente | Sí | GET/POST pueden cambiar |
| 302 | HTTP/1.0 | Temporal | No por defecto | GET/POST pueden cambiar |
| 303 | HTTP/1.1 | Temporal | Nunca | Siempre GET |
| 307 | HTTP/1.1 | Temporal | No por defecto | No pueden cambiar |
| 308 | HTTP/1.1 | Permanente | Por defecto | No pueden cambiar |

Todos estos **códigos de estado** requieren que la **URL** del objetivo se facilite mediante el header **Location**. 

En PHP una **redirección 301** simple podría hacerse como sigue:

```
header('HTTP/1.1 301 Moved Permanently');
header('Location: http://www.example.com/');
exit();
```

En Symfony hay dos opciones:

* # Redireccionar externamente

```
/**
 * @Route("/myredirect", name="myredirect")
 */
public function myRedirectAction()
{
    return $this->redirect('http://www.google.com');
}
```

Que por defecto emplea el código **HTTP/1.1 302 Found**, pero puedes utilizar otro código de redirección como segundo argumento.

* # Redireccionar internamente

Para ello emplea un método, _redirectToRoute()_ que es un shortcut de un objeto [RedirectResponse](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/RedirectResponse.html). 

```
/**
 * @Route("/myredirect", name="myredirect")
 */
public function myRedirectAction()
{
    return $this->redirectToRoute('homepage', array(), 301);
}
```

El segundo parámetro acepta un _array_ de parámetros que pasar a la ruta de destino, y el tercero con qué **código de estado** redireccionar.

El código anterior equivale a este: 

```
/**
 * @Route("/myredirect", name="myredirect")
 */
public function myRedirectAction()
{
    return $this->RedirectResponse($this->generateUrl('homepage'));
}
```