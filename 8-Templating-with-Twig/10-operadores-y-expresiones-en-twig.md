Las **expresiones en Twig** funcionan de forma similar a como lo hacen en **PHP**. 

El **orden de operadores** es como sigue, ordenados de menor a mayor precedencia:

| | | |
| -------- | -------- |
| 1\. b-and | 11\. in | 21\. // |
| 2\. b-xor | 12\. matches | 22\. % |
| 3\. or | 13\. starts with | 23\. is |
| 4\. and | 14\. ends with | 24\. ** |
| 5\. == | 15\. .. | 25\. \| |
| 6\. != | 16\. + | 26\. [] |
| 7\. < | 17\. - | 27\. . |
| 8\. > | 18\. ~ | |
| 9\. >= | 19\. * | | 
| 10\. <= | 20\. / | |

Ejemplo de **orden de precedencia**:

```
{% set greeting = 'Hello ' %}
{% set name = 'Fabien' %}

{{ greeting ~ name|lower }}   {# Hello fabien #}

{# emplea paréntesis para cambiar la precedencia #}
{{ (greeting ~ name)|lower }} {# hello fabien #}
```

### Literals

La forma más simple de expresiones son los _literals_, que son representaciones de PHP types como strings, números y arrays. Existen los siguientes literals:

*   **"Hola Mundo"**. Todo lo que vaya entre **comillas simples o dobles** es un _string_. Se emplean siempre que necesites un string en la template (argumentos de llamadas a funciones, filtros o simplemente para extender o incluir una template). Un string puede contener un delimitador si es precedido por un **backslash** (\), como: 'It\'s ok'.
*   **32 / 32.45**. Integers y floats. Si el número presenta un punto se trata de un _float_, sino es un _integer_.
*   **["foo", "bar"]**. Los _arrays_ son definidos como una secuencia de expresiones separadas por una coma "," y envueltas por corchetes [].
*   **{"foo" : "bar"}**. Los _hashes_ se definen como una lista de keys y values separados por dos puntos ":" y envueltos por llaves {}.

```
{# keys como string #}
{ 'foo': 'foo', 'bar': 'bar' }

{# keys como nombres (equivale al hash anterior) #}
{ foo: 'foo', bar: 'bar' }

{# keys como integer #}
{ 2: 'foo', 4: 'bar' }

{# keys como expresiones (la expresión debe ir entre paréntesis) #}
{ (1 + 1): 'foo', (a ~ 'b'): 'bar' }
```

*   **true / false**. Representan valores booleanos normales.
*   **null**. Representa un valor no especificado. Se devuelve null cuando una variable no existe. _none_ es un alias de _null_.

Arrays y hashes pueden ir anidados:

```
{% set foo = [1, {"foo": "bar"}] %}
```

Emplear dobles comillas o simples no tiene impacto en el rendimiento pero **string interpolation** sólo está soportado por comillas dobles.

### Matemática

Twig te permite calcular con valores. Su uso no es muy frecuente en las templates.

*   +. Operador suma. {{ 1 + 1 }} es 2.
*   -. Operador resta. {{ 4 - 2 }} es 2.
*   /. Operador división. Devuelve un _float_ en caso de no ser divisible. {{ 1 / 2 }} es {{ 0.5 }}.
*   %. Operador módulo. {{ 12 % 8 }} es 4.
*   //. Divide dos números y devuelve el _integer_ del resultado. {{ 20 / 6 }} es 3.
*   *. Operador multiplicación. {{ 3 * 3 }} es 9.
*   **. Operador exponencial. {{ 3 ** 3 }} es 27.

### Lógica

Puedes combinar múltiples expresiones con los siguientes operadores:

*   and. Devuelve true si los operandos izquierdo y derecho son ambos true.
*   or. Devuelve true si el operando izquierdo o derecho (o ambos) son true.
*   not. Niega una sentencia.
*   (_expresión_). Agrupa una expresión.

Los operadores son sensibles a mayúsculas y minúsculas.

### Comparadores

Los siguientes operadores de comparación se pueden emplear en cualquier expresión: ==, !=, <, >, >= y <=.

Puedes también comprobar si un string empieza (_starts with_) o termina (_ends with_) con otro string:

```
{% if 'Fabien' starts with 'F' %}
{% endif %}

{% if 'Fabien' ends with 'n' %}
{% endif %}
```

Para comparaciones complejas de _strings_, el operador _matches_ permite el uso de [expresiones regulares](http://diego.com.es/expresiones-regulares-en-php):

```
{% if phone matches '/^[\\d\\.]+$/' %}
{% endif %}
```

### Operador de contención

El operador _in_ comprueba si un elemento pertenece a otro. Devuelve **true** en ese caso:

```
{# devuelve true #}

{{ 1 in [1, 2, 3] }}

{{ 'cd' in 'abcde' }}
```

Puedes emplear este filtro para comprobar la pertenencia de elementos en _strings_, _arrays_ u _objetos_ que implementan la interface **Traversable**.

También existe su versión negativa, _not in_:

```
{% if 1 not in [1, 2, 3] %}

{# is equivalent to #}
{% if not (1 in [1, 2, 3]) %}
```

### Operador test

El operador is lleva a cabo un test. Los tests se emplean para comprobar una variable frente a una expresión. 

```
{# comprueba si una variable es par #}

{{ name is odd }}
```

Los tests pueden aceptar argumentos también:

```
{% if post.status is constant('Post::PUBLISHED') %}
```

Los tests tienen versión negativa, _is not_:

```
{% if post.status is not constant('Post::PUBLISHED') %}

{# is equivalent to #}
{% if not (post.status is constant('Post::PUBLISHED')) %}
```

### Otros operadores

*   |  Aplica un **filtro**.
*   ..  Crea una secuencia basada en el operador de antes y el de después (equivale a la [función range](http://diego.com.es/funciones-en-twig#range)).

```
{{ 1..5 }}

{# es equivalente a #}
{{ range(1, 5) }}
```

Hay que emplear **paréntesis** cuando se combine con el **operador de filtrado** debido a la precedencia de operadores.

```
(1..5)|join(', ')
```

*   ~  Convierte todos los operandos en strings y los concatena. 

```
{# Si name es John :#}
{{ "Hello " ~ name ~ "!" }}
{# Devuelve Hello John! #}
```

*   . , []  Obtiene un atributo de un objeto.
*   ?:  El **operador ternario**.

```
{{ foo ? 'yes' : 'no' }}

{{ foo ?: 'no' }} is the same as {{ foo ? foo : 'no' }}
{{ foo ? 'yes' }} is the same as {{ foo ? 'yes' : '' }}
```

*   ??  El operador de no coalescencia.

```
{# devuelve el valor de foo si está definido y no es null, sino devuelve 'no' #}
{{ foo ?? 'no' }}
```

### Interpolación de strings

La **interpolación de strings** _#{expression}_ permite a cualquier expresión válida aparecer dentro de un string con dobles comillas. El resultado de evaluar la expresión se inserta en el string:

```
{{ "foo #{bar} baz" }}
{{ "foo #{1 + 2} baz" }}
```