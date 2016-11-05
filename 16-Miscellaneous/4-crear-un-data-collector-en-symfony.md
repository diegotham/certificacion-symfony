El [Symfony Profiler](http://symfony.com/doc/current/cookbook/profiler/index.html) delega recopilaciones de datos a algunas clases especiales llamadas _data collectors_. Symfony viene con algunas de ellas, pero puedes crear una propia fácilmente.

### Crear un data collector customizado

Crear un data collector customizado es tan simple como implementar [DataCollectorInterface](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/DataCollector/DataCollectorInterface.html):

```
interface DataCollectorInterface
{
    function collect(Request $request, Response $response, \Exception $exception = null);
    function getName();
}
```

El método [getName()](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/DataCollector/DataCollectorInterface.html#method_getName) devuelve el nombre del data collector y debe ser único en la aplicación. Este valor también se usa para acceder a la información después (por ejemplo al [usar el profiler en un functional test](http://symfony.com/doc/current/cookbook/testing/profiling.html)).

El método [collect()](http://api.symfony.com/3.0/Symfony/Component/HttpKernel/DataCollector/DataCollectorInterface.html#method_collect) es responsable de guardar los datos recopilados en propiedades locales.

La mayoría del tiempo es conveniente extender **DataCollector** y rellenar la propiedad _$this->data_ (se encarga de serializar la propiedad _$this->data_). Por ejemplo si creas un nuevo data collector que recopilar el _method_ y los _content types_ aceptados del request:

```
// src/AppBundle/DataCollector/RequestCollector.php
namespace AppBundle\DataCollector;

use Symfony\Component\HttpKernel\DataCollector\DataCollector;

class RequestCollector extends DataCollector
{
    public function collect(Request $request, Response $response, \Exception $exception = null)
    {
        $this->data = array(
            'method' => $request->getMethod(),
            'acceptable_content_types' => $request->getAcceptableContentTypes(),
        );
    }

    public function getMethod()
    {
        return $this->data['method'];
    }

    public function getAcceptableContentTypes()
    {
        return $this->data['acceptable_content_types'];
    }

    public function getName()
    {
        return 'app.request_collector';
    }
}
```

Los _getters_ se añaden para dar a la template acceso a la información recopilada.

Ya que el profiler serializa instancias de data collector, no deberías guardar objetos que no pueden ser serializados (como objetos PDO) o tiene que proporcionar tu propio método _serialize()_. 

### Activar data collectors cutomizados

Para activar un data collector, defínelo como un service regular y etiquétalo como _data_collector_:

```
# app/config/services.yml
services:
    app.request_collector:
        class: AppBundle\DataCollector\RequestCollector
        public: false
        tags:
            - { name: data_collector }
```

### Añadir templates web profiler

La información recopilada por tu data collector puede mostrarse en la _web debug toolbar_ y en el _web profiler_. Para hacerlo tienes que crear una template Twig que incluye algunos bloques específicos. 

En el caso más simple, simplemente quieres mostrar la información en la toolbar sin proporcionar un panel profiler. Esto requiere definir el bloque toolbar y establecer el valor de dos variables, _icon_ y _text_:

```
{% extends 'WebProfilerBundle:Profiler:layout.html.twig' %}

{% block toolbar %}
    {% set icon %}
        {# este es el contenido mostrado como un panel en la toolbar #}
        <span class="icon"><img src="..." alt=""/>
        <span class="sf-toolbar-status">Request
    {% endset %}

    {% set text %}
        {# este es el contenido mostrado cuando se coloca el ratón encima
           del panel toolbar #}
        <div class="sf-toolbar-info-piece">
            <b>Method</b>
            <span>{{ collector.method }}
        </div>

        <div class="sf-toolbar-info-piece">
            <b>Accepted content type</b>
            <span>{{ collector.acceptableContentTypes|join(', ') }}
        </div>
    {% endset %}

    {# el valor 'link' establecido a 'false' significa que este panel no
       muestra una sección en el profiler #}
    {{ include('@WebProfiler/Profiler/toolbar_item.html.twig', { link: false }) }}
{% endblock %}
```

Las templates collector incorporadas definen todas sus imágenes como imágenes base64-encoded incrustadas. Esto hace que funcionen en cualquier lado sin tener que tratar con los enlaces a assets:

```
<img src="data:image/png;base64,..." />
```

Otra solución es definir las imágenes como **archivos SVG**. Además de ser independientes de la resolución, estas imágenes pueden ser fácilmente incrustadas en la template Twig o incluídas desde un archivo exterior para reusarlas en varias templates:

```
{{ include('@App/data_collector/icon.svg') }}
```

Si el panel toolbar incluye información del web profiler extendida, la template Twig debe también definir bloques adicionales:

```
{% extends '@WebProfiler/Profiler/layout.html.twig' %}

{% block toolbar %}
    {% set icon %}
        <span class="icon"><img src="..." alt=""/>
        <span class="sf-toolbar-status">Request
    {% endset %}

    {% set text %}
        <div class="sf-toolbar-info-piece">
            {# ... #}
        </div>
    {% endset %}

    {{ include('@WebProfiler/Profiler/toolbar_item.html.twig', { 'link': true }) }}
{% endblock %}

{% block head %}
    {# Opcional. Aquí puedes enlazar o definir contenidos CSS y JS. #}
    {# Usa {{ parent() }} para extender los estilos por defecto en lugar de sobreescribirlos. #}
{% endblock %}

{% block menu %}
    {# Este menú izquierdo aparece cuando se usa el profiler a pantalla completa. #}
    <span class="label">
        <span class="icon"><img src="..." alt=""/>
        <strong>Request</strong>
    
{% endblock %}

{% block panel %}
    {# Opcional, para mostrar la mayoría de los detalles. #}
    <h2>Acceptable Content Types</h2>
    <table>
        <tr>
            <th>Content Type</th>
        </tr>

        {% for type in collector.acceptableContentTypes %}
        <tr>
            <td>{{ type }}</td>
        </tr>
        {% endfor %}
    </table>
{% endblock %}
```

Los bloques menu y panel son los únicos paneles requeridos para definir los contenidos mostrados en el panel web profiler asociado con este data collector. Todos los bloques tienen acceso al objeto collector.

Finalmente, para activar la template data collector, añade un atributo template a la etiqueta _data_collector_ en la configuración de tu service:

```
# app/config/services.yml
services:
    app.request_collector:
        class: AppBundle\DataCollector\RequestCollector
        tags:
            -
                name:     data_collector
                template: 'data_collector/template.html.twig'
                id:       'app.request_collector'
        public: false
```

El atributo _id_ debe coincidir con el valor devuelto por el método _getName()_.

La **posición de cada panel en la toolbar** es determinada por la prioridad definida por cada collector. La mayoría de collectors incorporados utilizan _255_ como su prioridad. Si quieres que tu collector se muestre antes, emplea un valor superior:

```
# app/config/services.yml
services:
    app.request_collector:
        class: AppBundle\DataCollector\RequestCollector
        tags:
            - { name: data_collector, template: '...', id: '...', priority: 300 }
```