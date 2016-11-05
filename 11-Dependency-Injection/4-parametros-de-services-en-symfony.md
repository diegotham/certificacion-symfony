Puedes **definir parámetros en el service container** que pueden usarse directamente o como parte de las definiciones de services, lo que puede ser útil para separar valores que crees cambiar más a menudo.

### Obtener y establecer parámetros

Trabajar con parámetros del container es sencillo con los métodos del contenedor para parámetros.

Para **comprobar si se ha definido un parámetro**:

```
$container->hasParameter('mailer.transport');
```

Para **obtener un parámetro**:

```
$container->getParameter('mailer.transport');
```

Para **establecer un parámetro**:

```
$container->setParameter('mailer.transport', 'sendmail');
```

Sólo se puede definir un parámetro antes de que se compile el container.

### Parámetros en archivos de configuración

También se pueden establecer parámetros desde un archivo de configuración: 

```
parameters:
    mailer.transport: sendmail
```

Puedes hacer referencia a estos parámetros en otras partes con el símbolo de porcentaje %, _%mailer.transport%_. Un uso de esto es inyectar los valores en tus services. Esto te permite configurar **diferentes versiones** de _services_ entre aplicaciones o _multiple services_ basados en la misma clase pero configurados de forma diferente dentro de una aplicación. 

Podrías por ejemplo inyectar el mail transport directamente en la clase **Mailer**, pero declararlo como parámetro lo hace más fácil de cambiar:

```
parameters:
    mailer.transport: sendmail

services:
    mailer:
        class:     Mailer
        arguments: ['%mailer.transport%']
```

Los valores entre las etiquetas parámeter en **archivos de configuración XML** no se ajustan con _trim_.

Esto significa que una configuración como esta:

```
<parameter key="mailer.transport">
    sendmail
</parameter>
```

Se traduce en \n sendmail\n. Para evitar esto conviene establecer los parámetros ajustados:

```
<parameter key="mailer.transport">sendmail</parameter>
```

### Parámetros array

Los parámetros no tienen por qué ser sólo _strings_, pueden también contener _arrays_:

```
parameters:
    my_mailer.gateways:
        - mail1
        - mail2
        - mail3
    my_multilang.language_fallback:
        en:
            - en
            - fr
        fr:
            - fr
            - en
```

### Constantes como parámetros

El container también soporta establecer **constantes PHP como parámetros**. Esta funcionalidad sólo está disponible en **configuración XML o PHP**. 

```
<?xml version="1.0" encoding="UTF-8" ?>
<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

    <parameters>
        <parameter key="global.constant.value" type="constant">GLOBAL_CONSTANT</parameter>
        <parameter key="my_class.constant.value" type="constant">My_Class::CONSTANT_NAME</parameter>
    </parameters>
</container>
```

Si empleas normalmente YAML y quieres esta funcionalidad, puedes importar una **configuración XML** así:

```
imports:
    - { resource: parameters.xml }
```

### Palabras PHP en XML

Por defecto, true, false y null en **XML** se convierten a las **palabras en PHP** true, false y null.

```
<parameters>
    <parameter key="mailer.send_all_in_once">false</parameter>
</parameters>

<!-- after parsing
$container->getParameter('mailer.send_all_in_once'); // devuelve false
-->
```

Para desactivar este comportamiento, emplea el type string:

```
<parameters>
    <parameter key="mailer.some_parameter" type="string">true</parameter>
</parameters>

<!-- after parsing
$container->getParameter('mailer.some_parameter'); // devuelve "true"
-->
```

Esto no es un problema en YAML o PHP, ya que tienen soporte incorporado para palabras PHP.