Una route puede configurarse para coincidir con sólo **wildcards** concretos (a través de **expresiones regulares**), **métodos HTTP** o **nombres de host**. Pero el sistema de routing también se puede extender para tener una flexibilidad muy elevada empleando **condiciones**:

```
contact:
    path:     /contact
    defaults: { _controller: AcmeDemoBundle:Main:contact }
    condition: "context.getMethod() in ['GET', 'HEAD'] and request.headers.get('User-Agent') matches '/firefox/i'"
```

La condición es una expresión, escrita según el componente [ExpressionLanguage](http://symfony.com/doc/current/components/expression_language/syntax.html). Con esto, la route no coincidirá a no ser que el **método HTTP** sea **GET** o **HEAD** y si el **User-Agent** tenga la palabra _firefox_.

Puedes establecer lógicas más complejas en la expresión proporcionando dos variables que se pasarán a la expresión.

**context**

Una instancia de [RequestContext](http://api.symfony.com/3.0/Symfony/Component/Routing/RequestContext.html), que almacena información fundamental sobre la route: _baseUrl_, _pathInfo_, _method_, _host_, _scheme_, _httpPort_, _httpsPort_, _queryString_.

**request**

El objeto [Request](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Request.html) de Symfony.

Las condiciones no se tienen en cuenta cuando se genera una URL.

Las expresiones se compilan a PHP puro. El ejemplo anterior generaría lo siguiente:

```
if (rtrim($pathinfo, '/contact') === '' && (
    in_array($context->getMethod(), array(0 => "GET", 1 => "HEAD"))
    && preg_match("/firefox/i", $request->headers->get("User-Agent"))
)) {
    // ...
}
```

Por esa razón emplear la key _condition_ no causa retraso en la ejecución aparte del poco que genere el código de PHP.