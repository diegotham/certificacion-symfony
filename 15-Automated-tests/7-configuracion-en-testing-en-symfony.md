El **Client** utilizado por [functional tests](https://diego.com.es/functional-tests-en-symfony) crea un **Kernel** que se ejecuta en un entorno especial _test_. Ya que Symfony carga el archivo app/config/config_test.yml en el entorno test, puedes retocar cualquier ajuste de la aplicación específicamente para el testeo.

Por ejemplo, por defecto, el **Swift Mailer** está configurado para no enviar emails en el entorno _test_. Puedes verlo en las opciones de configuración de _swiftwailer_:

```
# app/config/config_test.yml

# ...
swiftmailer:
    disable_delivery: true
```

Puedes también utilizar un entorno totalmente diferente, o **sobreescribir el modo debug por defecto** (_true_) pasando cada uno como opciones al método _createClient()_:

```
$client = static::createClient(array(
    'environment' => 'my_test_env',
    'debug'       => false,
));
```

Si tu aplicación varía en función de algunos [headers HTTP](https://diego.com.es/headers-del-protocolo-http), pásalos como segundo argumento de _createClient()_:

```
$client = static::createClient(array(), array(
    'HTTP_HOST'       => 'en.example.com',
    'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
));
```

También puedes **sobreescribir headers HTTP por cada request**:

```
$client->request('GET', '/', array(), array(), array(
    'HTTP_HOST'       => 'en.example.com',
    'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
));
```

El **cliente test** está disponible como _service_ en el _container_ en el entorno _test_ (o donde esté activada la opción _framework.test_). Esto significa que puedes sobreescribir el service completamente si lo necesitas.

#### Configuración de PHPUnit

Cada aplicación tiene su propia **configuración PHPUnit**, guardada en el archivo _phpunit.xml.dist_. Puedes editar este archivo para cambiar los valores por defecto o crear un archivo _phpunit.xml_ para establecer una configuración sólo para tu **entorno local**. Recuerda guardar el phpunit.xml.dist en el repositorio de la aplicación e ignorar el archivo _phpunit.xml_.

Por defecto sólo los tests guardados en _/tests_ se ejecutan a través del comando _phpunit_, tal y como está configurado en el archivo _phpunit.xml.dist_:

```
<!-- phpunit.xml.dist -->
<phpunit>
    <!-- ... -->
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>tests</directory>
        </testsuite>
    </testsuites>
    <!-- ... -->
</phpunit>
```

Pero puedes añadir más directorios fácilmente. Por ejemplo, la siguiente configuración añade tests desde un directorio propio _lib/tests_:

```
<!-- phpunit.xml.dist -->
<phpunit>
    <!-- ... -->
    <testsuites>
        <testsuite name="Project Test Suite">
            <!-- ... --->
            <directory>../lib/tests</directory>
        </testsuite>
    </testsuites>
    <!-- ... --->
</phpunit>
```

Para incluir otros directorios en la cobertura del código, edita también la sección _filter_:

```
<!-- phpunit.xml.dist -->
<phpunit>
    <!-- ... -->
    <filter>
        <whitelist>
            <!-- ... -->
            <directory>../lib</directory>
            <exclude>
                <!-- ... -->
                <directory>../lib/tests</directory>
            </exclude>
        </whitelist>
    </filter>
    <!-- ... --->
</phpunit>
```