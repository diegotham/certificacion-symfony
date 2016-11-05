Las **variables en Twig** pueden modificarse con **filtros**. Los filtros se separan con el símbolo _pipe_ (|) y pueden tener también argumentos en paréntesis. Se pueden encadenar unos con otros, de forma que el output de un filtro se aplica al siguiente.

Por ejemplo el filtro _striptags_ elimina las **etiquetas HTML** de la variable name y el filtro _title_ hace mayúscula la primera letra de cada palabra: 

```
{{ name|striptags|title }}
```

También podemos aplicar un filtro a una sección del código:

```
{% filter upper %}
    Este texto será en mayúsculas
{% endfilter %}
```

**Indice de contenido**

| | | | |
| -------- | -------- |
| 1\. abs | 9\. first | 17\. merge | 25\. sort |
| 2\. batch | 10\. format | 18\. nl2br | 26\. split |
| 3\. capitalize | 11\. join | 19\. number_format | 27\. striptags |
| 4\. convert_encoding | 12\. json_encode | 20\. raw | 28\. title |
| 5\. date | 13\. keys | 21\. replace | 29\. trim |
| 6\. date_modify | 14\. last | 22\. reverse | 30\. upper |
| 7\. default | 15\. length | 23\. round | 31\. url_encode |
| 8\. escape | 16\. lower | 24\. slice | |

### 1\. abs

Devuelve el valor absoluto:

```
{# numero = -5 #}
{{ numero|abs }}
{# devuelve 5 #}
```

### 2\. batch

Este filtro agrupa elementos devolviendo una **lista de listas** con el número dado de elementos. Se puede proporcionar un segundo parámetro para los valores que no se han podido completar:

```
{% set elementos = ['a', 'b', 'c', 'd', 'e', 'f', 'g'] %}

<table>
    {% for row in elementos|batch(3, 'Vacío') %}
        <tr>
            {% for column in row %}
                <td>{{ column }}</td>
            {% endfor %}
        </tr>
    {% endfor %}
</table>
```

El resultado formará:

```
<table>
    <tr>
        <td>a</td>
        <td>b</td>
        <td>c</td>
    </tr>
    <tr>
        <td>d</td>
        <td>e</td>
        <td>f</td>
    </tr>
    <tr>
        <td>g</td>
        <td>No item</td>
        <td>No item</td>
    </tr>
</table>
```

Los argumentos por tanto son:

*   _size_: El tamaño de la agrupación (los números float se redondearán hacia arriba).
*   _fill_: Para rellenar agrupaciones sin valores.

### 3\. capitalize

El filtro capitalize pone en mayúscula la primera letra de un texto, y las demás en minúsculas:

```
{{ 'my first car'|capitalize }}

{# outputs 'My first car' #}
```

### 4\. convert_encoding

Este filtro convierte un string de una codificación a otra. El primer argumento es la codificación de salida y el segundo la de entrada:

```
{{ data|convert_encoding('UTF-8', 'iso-2022-jp') }}
```

El filtro se basa en las librerías [iconv](http://php.net/iconv) o [mbstring](http://php.net/mbstring), por lo que por lo menos una de ellas ha de estar instalada. Si ambas lo están se usará mbstring.

### 5\. date

El filtro date produce una fecha en un formato dado:

```
{{ post.published_at|date("m/d/Y") }}
```

El formato que se proporciona es el mismo que en la **función de PHP** [date](http://php.net/manual/es/function.date.php), salvo cuando los datos que se filtran son del tipo **DateInterval**, que entonces se ha de especificar en formato [DateInterval](http://www.php.net/DateInterval.format).

El filtro date acepta _strings_ (los cuales han de estar soportados por la función [strtotime](http://www.php.net/strtotime)), instancias [DateTime](http://www.php.net/DateTime) o instancias [DateInterval](http://www.php.net/DateInterval).

Por ejemplo para mostrar la fecha actual:

```
{{ "now"|date("m/d/Y") }}
```

Para escapar valores literales al poner fechas, se usa doble barra oblícua:

```
{{ "+5 days"|date("F jS \\a\\t g:ia") }}
```

Si el valor que se pasa al filtro _date_ es _null_, devolverá la fecha actual por defecto. Si se prefiere un string vacía en lugar de la fecha actual, puedes emplear un operador ternario:

```
{{ post.published_at is empty ? "" : post.published_at|date("m/d/Y") }}
```

Si no se proporciona ningún formato, **Twig** empleará el por defecto: _F j, Y H:i_.

El segundo argumento que acepta _date_ es el **Timezone**. Por defecto se aplica el timezone por defecto (especificada en _php.ini_ o declarado en Twig) pero se puede sobreescribir de forma explícita:

```
{{ post.published_at|date("m/d/Y", "Europe/Paris") }}
```

### 6\. date_modify

El filtro _date_modify_ modifica una fecha con un string dado:

```
{{ post.published_at|date_modify("+1 day")|date("m/d/Y") }}
```

Este filtro acepta strings (soportados por _strtotime()_) o instancias **DateTime**. Puedes combinarlo fácilmente con el filtro _date_ para darle un formato específico como se ve en el ejemplo.

### 7\. default

El filtro default en una variable devuelve un valor por defecto si el valor no está definido o está vacío, y sino muestra su valor establecido:

```
{{ var|default('var no está definida') }}

{{ var.foo|default('el elemento foo de var no está definido') }}

{{ var['foo']|default('el elemento foo de var no está definido') }}

{{ ''|default('La variable que se ha pasado está vacía!')  }}
```

Cuando empleas el filtro _default_ en una expresión que utiliza variables en algunas llamadas a métodos, asegúrate de usar el filtro _default_ cuando una variable no esté definida:

```
{{ var.method(foo|default('foo'))|default('foo') }}
```

### 8\. escape

El filtro _escape_ escapa un string para su output. Soporta diferentes estrategias dependiendo del contexto.

Por defecto escapa HTML:

```
{{ user.username|escape }}
```

Puede emplearse el shortcut _e_:

```
{{ user.username|e }}
```

Puede emplearse un argumento para utilizar el filtro en otros contextos que no sean HTML, por ejemplo JS:

```
{{ user.username|escape('js') }}
{{ user.username|e('js') }}
```

Argumentos soportados: html, js, css, url y html_attr (este último escapa en contexto de atributos HTML)

Internamente emplea la **función de PHP** [htmlspecialchars](http://php.net/htmlspecialchars). 

### 9\. first

El filtro first devuelve el primer elemento de una secuencia, un mapeado o un string:

```
{{ [1, 2, 3, 4]|first }}
{# devuelve 1 #}

{{ { a: 1, b: 2, c: 3, d: 4 }|first }}
{# devuelve 1 #}

{{ '1234'|first }}
{# devuelve 1 #}
```

### 10\. format

El filtro format da formato a un string dado reemplazando los placeholders (siguiendo la notación de [sprintf](http://www.php.net/sprintf)):

```
{% set manzanas = "manzanas" %}
{{ "Prefiero %2$s a %1$s."|format(manzanas, "peras") }}
```

### 11\. join

El filtro _join_ devuelve un string que es la concatenación de los elementos de una secuencia:

```
{{ [1, 2, 3]|join }}
{# returns 123 #}
```

Se puede especificar un separador entre elementos, que por defecto es un string vacío, pero puedes definirlo como parámetro opcional:

```
{{ [1, 2, 3]|join('|') }}
{# outputs 1|2|3 #}
```

### 12\. json_encode

El filtro _json_encode_ devuelve una representación **JSON** de un valor:

```
{{ data|json_encode() }}
```

Internamente Twig emplea la función de PHP [json_encode](http://php.net/json_encode). Puede aceptar argumentos que son constantes de esta misma función:

```
{{ data|json_encode(constant('JSON_PRETTY_PRINT')) }}
```

### 13\. keys

El filtro _keys_ devuelve los keys de un array. Es útil cuando quieres iterar sobre los keys de un array:

```
{% for key in array|keys %}
    ...
{% endfor %}
```

### 14\. last

El filtro _last_ devuelve el último elemento de una secuencia, un mapeado o un string:

```
{{ [1, 2, 3, 4]|last }}
{# devuelve 4 #}

{{ { a: 1, b: 2, c: 3, d: 4 }|last }}
{# devuelve 4 #}

{{ '1234'|last }}
{# devuelve 4 #}
```

### 15\. length

El filtro _length_ devuelve el número de elementos de una secuencia o mapead, o la longitud de un string:

```
{% if users|length > 10 %}
    ...
{% endif %}
```

### 16\. lower

Convierte una variable a minúsculas:

```
{{ 'WELCOME'|lower }}

{# devuelve 'welcome' #}
```

### 17\. merge

El filtro _merge_ une un array con otro. 

```
{% set arr = [1, 2] %}

{% set arr = arr|merge(['apple', 'orange']) %}

{# arr ahora contiene [1, 2, 'apple', 'orange'] #}
```

Los valores ahora se añaden al final de los existentes. El filtro merge también funciona cuando hay conflictos:

```
{% set elementos = { 'manzana': 'fruta', 'naranja': 'fruta', 'peugeot': 'unknown' } %}

{% set elementos = elementos|merge({ 'peugeot': 'coche', 'renault': 'coche' }) %}

{# elementos ahora contiene { 'manzana': 'fruta', 'naranja': 'fruta', 'peugeot': 'coche', 'renault': 'coche' } #}
```

En los conflictos, el proceso se unión se basa en las keys: si el key no existe todavía, se añade, pero si ya existe, se sobreescribe.

Internamente, **Twig** emplea la **función de PHP** [array_merge](http://php.net/array_merge).

### 18\. nl2br

El filtro _nl2br_ inserta saltos de línea de HTML en las nuevas líneas de un string:

```
{{ "Hola.\nQue tal."|nl2br }}
{# outputs

    Hola.<br />
    Que tal.

#}
```

### 19\. number_format

El filtro _number_format_ filtra números. Envuelve a la **función de PHP** [number_format](http://php.net/number_format):

```
{{ 200.35|number_format }} {# Muestra 200 #}
{{ 9800.333|number_format(2, '.', ',') }} {# Muestra 9,800.33 #}
```

Si no se proporcionan opciones de formateo Twig empleará las opciones por defecto:

1.  0 números decimales
2.  . como separador de decimales
3.  , como separador de miles

### 20\. raw

El filtro _raw_ marca el valor como "seguro", por lo que no será escapado:

```
{% autoescape %}
    {{ var|raw }} {# var no será escapada #}
{% endautoescape %}
```

Hay que tener cuidado al utilizar el filtro _raw_ en expresiones:

```
{% autoescape %}
    {% set hello = '<strong>Hello</strong>' %}
    {% set hola = '<strong>Hola</strong>' %}

    {{ false ? '<strong>Hola</strong>' : hello|raw }} {# Hello #}
    no renderiza de la misma forma que
    {{ false ? hola : hello|raw }} {# <strong>Hello</strong> #}
    pero sí que
    {{ (false ? hola : hello)|raw }} {# Hello #}
{% endautoescape %}
```

La primera sentencia del operador ternario no se escapa, _hello_ se marca como seguro y Twig no escapa valores estáticos. En la segunda sentencia del operador ternario, incluso cuando hello se marca como seguro, hola permanece no seguro y por tanto la expresión entera. En la tercera sentencia se marca como seguro y el resultado no se escapa.

### 21\. replace

El filtro replace da formato a un string reemplazando los placeholders:

```
{% set color = 'verde' %}
{{ "Eso es %this% y %that%."|replace({'%this%': color, '%that%': "azul"}) }}
```

### 22\. reverse

El filtro _reverse_ invierte una secuencia, un mapeo o un string:

```
{% endblock %}

{% for user in users|reverse %}
    ...
{% endfor %}

{{ '1234'|reverse }}

{# devuelve 4321 #}
```

Para secuencias y mapeos, los **keys numéricos** no se mantienen. Para revetirlos también, añade **true** como argumento:

```
{% for key, value in {1: "a", 2: "b", 3: "c"}|reverse %}
    {{ key }}: {{ value }}
{%- endfor %}

{# devuelve: 0: c    1: b    2: a #}

{% for key, value in {1: "a", 2: "b", 3: "c"}|reverse(true) %}
    {{ key }}: {{ value }}
{%- endfor %}

{# devuelve: 3: c    2: b    1: a #}
```

### 23\. round

El filtro round redondea un número a una precisión dada:

```
{{ 42.55|round }}
{# devuelve 43 #}

{{ 42.55|round(1, 'floor') }}
{# devuelve 42.5 #}
```

El filtro acepta **dos argumentos opcionales**: el primero especifica la **precisión** (por defecto es 0), y el segundo especifica el **método de redondeo** (que por defecto es _common_):

*   common. redondea arriba o abajo. Si el valor es medio (como 1.5) redondea el valor hacia arriba con valores positivos (de 1.5 a 2) y negativos hacia abajo (de -1.5 a -2).
*   ceil. Redondea hacia arriba.
*   floor. Redondea hacia abajo.

### 24\. slice

El filtro _slice_ extrae una parte de una secuencia, un mapeo o un string:

```
{% for i in [1, 2, 3, 4, 5]|slice(1, 2) %}
    {# iterará sobre 2 y 3 #}
{% endfor %}

{{ '12345'|slice(1, 2) }}

{# devuelve 23 #}
```

Puedes también emplear **expresiones** válidas para el principio y el final:

```
{% for i in [1, 2, 3, 4, 5]|slice(start, length) %}
    {# ... #}
{% endfor %}
```

El filtro _slice_ funciona con la **función de PHP** [array_slice](http://php.net/array_slice) para arrays y con [mb_substr](http://php.net/manual/es/function.mb-substr.php) o [substr](http://php.net/substr) para strings.

### 25\. sort 

El filtro _sort_ ordena un array:

```
{% for user in users|sort %}
    ...
{% endfor %}
```

Internamente Twig emplea la **función PHP** [asort](http://php.net/asort) para mantener la asociación de las keys.

### 26\. split

El filtro _split_ divide un string por el delimitador especificado y devuelve una lista de strings:

```
{% set numeros = "uno,dos,tres"|split(',') %}
{# numeros contiene ['uno', 'dos', 'tres'] #}
```

Podemos aplicar un segundo argumento _length_ para delimitar un número especificado de partes:

```
{% set numeros = "uno,dos,tres,cuatro,cinco"|split(',', 3) %}
{# numeros contiene ['uno', 'dos', 'tres,cuatro,cinco'] #}
```

Si el delimitador es un string vacío, el valor se dividirá en partes iguales. El argumento length es un carácter por defecto:

```
{% set numeros = "123"|split('') %}
{# numeros contains ['1', '2', '3'] #}

{% set letras = "aabbcc"|split('', 2) %}
{# letras contiene ['aa', 'bb', 'cc'] #}
```

Internamente **Twig** emplea la **función de PHP** [explode](http://php.net/explode) o [str_split](http://php.net/str_split) (si el delimitador está vacío).

### 27. striptags

El filtro _striptags_ filtra etiquetas SGML/XML y reemplaza espacios en blanco adyacentes por un espacio:

```
{{ some_html|striptags }}
```

Internamente **Twig** emplea la función de PHP [strip_tags](http://php.net/strip_tags).

### 28\. title

El filtro _title_ devuelve un string con cada palabra empezando con mayúscula:

```
{{ 'mi primer coche'|title }}

{# outputs 'Mi Primer Coche' #}
```

### 29\. trim

El filtro _trim_ filtra especios en blanco (u otros caracteres) del principio o final de un string:

```
{{ '  Me gusta Twig.  '|trim }}

{# devuelve 'Me gusta Twig.' #}

{{ '  Me gusta Twig.'|trim('.') }}

{# devuelve '  Me gusta Twig' #}
```

Internamente Twig emplea la función de PHP [trim](http://php.net/trim). 

### 30\. upper

El filtro _upper_ convierte un valor a mayúsculas:

```
{{ 'welcome'|upper }}

{# outputs 'WELCOME' #}
```

### 31\. url_encode

El filtro _url_encode_ codifica un string con % en una **URL** o un array como **query string**:

```
{{ "path-seg*ment"|url_encode }}
{# devuelve "path-seg%2Ament" #}

{{ "string with spaces"|url_encode }}
{# devuelve "string%20with%20spaces" #}

{{ {'param': 'value', 'foo': 'bar'}|url_encode }}
{# devuelve "param=value&foo=bar" #}
```