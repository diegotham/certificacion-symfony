Una [estructura de control](http://diego.com.es/estructuras-de-control-en-php) hace referencia a las sentencias que controlan el **flujo de la aplicación**: **condicionales**, **loops** o estructuras como **bloques**. Las estructuras de control aparecen en bloques _{% ... %}_.

| | | |
| -------- | -------- |
| 1\. autoescape | 7\. flush | 13\. macro |
| 2\. block | 8\. for | 14\. sandbox |
| 3\. do | 9\. from | 15\. set |
| 4\. embed | 10\. if | 16\. spaceless |
| 5\. extends | 11\. import | 17\. use |
| 6\. filter | 12\. include | 18\. verbatim |

### 1\. autoescape

Independientemente de si el **autoescaping** está activado o no, puedes marcar una sección de una template como que se escape con la etiqueta _autoescape_:

```
{% autoescape %}
    Todo se escapará en este bloque
    con la estrategia HTML
{% endautoescape %}

{% autoescape 'html' %}
    Todo se escapará en este bloque
    con la estrategia HTML
{% endautoescape %}

{% autoescape 'js' %}
    Todo se escapará en este bloque
    con la estrategia JS
{% endautoescape %}

{% autoescape false %}
    Nada será escapado en este bloque
{% endautoescape %}
```

Dentro de un bloque _autoescape_ se puede marcar una variable con el [filtro raw](http://diego.com.es/filtros-en-twig#raw) para evitar su escape:

```
{% autoescape %}
    {{ safe_value|raw }}
{% endautoescape %}
```

### 2\. block

Los bloques _block_ se usan para **herencia** y actúan como **placeholders** y **reemplazamientos** al mismo tiempo. Se explica más detenidamente en la etiqueta _extends_. 

Los nombres de _blocks_ han de consistir en **caracteres alfanuméricos y barras bajas**. 

### 3\. do

La etiqueta _do_ funciona exactamente como la expresión **{{ ... }}** salvo que no imprime nada.

```
{% do 1 + 2 %}
```

### 4\. embed

La etiqueta _embed_ combina el comportamiento de _include_ y _extends_. Permite incluir los contenidos de otra template, como hace _include_. Pero también permite sobreescribrir cualquier bloque definido en el interior de la template incluída, como cuando se extiende una template.

Es algo así como un _sublayout_ :

```
{% embed "teasers_skeleton.twig" %}
    {# Estos bloques se definen en "teasers_skeleton.twig" #}
    {# y los sobreescribimos aquí:                    #}
    {% block left_teaser %}
        Algún contenido para la caja left_teaser
    {% endblock %}
    {% block right_teaser %}
        Algún contenido para la caja right_teaser
    {% endblock %}
{% endembed %}
```

La etiqueta embed lleva la idea de la herencia de templates al nivel de fragmentos de contenido. Mientras que la herencia de templates permite "esqueletos de documentos", que son rellenados con herederas, la etiqueta _embed_ permite crear "skeletons" para unidades más pequeñas de contenido y reusarlas y rellenarlas donde quieras.

**Ejemplo**:

Suponemos la siguiente template base:
```
┌─── page layout ─────────────────────┐
│                                     │
│           ┌── block "content" ──┐   │
│           │                     │   │
│           │                     │   │
│           │ (child template to  │   │
│           │  put content here)  │   │
│           │                     │   │
│           │                     │   │
│           └─────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

Algunas páginas ("pim" y "pam") comparten la misma estructura de contenido, dos cajas apliladas verticalmente:
```
┌─── page layout ─────────────────────┐
│                                     │
│           ┌── block "content" ──┐   │
│           │ ┌─ block "top" ───┐ │   │
│           │ │                 │ │   │
│           │ └─────────────────┘ │   │
│           │ ┌─ block "bottom" ┐ │   │
│           │ │                 │ │   │
│           │ └─────────────────┘ │   │
│           └─────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

Otras páginas ("pum" y "pem") comparten una estructura diferente, dos cajas una al lado de la otra:
```
┌─── page layout ─────────────────────┐
│                                     │
│           ┌── block "content" ──┐   │
│           │                     │   │
│           │ ┌ block ┐ ┌ block ┐ │   │
│           │ │"left" │ │"right"│ │   │
│           │ │       │ │       │ │   │
│           │ │       │ │       │ │   │
│           │ └───────┘ └───────┘ │   │
│           └─────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

En esta situación lo ideal es emplear la etiqueta _embed_. El código del layout común puede estar en una template base individual, y las dos diferentes estructuras de contenido, o "sublayouts" van en templates separadas que se embeben:

Página de la template _pim.twig_:

```
{% extends "layout_skeleton.twig" %}

{% block content %}
    {% embed "vertical_boxes_skeleton.twig" %}
        {% block top %}
            Contenido de la caja de arriba
        {% endblock %}

        {% block bottom %}
            Contenido de la caja de abajo
        {% endblock %}
    {% endembed %}
{% endblock %}
```

Ahora el código del esqueleto de las cajas verticales, _vertical_boxes_skeleton.twig_:

```
<div class="top_box">
    {% block top %}
        Top box default content
    {% endblock %}
</div>

<div class="bottom_box">
    {% block bottom %}
        Bottom box default content
    {% endblock %}
</div>
```

La etiqueta embed toma los mismos argumentos que la etiqueta include:

```
{% embed "base" with {'foo': 'bar'} %}
    ...
{% endembed %}

{% embed "base" with {'foo': 'bar'} only %}
    ...
{% endembed %}

{% embed "base" ignore missing %}
    ...
{% endembed %}
```

Ya que las templates embebidas no tienen **nombres**, las estrategias de autoescapado basadas en el nombre de archivo de la template no funcionarán correctamente si cambias el contexto (por ejemplo, si embebes una template **CSS/JavaScript** en una **HTML**). En ese caso, emplea la etiqueta _autoescape_.

### 5\. extends

La etiqueta _extends_ se emplea para extender una template de otra. 

Como PHP, Twig tampoco soporta **herencia múltiple**, por lo que sólo puede haber una extends tag por renderización. Pero sí permite reutilización horizontal con la etiqueta _use_.

Definimos una template base:

```
<!DOCTYPE html>
<html>
    <head>
        {% block head %}
            <link rel="stylesheet" href="style.css" />
            <title>{% block title %}{% endblock %} - My Webpage</title>
        {% endblock %}
    </head>
    <body>
        <div id="content">{% block content %}{% endblock %}</div>
        <div id="footer">
            {% block footer %}
                &copy; Copyright 2011 by <a href="http://domain.invalid/">you</a>.
            {% endblock %}
        </div>
    </body>
</html>
```

Hay 4 bloques _block_, que indican que cualquiera de las templates herederas pueden sobreescribir esos contenidos.

Una heredera puede ser:

```
{% extends "base.html" %}

{% block title %}Index{% endblock %}
{% block head %}
    {{ parent() }}
    <style type="text/css">
        .important { color: #336699; }
    </style>
{% endblock %}
{% block content %}
    <h1>Index</h1>
    <p class="important">
        Welcome on my awesome homepage.
    </p>
{% endblock %}
```

Cuando el sistema de templates evalúa este template, primero localiza a la madre. La etiqueta extends ha de ser la primera tag de la template.

La template hija no define ningún bloque footer, usará el de la madre.

No es posible definir dos _blocks_ con el mismo nombre.

**Posibilidades**:

*   Imprimir un _block_ varias veces con la [función block](http://diego.com.es/funciones-en-twig#block).

```
<title>{% block title %}{% endblock %}</title>
<h1>{{ block('title') }}</h1>
{% block body %}{% endblock %}
```

*   Renderizar contenidos de la template madre con la [función parent](http://diego.com.es/funciones-en-twig#parent).

```
{% block sidebar %}
    <h3>Table Of Contents</h3>
    ...
    {{ parent() }}
{% endblock %}
```

*   Puedes poner el nombre del block después de la etiqueta para mayor legibilidad.

```
{% block sidebar %}
    {% block inner_sidebar %}
        ...
    {% endblock inner_sidebar %}
{% endblock sidebar %}
```

*   Los _blocks_ pueden anidarse para _layouts_ más complejos.

```
{% for item in seq %}
    <li>{% block loop_item %}{{ item }}{% endblock %}</li>
{% endfor %}
```

*   Para blocks con poco contenido, se puede emplear una sintaxis reducida:

```
{% block title %}
    {{ page_title|title }}
{% endblock %}
```

Equivale a:

```
{% block title page_title|title %}
```

*   **Herencia dinámica**, utilizando una variable como la template base:

```
{% extends some_var %}
```

Además de poder proporcionar varias templates (la primera que exista es la que se usará como madre).

```
{% extends ['layout.html', 'base_layout.html'] %}
```

*   **Herencia condicional**, empleando por ejemplo un operador ternario.

```
{% extends standalone ? "minimum.html" : "base.html" %}
```

### 6\. filter

La etiqueta _filter_ permite aplicar [filtros de Twig](http://diego.com.es/filtros-en-twig) en un conjunto de datos.

```
{% filter upper %}
    Este texto será mayúsculas
{% endfilter %}
```

También se pueden encadenar filtros:

```
{% filter lower|escape %}
    <strong>SOME TEXT</strong>
{% endfilter %}

{# devuelve "&lt;strong&gt;some text&lt;/strong&gt;" #}
```

### 7\. flush

La etiqueta _flush_ le dice a **Twig** que haga flush en el **output buffer**:

```
{% flush %}
```

Internamente Twig emplea la función de PHP [flush](http://php.net/flush).

### 8\. for

Crea un **loop** de cada objeto en una secuencia. Si por ejemplo queremos mostrar los usuarios de una lista:

```
<h1>Members</h1>
<ul>
    {% for user in users %}
        <li>{{ user.username|e }}</li>
    {% endfor %}
</ul>
```

La secuencia puede ser un **array** u **objeto** implementando la interface **Traversable**.

Si quieres iterar sobre una secuencia de números puedes emplear el operador .. :

```
{% for i in 0..10 %}
    * {{ i }}
{% endfor %}
```

También se pueden emplear letras:

```
{% for letter in 'a'..'z' %}
    * {{ letter }}
{% endfor %}
```

E includo pueden aplicarse expresiones en ambos lados:

```
{% for letter in 'a'|upper..'z'|upper %}
    * {{ letter }}
{% endfor %}
```

Dentro del loop _for_ se puede acceder a varias **variables especiales**:

| | |
| -------- | -------- |
| **Variable** | **Descripción** |
| loop.index | Iteración actual del loop (desde 1) |
| loop.index0 | Iteración actual del loop (desde 0) |
| loop.revindex | Número de iteraciones desde el final del loop (desde 1) |
| loop.revindex0 | Número de iteraciones desde el final del loop (desde 0) |
| loop.first | True si es la primera iteración |
| loop.last | True si es la última iteración |
| loop.length | Número de items en la secuencia |
| loop.parent | Contexto de herencia |

```
{% for user in users %}
    {{ loop.index }} - {{ user.username }}
{% endfor %}
```

Las variables _loop.length_, _loop.revindex_, _loop.devindex0_ y _loop.last_ están sólo disponibles para **arrays** u **objetos** que implementan la interface **Countable**. Tampoco están disponibles cuando se hace looping con una condición.

**Posibilidades**:

*   Al contrario que con PHP, no es posible usar _break_ o _continue_ en un loop. Sin embargo, puedes filtrar la secuencia durante la iteración, lo que permite **saltar elementos**. El siguiente ejemplo salta los usuarios que no están activos:

```
<ul>
    {% for user in users if user.active %}
        <li>{{ user.username|e }}</li>
    {% endfor %}
</ul>
```

La ventaja es que el loop no contará a los usuarios que no estén activos. Recuerda que con condiciones no podríamos emplear variables como _loop.last_.

*   Si la secuencia está vacía, puedes reemplazarla por un bloque _else_:

```
<ul>
    {% for user in users %}
        <li>{{ user.username|e }}</li>
    {% else %}
        <li><em>no user found</em></li>
    {% endfor %}
</ul>
```

*   Por defecto, un loop itera sobre los _values_ de una secuencia. Puedes iterar sobre los _keys_ con el [filtro keys](http://diego.com.es/filtros-en-twig#keys):

```
<h1>Members</h1>
<ul>
    {% for key in users|keys %}
        <li>{{ key }}</li>
    {% endfor %}
</ul>
```

*   También puedes acceder a los _keys_ y _values_:

```
<h1>Members</h1>
<ul>
    {% for key, user in users %}
        <li>{{ key }}: {{ user.username|e }}</li>
    {% endfor %}
</ul>
```

*   Iterar sobre un subconjunto de valores, con el [filtro slice](http://diego.com.es/filtros-en-twig#slice):

```
<h1>Top Ten Members</h1>
<ul>
    {% for user in users|slice(0, 10) %}
        <li>{{ user.username|e }}</li>
    {% endfor %}
</ul>
```

### 9\. from

La etiqueta _from_ importa nombres _macro_ en el namespace actual. La etiquetas _import_ y _macro_ se ven más adelante.

### 10\. if

La sentencia if en **Twig** es similar a **PHP**.

```
{% if online == false %}
    <p>Our website is in maintenance mode. Please, come back later.</p>
{% endif %}
```

Puedes también comprobar si un array no está vacío:

```
{% if users %}
    <ul>
        {% for user in users %}
            <li>{{ user.username|e }}</li>
        {% endfor %}
    </ul>
{% endif %}
```

Si quieres comprobar si una variable está definida, emplea _if users is defined_ en su lugar.

**Posibilidades**:

*   Puedes usar **not** para comprobar _values_ que son **false**.

```
{% if not user.subscribed %}
    <p>You are not subscribed to our mailing list.</p>
{% endif %}
```

*   Para **múltiples condiciones**, _and_ y _or_ se pueden emplear.

```
{% if temperature > 18 and temperature < 27 %}
    <p>It's a nice day for a walk in the park.</p>
{% endif %}
```

*   Para múltiples ramas _elseif_ y _else_ se pueden emplear como en **PHP**. Aunque usar _expressions_ más complejas también.

```
{% if kenny.sick %}
    Kenny is sick.
{% elseif kenny.dead %}
    You killed Kenny! You bastard!!!
{% else %}
    Kenny looks okay --- so far
{% endif %}
```

Las reglas para determinar si una expresión es true o false son las mismas que en PHP, algunas de ellas:

| | |
| -------- | -------- |
| Valor | Booleano |
| string vacío | false |
| 0 | false |
| string con espacio en blanco | true |
| array vacío | false |
| null | false |
| array no vacío | true |
| objeto | true |

### 11\. import

Twig soporta poner código utilizado en macros. Estos macros pueden ir en diferentes templates y ser importadas desde aquí.

Hay dos formas de importar templates. Puedes importar la template entera en una variable o solicitar macros específicos de la misma.

Imagina que tenemos un módulo helper que renderiza formularios (llamado forms.html):

```
{% macro input(name, value, type, size) %}
    <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
{% endmacro %}

{% macro textarea(name, value, rows, cols) %}
    <textarea name="{{ name }}" rows="{{ rows|default(10) }}" cols="{{ cols|default(40) }}">{{ value|e }}</textarea>
{% endmacro %}
```

La forma más fácil y flexible es **importar el módulo en una variable**. De esta forma puedes acceder a los atributos:

```
{% import 'forms.html' as forms %}

<dl>
    <dt>Username</dt>
    <dd>{{ forms.input('username') }}</dd>
    <dt>Password</dt>
    <dd>{{ forms.input('password', null, 'password') }}</dd>
</dl>
<p>{{ forms.textarea('comment') }}</p>
```

De forma alternativa puedes importar nombres de la **template** en el **namespace** actual:

```
{% from 'forms.html' import input as input_field, textarea %}

<dl>
    <dt>Username</dt>
    <dd>{{ input_field('username') }}</dd>
    <dt>Password</dt>
    <dd>{{ input_field('password', '', 'password') }}</dd>
</dl>
<p>{{ textarea('comment') }}</p>
```

Para importar _macros_ del archivo actual puedes usar la variable especial __self_ para la fuente.

### 12\. include

La sentencia _include_ incluye una template y devuelve el contenido renderizado de ese archivo en el namespace actual:

```
{% include 'header.html' %}
    Body
{% include 'footer.html' %}
```

Las templates incluídas tienen acceso a las variables del contexto en el que se incluyen.

**Posibilidades**:

*   Añadir **variables adicionales** con la palabra _with._

```
# template.html tendrá acceso a las variables del contexto actual y a las adicionales #}
{% include 'template.html' with {'foo': 'bar'} %}

{% set vars = {'foo': 'bar'} %}
{% include 'template.html' with vars %}
```

*   Impedir el acceso a las variables del contexto añadiendo _only._

```
{# sólo la variable foo será accesible #}
{% include 'template.html' with {'foo': 'bar'} only %}
```

```
{# no habrá variables accesibles #}
{% include 'template.html' only %}
```

*   El nombre de la template puede ser cualquier expresión válida de **Twig**.

```
{% include some_var %}
{% include ajax ? 'ajax.html' : 'not_ajax.html' %}
```

*   Ignorar la sentencia si la template no existe con la palabra _ignore missing_.

```
{% include 'sidebar.html' ignore missing %}
{% include 'sidebar.html' ignore missing with {'foo': 'bar'} %}
{% include 'sidebar.html' ignore missing only %}
```

*   Proporcionar una lista de templates, la primera que exista se incluye.

```
{% include ['page_detailed.html', 'page.html'] %}
```

### 13\. macro

Las macros son comparables a las funciones en los lenguajes de programación. Son útiles para poner fragmentos de HTML en elementos reusables para evitar la repetición.

```
{% macro input(name, value, type, size) %}
    <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
{% endmacro %}
```

Las macros se diferencian de las funciones de PHP en varios aspectos:

*   Los valores por defecto de argumentos se definen con el filtro _default_.
*   Los argumentos de una macro son siempre opcionales.
*   Si se pasan argumentos extra a la macro, formarán parte de una lista de valores de la variable especial _varargs_.

Como las **funciones de PHP**, las macros no tienen acceso a las variables de la template actuales. Pero puedes pasar el contexto actual como argumento empleando la variable __context_.

Las macros se pueden definir en cualquier template, y han de ser importadas antes de usarse:

```
{% import "forms.html" as forms %}
```

El archivo _forms.html_ puede contener sólo macros o una template y algunos macros, se importa el archivo y las funciones como elementos de la variable _forms_.

La macro puede entonces llamarse:

```
<p>{{ forms.input('username') }}</p>
<p>{{ forms.input('password', null, 'password') }}</p>
```

Si las macros son definidas y usadas en la misma template, puedes usar la variable especial __self_ para importarlas:

```
{% import _self as forms %}

<p>{{ forms.input('username') }}</p>
```

Cuando quieras usar una macro en otra macro desde el mismo archivo, tienes que importarla localmente:

```
{% macro input(name, value, type, size) %}
    <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
{% endmacro %}

{% macro wrapped_input(name, value, type, size) %}
    {% import _self as forms %}

    <div class="field">
        {{ forms.input(name, value, type, size) }}
    </div>
{% endmacro %}
```

### 14\. sandbox

La etiqueta _sandbox_ se puede emplear para activar el modo **sandboxing** para una template incluída, cuando sandboxing no está activado de forma global:

```
{% sandbox %}
    {% include 'user.html' %}
{% endsandbox %}
```

La etiqueta _sandbox_ sólo puede emplearse con una tag _include_, no puede usarse con ninguna sección de la template. Lo siguiente no funciona.

```
{% sandbox %}
    {% for i in 1..2 %}
        {{ i }}
    {% endfor %}
{% endsandbox %}
```

### 15\. set

Dentro de bloques de código puedes asignar **valores** a **variables**. La asignación emplea la etiqueta _set_ y puede asignar múltiples objetivos.

```
{% set foo = 'bar' %}
```

Una vez establecida la variable, estará disponible en la template como cualquier otra:

```
{# displays bar #}
{{ foo }}
```

El valor asignado puede ser cualquier expresión Twig válida:

```
{% set foo = [1, 2] %}
{% set foo = {'foo': 'bar'} %}
{% set foo = 'foo' ~ 'bar' %}
```

Varias variables pueden asignarse en un _block_:

```
{% set foo, bar = 'foo', 'bar' %}

{# es equivalente a #}

{% set foo = 'foo' %}
{% set bar = 'bar' %}
```

La etiqueta set también puede usarse para capturar bloques de texto:

```
{% set foo %}
    <div id="pagination">
        ...
    </div>
{% endset %}
```

Si tienes activado el escape automático, Twig sólo considerará que el contenido es seguro cuando se capturen bloques de texto.

Una variable declarada dentro de un loop _for_ no estará accesible fuera del loop:

```
{% for item in list %}
    {% set foo = item %}
{% endfor %}

{# foo no está disponible #}
```

Si quieres acceder a la variable, simplemente declárala antes del loop:

```
{% set foo = "" %}
{% for item in list %}
    {% set foo = item %}
{% endfor %}

{# foo está disponible #}
```

### 16\. spaceless

Emplea la etiqueta _spaceless_ para **eliminar espacios en blanco entre etiquetas HTML** (no dentro de etiquetas HTML o espacios en blaco en texto).

```
{% spaceless %}
    <div>
        <strong>foo</strong>
    </div>
{% endspaceless %}

{# devolverá <div><strong>foo</strong></div> #}
```

Esta etiqueta no pretende optimizar el tamaño del contenido HTML, simplemente evitar espacios extra. Para optimizar el tamaño del contenido HTML comprímelo en gzip.

### 17\. use

La **reutilización horizontal** es una funcionalidad que no suele emplearse mucho. Se emplea en proyectos que han de crear **bloques de templates reusables** sin emplear herencia.

La herencia normal es como sigue:

```
{% extends "base.html" %}

{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

Si ahora empleamos la reutilización horiontal, podemos tener los mismos beneficios que con herencia múlltiple, pero sin su complejidad:

```
{% extends "base.html" %}

{% use "blocks.html" %}

{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

La sentencia use le dice a Twig que importe los bloques definidos en blocks.html en la template actual (como los macros pero para blocks).

La template blocks puede ser:

```
{# blocks.html #}

{% block sidebar %}{% endblock %}
```

El código resultante es equivalente al siguiente:

```
{% extends "base.html" %}

{% block sidebar %}{% endblock %}
{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

La etiqueta _use_ importa una template si ésta no extiende a otra, si no define macros y si el body está vacío. Pero puede usar _use_ en otras templates.

Ya que las sentencias _use_ pueden resolverse independientemente del contexto pasado a la template, la referencia a la template no puede ser una expresión.

La template principal puede también sobreescribir cualquier bloque importado. Si la template ya define el **block sidebar**, entonces se ignorará el definido en _blocks.html_. Para evitar conflictos de nombres, puedes renombrar bloques importados:

```
{% extends "base.html" %}

{% use "blocks.html" with sidebar as base_sidebar, title as base_title %}

{% block sidebar %}{% endblock %}
{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

La función _parent_ determina automáticamente el correcto árbol de herencia, por lo que puede usarse cuando se sobreescribe un bloque definido en una template importada.

```
{% extends "base.html" %}

{% use "blocks.html" %}

{% block sidebar %}
    {{ parent() }}
{% endblock %}

{% block title %}{% endblock %}
{% block content %}{% endblock %}
```

En este ejemplo _parent_ llamará al bloque _sidebar_ de la template _blocks.html_.

También puedes **simular herencia** llamando al bloque madre:

```
{% extends "base.html" %}

{% use "blocks.html" with sidebar as parent_sidebar %}

{% block sidebar %}
    {{ block('parent_sidebar') }}
{% endblock %}
```

Se pueden usar tantas sentencias _use_ como se desee. Si dos templates importadas definen el mismo bloque, gana el último.

### 18\. verbatim

La etiqueta verbatim marca secciones como texto plano que no ha de ser parseado. Por ejemplo para poner sintaxis Twig como ejemplo en una template:

```
{% verbatim %}
    <ul>
    {% for item in seq %}
        <li>{{ item }}</li>
    {% endfor %}
    </ul>
{% endverbatim %}
```