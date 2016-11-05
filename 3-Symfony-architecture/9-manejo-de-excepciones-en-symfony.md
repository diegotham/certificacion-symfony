En el proceso de crear una aplicación lo normal es que ocurra algún tipo de **error** o **excepción**. Por defecto el **kernel** captura cualquier excepción e intenta encontrar una respuesta. El manejo de las **peticiones** va en un bloque **try/catch**:

```
// ../HttpKernel.php
public function handle(Request $request, $type = HttpKernelInterface::MASTER_REQUEST, $catch = true)
{
    $request->headers->set('X-Php-Ob-Level', ob_get_level());

    try {
        return $this->handleRaw($request, $type);
    } catch (\Exception $e) {
        if (false === $catch) {
            $this->finishRequest($request, $type);

            throw $e;
        }

        return $this->handleException($e, $request, $type);
    }
}
```

Cuando _$catch_ es true, se llama al método _handleException()_ y se espera crear una respuesta. El método lanza el evento **KernelEvents::EXCEPTION** (_kernel.exception_) con un objeto **GetResponseForExceptionEvent**.

```
// ../HttpKernel.php
private function handleException(\Exception $e, $request, $type)
{
    $event = new GetResponseForExceptionEvent($this, $request, $type, $e);
    $this->dispatcher->dispatch(KernelEvents::EXCEPTION, $event);

    // a listener might have replaced the exception
    $e = $event->getException();

    if (!$event->hasResponse()) {
        $this->finishRequest($request, $type);

        throw $e;
    }

    $response = $event->getResponse();

    // ... Continúa en el siguiente código
```

Los listeners para el evento _kernel.exception_ pueden establecer un objeto **Response** para esta excepción específica o reemplazar el objeto **Exception** original. Si ninguno de los listeners llama al método _setResponse()_ la excepción se lanzará de nuevo, pero esta vez no se manejará (y si _display_errors_ está activado en _php.ini_, mostraría los errores).

Si cualquiera de los listeners tiene un objeto Response, **HttpKernel** examina el objeto para establecer el **código de estado** correcto para la respuesta:

```
// the developer asked for a specific status code
if ($response->headers->has('X-Status-Code')) {
    $response->setStatusCode($response->headers->get('X-Status-Code'));

    $response->headers->remove('X-Status-Code');
} elseif (!$response->isClientError() && !$response->isServerError() && !$response->isRedirect()) {
    // ensure that we actually have an error response
    if ($e instanceof HttpExceptionInterface) {
        // keep the HTTP status code and headers
        $response->setStatusCode($e->getStatusCode());
        $response->headers->add($e->getHeaders());
    } else {
        $response->setStatusCode(500);
    }
}
// ... Continúa en el siguiente código
```

Si queremos podemos forzar a usar un código de estado concreto añadiendo un header **X-Status-Code** al objeto **Response** (aunque sólo funciona para excepciones capturadas por el **HttpKernel**) o lanzando excepciones que implementan HttpExceptionInterface. Por defecto el código de estado será 500 (Internal Server Error). Este comportamiento es mejor que el estándar de PHP, que devuelve código 200 (Ok).

Cuando un event listener establece un objeto Response, la respuesta no se maneja de forma diferente que una respuesta normal, por lo que el último paso para manejar una excepción es filtrar la respuesta. Cuando otra excepción es lanzada mientras se filtra la respuesta, la excepción será ignorada, y la respuesta sin filtrar se enviará al cliente.

```
    try {
        return $this->filterResponse($response, $request, $type);
    } catch (\Exception $e) {
        return $response;
    }
}
```