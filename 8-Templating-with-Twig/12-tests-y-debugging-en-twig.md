Los _tests_ en Twig permiten comprobar **variables** frente a **expresiones**, como por ejemplo para comprobar su existencia o que equivale a valor determinado.

### constant

_constant_ comprueba si una variable tiene el mismo valor que una constante. Puedes comprobar **constantes globales o de clases**:

```
{% if post.status is constant('Post::PUBLISHED') %}
    the status attribute is exactly the same as Post::PUBLISHED
{% endif %}
```

O también de **instancias de objetos**:

```
{% if post.status is constant('PUBLISHED', post) %}
    the status attribute is exactly the same as Post::PUBLISHED
{% endif %}
```

### defined

_defined_ comprueba si una variable está definida en el contexto actual. Es muy útil si empleas la opción _strict_variables_:

```
{# defined works with variable names #}
{% if foo is defined %}
    ...
{% endif %}

{# and attributes on variables names #}
{% if foo.bar is defined %}
    ...
{% endif %}

{% if foo['bar'] is defined %}
    ...
{% endif %}
```

Cuando se emplea el test _defined_ en una expresión que utiliza variables en algunas llamadas a métodos, asegúrate primero de que están definidas esas variables primero:

```
{% if var is defined and foo.method(var) is defined %}
    ...
{% endif %}
```

### divisible by

_divisible by_ comprueba si una variable es divisible por un número:

```
{% if loop.index is divisible by(3) %}
    ...
{% endif %}
```

### empty

_empty_ comprueba si una variable está vacía:

```
{# devuelve true si la variable foo es null, false, una array vacío, o un string vacío #}
{% if foo is empty %}
    ...
{% endif %}
```

### even

_even_ devuelve **true** si el número dado es impar:

```
{{ var is even }}
```

### iterable

_iterable_ comprueba si una variable es un array o un objeto _traversable_:

```
{# devuelve true si la variable foo es iterable #}
{% if users is iterable %}
    {% for user in users %}
        Hello {{ user }}!
    {% endfor %}
{% else %}
    {# users es probablemente un string #}
    Hello {{ users }}!
{% endif %}
```

### null

_null_ devuelve true si la variable es null. _none_ es un alias de null.

```
{{ var is null }}
```

### odd

_odd_ devuelve true si el número dado es par:

```
{{ var is odd }}
```

### same as

_same as_ comprueba si una variable es la misma que otra variable. Es el equivalente a === en PHP:

```
{% if foo.attribute is same as(false) %}
    El atributo foo equivale al 'false' de PHP
{% endif %}
```

### Debugging

En PHP podemos emplear la [función dump del componente VarDumper](http://symfony.com/doc/current/components/var_dumper/introduction.html#components-var-dumper-dump) si necesitas comprobar el valor de una variable. Esto puede emplearse por ejemplo en un controller:

```
// src/AppBundle/Controller/ArticleController.php
namespace AppBundle\Controller;

// ...

class ArticleController extends Controller
{
    public function recentListAction()
    {
        $articles = ...;
        dump($articles);

        // ...
    }
}
```

La salida de la función _dump_ es entonces renderizada en la barra web del desarrollador.

El mismo mecanismo puede emplearse en las templates Twig con la función _dump_:

```
{# app/Resources/views/article/recent_list.html.twig #}
{{ dump(articles) }}

{% for article in articles %}
    <a href="/article/{{ article.slug }}">
        {{ article.title }}
    </a>
{% endfor %}
```

Las variables sólo se mostrarán si el ajuste de Twig _debug_ (en _config.yml_) es **true**. Por defecto esto significa que las variables se volcarán en el entorno _dev_ pero no en el _prod_.

### Comprobación de sintaxis

Puedes comprobar errores de sintaxis en **templates Twig** con el comando de consola _lint:twig_:

```
# Puedes comprobar por nombre de archivo:
$ php bin/console lint:twig app/Resources/views/article/recent_list.html.twig

# o por directorio:
$ php bin/console lint:twig app/Resources/views
```