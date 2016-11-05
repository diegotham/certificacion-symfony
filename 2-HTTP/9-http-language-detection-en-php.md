Language detection viene a ser esencialmente [content negotiation](http://diego.com.es/content-negotiation-en-http), utilizando el header HTTP **Accept-Language** para determinar páginas distintas en función del idioma detectado proporcionado por el user agent.

Si tenemos diferentes versiones del index en diferentes idiomas, podríamos de forma muy simple redireccionar a los usuarios en función de sus lenguajes de la siguiente forma:

```
$lang = substr($_SERVER['HTTP_ACCEPT_LANGUAGE'], 0, 2);
switch ($lang){
    case "es":
        include("index_es.php");
        break;
    case "it":
        include("index_it.php");
        break;
    case "en":
        include("index_en.php");
        break;
    default:
        // Incluye EN en cualquier idioma que no sean los anteriores:
        include("index_en.php");
        break;
}
```

Pero esta solución no tiene en cuenta las diferentes prioridades entre lenguajes. **Accept-language** es una lista de valores ponderados, por lo que simplemente mirando al primer idioma no significa que es también el preferido. Un índice _q_ con valor de 0 significa que no se acepta el lenguaje. Así que en lugar de sólo mirar al primer lenguaje, es necesario mirar los restantes y sus índices de valor. El siguiente script es una posible solución:

```
function prefered_language(array $available_languages, $http_accept_language) {

    $available_languages = array_flip($available_languages);

    $langs = [];

    preg_match_all('~([\w-]+)(?:[^,\d]+([\d.]+))?~', strtolower($http_accept_language), $matches, PREG_SET_ORDER);

    foreach($matches as $match) {

        list($a, $b) = explode('-', $match[1]) + array('', '');
        $value = isset($match[2]) ? (float) $match[2] : 1.0;

        if(isset($available_languages[$match[1]])) {
            $langs[$match[1]] = $value;
            continue;
        }

        if(isset($available_languages[$a])) {
            $langs[$a] = $value - 0.1;
        }
    }

    arsort($langs);
    return $langs;

}
```

Con el script anterior podemos obtener un array con los lenguajes y sus índices. Si recibimos un request con un HTTP_ACCEPT_LANGUAGE así:

```
$_SERVER["HTTP_ACCEPT_LANGUAGE"] = 'en-us,en;q=0.8,es-es;q=0.5,zh-cn;q=0.3,fr;q=0.1';
```

Podemos usar el script de la siguiente forma:

```
// Lenguajes que soportamos:
$available_languages = array("en", "fr", "es-es");
// Array con los idiomas
$langs = prefered_language($available_languages, $_SERVER["HTTP_ACCEPT_LANGUAGE"]);
// Mostramos el array
var_dump($langs);
/* Resultado:
array (size=3)
  'en' => float 0.8
  'es-es' => float 0.5
  'fr' => float 0.1
*/
```

Otra forma de poner solución al problema es emplear la extensión [PECL HTTP](https://pecl.php.net/package/pecl_http). Con esta extensión podremos emplear la función [http_negotiate_language](http://www.php.net/manual/en/function.http-negotiate-language.php).