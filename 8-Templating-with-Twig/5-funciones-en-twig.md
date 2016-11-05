Las **funciones** se utilizan en **Twig** para **generar contenido**. Se llaman con su nombre seguido de paréntesis () y pueden llevar argumentos.

**Indice de contenido**

| | | |
| -------- | -------- |
| 1\. attribute | 6\. dump | 11\. random |
| 2\. block | 7\. include | 12\. range |
| 3\. constant | 8\. max | 13\. source |
| 4\. cycle | 9\. min | |
| 5\. date | 10\. parent | |

### 1\. attribute

La función attribute se puede emplear para acceder a un **atributo dinámico** de una variable, es decir, a lo que pueda devolver un método, un método con sus argumentos, o un elemento concreto de un array:

```
{{ attribute(object, method) }}
{{ attribute(object, method, arguments) }}
{{ attribute(array, item) }}
```

Además se puede emplear la palabra _define_ para comprobar la existencia de un atributo dinámico:

```
{{ attribute(object, method) is defined ? 'El método existe' : 'El método no existe' }}
```

### 2\. block

Cuando una template emplea **herencia** y quieres **imprimir un bloque varias veces**, puedes emplear la función _block_:

```
<title>{% block title %}{% endblock %}</title>

<h1>{{ block('title') }}</h1>

{% block body %}{% endblock %}
```

### 3\. constant

constant devuelve el valor de la constante de un string:

```
{{ some_date|date(constant('DATE_W3C')) }}
{{ constant('Namespace\\Classname::CONSTANT_NAME') }}
```

También puedes leer constantes de instancias de objetos también:

```
{{ constant('RSS', date) }}
```

### 4\. cycle

La función _cycle_ recorre un array de valores por ciclos:

```
{% set start_year = date() | date('Y') %}
{% set end_year = start_year + 5 %}

{% for year in start_year..end_year %}
    {{ cycle(['par', 'impar'], loop.index0) }}
{% endfor %}
```

El array puede contener cualquier número de valores:

```
{% set fruits = ['manzana', 'piña', 'naranja'] %}

{% for i in 0..10 %}
    {{ cycle(fruits, i) }}
{% endfor %}
```

### 5\. date

Convierte un argumento en una fecha para permitir **comparación de fechas**:

```
{% if date(user.created_at) < date('-2days') %}
    {# hacer algo #}
{% endif %}
```

El argumento debe estar en uno de los formatos soportados por [datetime](http://php.net/manual/en/datetime.formats.php) de **PHP**.

Puedes pasar el timezone como segundo argumento:

```
{% if date(user.created_at) < date('-2days', 'Europe/Paris') %}
    {# hacer algo #}
{% endif %}
```

Si no se pasa ningún argumento, la función devuelve la fecha actual:

```
{% if date(user.created_at) < date() %}
    {# siempre! #}
{% endif %}
```

### 6\. dump

La función _dump_ vuelca información sobre una **variable de una template**. Esto es especialmente útil en **debugging** para mostrar las variables cuando la template no se comporta como se esperaba. 

```
{{ dump(user) }}
```

También puedes hacerlo en varias variables de vez, como argumentos adicionales:

```
{{ dump(user, categories) }}
```

Si no pasas ningún argumento, se mostrarán todas las variables del contexto actual:

```
{{ dump() }}
```

Internamente, Twig emplea la función PHP [var_dump](http://php.net/var_dump).

### 7\. include

La función include devuelve el contenido renderizado de una template:

```
{{ include('template.html') }}
{{ include(some_var) }}
```

Las templates incluídas tienen acceso a las variables del contexto actual. 

El contexto se pasa por defectio a la template pero puedes pasar variables adicionales:

```
{# template.html tendrá acceso a las variables del contexto actual y a las adicionales que se faciliten #}
{{ include('template.html', {foo: 'bar'}) }}
```

Puedes desactivar el acceso al contexto estableciendo la flag _with_context_ a false:

```
{# sólo la variable foo será accesible #}
{{ include('template.html', {foo: 'bar'}, with_context = false) }}
```

```
{# no habrá variables accesibles #}
{{ include('template.html', with_context = false) }}
```

Si la flag _ignore_missing_ se establece como true, **Twig** devolverá un string vacío si la template no existe:

```
{{ include('sidebar.html', ignore_missing = true) }}
```

Puedes también proporcionar una **lista de templates** de las que se comprueba su existencia antes de su inclusión. La primera template que exista se renderizará:

```
{{ include(['page_detailed.html', 'page.html']) }}
```

Si se establece _ignore_missing_, no se devolverá ninguna de las templates si ninguna existe, sino lanzará una excepción. 

Cuando se incluye una template creada por un usuario, es conveniente emplear _sandboxed_:

```
{{ include('page.html', sandboxed = true) }}
```

### 8\. max

Devuelve el valor más grande de una **secuencia o conjunto de valores**:

```
{{ max(1, 3, 2) }}
{{ max([1, 3, 2]) }}
```

Cuando se llama con un mapeo, _max_ ignora los _keys_ y sólo compara _values_:

```
{{ max({2: "e", 1: "a", 3: "b", 5: "d", 4: "c"}) }}
{# devuelve "e" #}
```

### 9\. min

Devuelve el valor más pequeño de una **secuencia o conjunto de valores**:  

```
{{ min(1, 3, 2) }}
{{ min([1, 3, 2]) }}
```

Cuando se llama con un mapeo, _min_ ignora los _keys_ y sólo compara _values_:

```
{{ min({2: "e", 3: "a", 1: "b", 5: "d", 4: "c"}) }}
{# devuelve "a" #}
```

### 10\. parent

Cuando una template utiliza **herencia**, es posible renderizar los contenidos del bloque de la madre cuando se **sobreescribe un bloque**, con la función _parent_:

```
{% extends "base.html" %}

{% block sidebar %}
    <h3>Tabla de contenidos</h3>
    ...
    {{ parent() }}
{% endblock %}
```

La llamada a _parent_ devolverá el contenido del bloque _sidebar_ definido en la template _base.html_.

### 11\. random

La función _random_ devuelve un valor al azar dependiendo del tipo de parámetro:

*   Un **elemento** al azar para una secuencia.
*   Un **carácter** al azar para un string.
*   Un **integer** al azar entre 0 y el integer establecido.

```
{{ random(['apple', 'orange', 'citrus']) }} {# ejemplo: orange #}
{{ random('ABC') }}                         {# ejemplo: C #}
{{ random() }}                              {# ejemplo: 15386094 (funciona como la función de PHP mt_rand) #}
{{ random(5) }}                             {# ejemplo: 3 #}
```

### 12\. range

Devuelve una lista conteniendo una progresión aritmética de integers:

```
{% for i in range(0, 3) %}
    {{ i }},
{% endfor %}

{# devuelve 0, 1, 2, 3, #}
```

Cuando se proporciona el tercer argumento, especifica el valor del incremento:

```
{% for i in range(0, 6, 2) %}
    {{ i }},
{% endfor %}

{# devuelve 0, 2, 4, 6, #}
```

El operador de Twig **..** es lo mismo que emplear la función _range_:

```
{% for i in 0..3 %}
    {{ i }},
{% endfor %}
```

La función _range_ funciona como la función nativa de PHP [range](http://php.net/range).

### 13\. source

La función source devuelve el contenido de una template sin renderizarla:

```
{{ source('template.html') }}
{{ source(some_var) }}
```

Cuando se establece la flag _ignore_missing_, **Twig** devolverá un string vacío si la template no existe:

```
{{ source('template.html', ignore_missing = true) }}
```