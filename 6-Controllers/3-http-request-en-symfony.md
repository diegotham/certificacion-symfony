En **PHP** la **petición** se representa mediante **variables globales** como $_GET, $_POST, $_FILES, $_COOKIE, $_SESSION, etc, y la **respuesta** se genera mediante **funciones** como _**echo**_, _**header**_, _**setcookie**_, etc.

El componente **HttpFoundation** de Symfony sustituye estas variables globales y funciones por una forma orientada a objetos.

La forma más común de crear un **request** es basándose en las **PHP global variables** con _**createFromGlobals()**_:

```
use Symfony\Component\HttpFoundation\Request;

$request = Request::createFromGlobals();
```

lo que es casi equivalente a la llamada ___construct_ de **Request**:

```
$request = new Request(
    $_GET,
    $_POST,
    array(),
    $_COOKIE,
    $_FILES,
    $_SERVER
);
```

**Indice de contenido**

1.  Acceder a los datos request
2.  Identificando un request
3.  Simulando un request
4.  Acceder a la sesión
5.  Acceder a los headers Accept-*
6.  Sobreescribir el request

### 1. Acceder a los datos Request

El **objeto Request** guarda información sobre la **petición del cliente**. Se puede acceder a esa información a través de propiedades public:

*   **request** : equivale a $_POST
*   **query**: equivale a $_GET (_$request->query->get('name')_)
*   **cookies**: equivale a $_COOKIE
*   **attributes**: no tiene equivalente, guarda otros datos
*   **files**: equivale a $_FILES
*   **server**: equivale a $_SERVER
*   **headers**: mayormente equivalente a $_SERVER

Cada **propiedad** es una **instancia de la clase ParameterBag** (o de una heredera), que es una clase que almacena los datos:

*   **request**: ParameterBag
*   **query**: ParameterBag
*   **cookies**: ParameterBag
*   **attributes**: ParameterBag
*   **files**: FileBag
*   **server**: ServerBag
*   **headers**: HeaderBag

Todas las **instancias ParameterBag** tienen **métodos para devolver y actualizar sus datos**: 

*   _all()_ devuelve los parámetros
*   _keys()_ devuelve las claves de parámetro
*   _replace()_ sustituye los parámetros actuales por unos nuevos
*   _add()_ añade parámetros
*   _get()_ devuelve un parámetro por su nombre
*   _set()_ establece un parámetro por su nombre
*   _has()_ devuelve TRUE si el parámetro está definido
*   _remove()_ elimina un parámetro

La **instancia ParameterBag** también tiene algunos **métodos para filtrar los valores**:

*   _getAlpha()_ devuelve caracteres alfabéticos del valor de parámetro
*   _getAlnum()_ devuelve caracteres alfabéticos y dígitos del valor de parámetro
*   _getDigits()_ devuelve los dígitos del valor de parámetro
*   _getInt()_ devuelve el valor de parámetro convertido a integer
*   _filter()_ filtra el parámetro mediante la función PHP [filter_var](http://php.net/manual/en/function.filter-var.php)

Todos los **getters** tienen hasta tres argumentos: el primero es el **nombre del parámetro** y el segundo es el **valor por defecto a devolver** si el parámetro no existe:

```
// la query string es '?foo=bar'

$request->query->get('foo');
// devuelve bar

$request->query->get('bar');
// devuelve null

$request->query->get('bar', 'bar');
// devuelve 'bar'
```

Cuando **PHP** importa la petición **request**, organiza los parámetros request como **foo[bar]=bar** creando un array. Así puedes coger el parámetro _foo_ y obtendrás un array con el elemento _bar_. Pero a veces es posible que necesites el valor del nombre del parámetro original: **foo[bar]**. Esto es posible con los **getters** de **ParameterBag** como _get()_, a través del tercer argumento:

```
// la petición es '?foo[bar]=bar'

$request->query->get('foo');
// devuelve el array('bar' => 'bar')

$request->query->get('foo[bar]');
// devuelve null

$request->query->get('foo[bar]', null, true);
// devuelve 'bar'
```

Gracias a la propiedad **attributes** se pueden guardar datos adicionales en el request, que es también una instancia de **ParameterBag**. Esto se usa sobre todo para adjuntar información que pertenece al **Request** y requiere ser accedida desde diferentes puntos de una aplicación.

Finalmente la información enviada con el request puede ser accedida con _getContent()_:

```
$content = $request->getContent();
```

Por ejemplo puede ser útil para procesar un **_string_ JSON** enviado a la aplicación desde un servicio remoto a través del **método HTTP POST**.

### 2. Identificando un Request

En tu aplicación necesitarás una **forma de identificar un request**. La mayoría del tiempo se hace mediante el **path info**, al que se puede acceder con _getPathInfo()_:

```
// para un request a http://example.com/blog/index.php/post/hello-world
// el path info es "/post/hello-world"
$request->getPathInfo();
```

### 3. Simulando un Request

En lugar de **crear un request** basándote en las **PHP globals**, puedes simularlo:

```
$request = Request::create(
    '/hello-world',
    'GET',
    array('name' => 'Fabien')
);
```

El método _create()_ crea un **request** basado en una URI, un **método HTTP** y algunos parámetros dependiendo del método. También se pueden sobreescribir las variables, **Symfony** crea **valores por defecto para las variables globales**.

Basándonos en el request, puedes **sobreescribir las variables PHP globales** con _overrideGlobals()_:

```
$request->overrideGlobals();
```

También se pueden **duplicar requests** con [_duplicate()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Request.html#method_duplicate) o **cambiar algunos parámetros** con una llamada a [_initialize()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Request.html#method_initialize).

### 4. Acceder a la sesión

Si tienes una **sesión** adjuntada al **request**, puedes acceder a ella a través del método [_getSession()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Request.html#method_getSession). El método [_hasPreviousSession()_](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Request.html#method_hasPreviousSession) te informa de si el request contiene una sesión que empezó en un request previo.

### 5. Acceder a los headers Accept-*

Puedes acceder a datos básicos de los headers **Accept-*** mediante los siguientes métodos:

*   _getAcceptableContentTypes()_ devuelve la lista con **content types** aceptados ordenados por calidad descendente
*   _getLanguages()_ devuelve la lista de **lenguajes** aceptados ordenados por calidad descendente
*   _getCharsets()_ devuelve la lista de **charsets** aceptados ordenados por calidad descendente
*   _getEncodings()_ devuelve la lista de **encodings** aceptados ordenados por calidad descendente

Si necesitas acceso a los datos transformados de **Accept**, **Accept-Languaje**, **Accept-Charset** o **Accept-Encoding**, puedes usar la clase **AcceptHeader**:

```
use Symfony\Component\HttpFoundation\AcceptHeader;

$accept = AcceptHeader::fromString($request->headers->get('Accept'));
if ($accept->has('text/html')) {
    $item = $accept->get('text/html');
    $charset = $item->getAttribute('charset', 'utf-8');
    $quality = $item->getQuality();
}

// Los objetos del header Accept están ordenados por calidad descendente
$accepts = AcceptHeader::fromString($request->headers->get('Accept'))
    ->all();
```

La **clase Request** tiene muchos otros métodos que puedes usar para acceder a la **información del request**. Echa un vistazo al [API request](http://api.symfony.com/2.6/Symfony/Component/HttpFoundation/Request.html) para más información.

### 6. Sobreescribir el Request

La **clase Request** no debería ser sobreescrita porque es un **objeto con datos** que representa un **mensaje HTTP**. Pero cuando se trata de un **sistema heredado**, añadir métodos o cambiar algún comportamiento por defecto podría ayudar. En ese caso, se registra un **PHP callable** que permite crear una instancia de la **clase Request**:

```
use Symfony\Component\HttpFoundation\Request;

Request::setFactory(function (
    array $query = array(),
    array $request = array(),
    array $attributes = array(),
    array $cookies = array(),
    array $files = array(),
    array $server = array(),
    $content = null
) {
    return SpecialRequest::create(
        $query,
        $request,
        $attributes,
        $cookies,
        $files,
        $server,
        $content
    );
});

$request = Request::createFromGlobals();
```