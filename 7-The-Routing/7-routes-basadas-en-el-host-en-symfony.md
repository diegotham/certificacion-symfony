Se puede enlazar el **host HTTP** del **request** con una **route** directamente a un **controller**:

```
mobile_homepage:
    path:     /
    host:     m.example.com
    defaults: { _controller: AcmeDemoBundle:Main:mobileHomepage }

homepage:
    path:     /
    defaults: { _controller: AcmeDemoBundle:Main:homepage }
```

Ambas routes coinciden con la misma dirección /, pero se empleará la primera sólo si el host es _m.example.com._

### Utilizar placeholders

La opción host utiliza la misma sintaxis que el sistema de direcciones, lo que significa que se pueden emplear **placeholders** en el **hostname**:

```
projects_homepage:
    path:     /
    host:     "{project_name}.example.com"
    defaults: { _controller: AcmeDemoBundle:Main:mobileHomepage }

homepage:
    path:     /
    defaults: { _controller: AcmeDemoBundle:Main:homepage }
```

Puedes también establecer **restricciones** y opciones _default_ para los placeholders. Por ejemplo si quieres que se acepten m.example.com y mobile.example.com en la misma route:

```
mobile_homepage:
    path:     /
    host:     "{subdomain}.example.com"
    defaults:
        _controller: AcmeDemoBundle:Main:mobileHomepage
        subdomain: m
    requirements:
        subdomain: m|mobile

homepage:
    path:     /
    defaults: { _controller: AcmeDemoBundle:Main:homepage }
```

También se pueden emplear parámetros de _services_:

```
mobile_homepage:
    path:     /
    host:     "m.{domain}"
    defaults:
        _controller: AcmeDemoBundle:Main:mobileHomepage
        domain: '%domain%'
    requirements:
        domain: '%domain%'

homepage:
    path:  /
    defaults: { _controller: AcmeDemoBundle:Main:homepage }
```

Asegúrate de incluir también una opción _default_ para el placeholder **domain**, sino tendrás que incluir el valor domain cada vez que generes una **URL** con la **route**.

### Hosts en routes importadas

También puedes establecer la opción host en routes importadas:

```
acme_hello:
    resource: '@AcmeHelloBundle/Resources/config/routing.yml'
    host:     "hello.example.com"
```

El host _hello.example.com_ se establecerá en cada route cargada desde la nueva fuente de routes.

### Testear los Controllers

Tienes que establecer el header Host HTTP en tus objetos request si quieres estas funcionalidades en tus functional tests:

```
$crawler = $client->request(
    'GET',
    '/homepage',
    array(),
    array(),
    array('HTTP_HOST' => 'm.' . $client->getContainer()->getParameter('domain'))
);
```