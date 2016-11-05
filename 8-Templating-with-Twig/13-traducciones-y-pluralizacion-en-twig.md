La mayoría de las veces las traducciones ocurren en templates. **Symfony** proporciona soporte nativo para **templates PHP y Twig**.

Symfony proporciona etiquetas especiales **Twig** (_trans_ y _transchoice_) para ayudar a traducir bloques estáticos de texto:

```
{% trans %}Hello %name%{% endtrans %}

{% transchoice count %}
    {0} There are no apples|{1} There is one apple|]1,Inf[ There are %count% apples
{% endtranschoice %}
```

La etiqueta _transchoice_ obtiene automáticamente la variable **%count%** del contexto acutal y lo pasa al translator. Este mecanismo sólo funciona cuando usas un placeholder siguiendo el patrón %var%.

La notación **%var%** de placeholders es requerida cuando se traduce en **templates Twig** con la etiqueta. Si necesitas usar el carácter % en un string, puedes escaparlo poniéndolo dos veces: {% trans %}Percent: %percent%%%{% endtrans %}.

Puedes también especificar el _dominio_ del mensaje y pasar algunas variables adicionales:

```
{% trans with {'%name%': 'Fabien'} from "app" %}Hello %name%{% endtrans %}

{% trans with {'%name%': 'Fabien'} from "app" into "fr" %}Hello %name%{% endtrans %}

{% transchoice count with {'%name%': 'Fabien'} from "app" %}
    {0} %name%, there are no apples|{1} %name%, there is one apple|]1,Inf[ %name%, there are %count% apples
{% endtranschoice %}
```

Los filtros _trans_ y _transchoice_ pueden usarse para **traducir textos variables** y **expresiones complejas**:

```
{{ message|trans }}

{{ message|transchoice(5) }}

{{ message|trans({'%name%': 'Fabien'}, "app") }}

{{ message|transchoice(5, {'%name%': 'Fabien'}, 'app') }}
```

Utilizar las etiquetas o filtros de traducción tiene el mismo efecto, pero con una diferencia: el escapado automático sólo se aplica a las **traducciones mediante un filtro**. Si necesitas estar seguro de que tu mensaje traducido no se escapa, debes aplicar el filtro _raw_ después del **filtro de traducción**:

```
{# text translated between tags is never escaped #}
{% trans %}
    <h3>foo</h3>
{% endtrans %}

{% set message = '<h3>foo</h3>' %}

{# strings and variables translated via a filter are escaped by default #}
{{ message|trans|raw }}
{{ '<h3>bar</h3>'|trans|raw }}
```

Puedes establecer el **dominio de traducción** de una **template Twig** entera con una etiqueta:

```
{% trans_default_domain "app" %}
```

Esto sólo influye en la template actual, no templates incluídas con _include_.