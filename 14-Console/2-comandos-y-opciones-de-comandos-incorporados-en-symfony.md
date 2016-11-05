Además de las opciones que puedas especificar en los comandos, existen **opciones y comandos incorporados** para el **componente Console**. En los siguientes ejemplos suponemos que hemos creado el archivo _application.php_:

```
#!/usr/bin/env php
<?php
// application.php

use Symfony\Component\Console\Application;

$application = new Application();
// ...
$application->run();
```

### Comandos incorporados

El comando _list_ muestra las **opciones estándar y los comandos registrados**:

```
php application.php list
```

Se puede obtener el mismo resultado sin incluir ningún comando también:

```
php application.php
```

Por ejemplo la siguiente es una lista de comandos en una aplicación con algunos bundles como FOSUser, FOSElastica, Doctrine, LiipImagine, OneUploader: 

  _help_                                    Displays help for a command

  _list_                                    Lists commands

 **assets**

  _assets:install _                         Installs bundles web assets under a public web directory

 **cache**

  _cache:clear_                             Clears the cache

  _cache:warmup_                            Warms up an empty cache

 **config**

  _config:dump-reference_                   Dumps the default configuration for an extension

 **debug**

  _debug:config_                            Dumps the current configuration for an extension

  _debug:container_                         Displays current services for an application

  _debug:event-dispatcher _                 Displays configured listeners for an application

  _debug:router_                            Displays current routes for an application

  _debug:swiftmailer_                       Displays current mailers for an application

  _debug:translation_                       Displays translation messages information

  _debug:twig_                              Shows a list of twig functions, filters, globals and tests

 **doctrine**

  _doctrine:cache:clear-collection-region_  Clear a second-level cache collection region.

  _doctrine:cache:clear-entity-region _     Clear a second-level cache entity region.

  _doctrine:cache:clear-metadata_           Clears all metadata cache for an entity manager

  _doctrine:cache:clear-query       _       Clears all query cache for an entity manager

  _doctrine:cache:clear-query-region_       Clear a second-level cache query region.

  _doctrine:cache:clear-result_             Clears result cache for an entity manager

  _doctrine:database:create_                Creates the configured database

  _doctrine:database:drop_                  Drops the configured database

  _doctrine:ensure-production-settings_     Verify that Doctrine is properly configured for a production environment.

  _doctrine:generate:crud_                  Generates a CRUD based on a Doctrine entity

  _doctrine:generate:entities_              Generates entity classes and method stubs from your mapping information

  _doctrine:generate:entity_                Generates a new Doctrine entity inside a bundle

  _doctrine:generate:form_                  Generates a form type class based on a Doctrine entity

  _doctrine:mapping:convert_                Convert mapping information between supported formats.

  _doctrine:mapping:import_                 Imports mapping information from an existing database

  _doctrine:mapping:info_                   

  _doctrine:migrations:diff_                Generate a migration by comparing your current database to your mapping information.

  _doctrine:migrations:execute_             Execute a single migration version up or down manually.

  _doctrine:migrations:generate_            Generate a blank migration class.

  _doctrine:migrations:latest_              Outputs the latest version number

  _doctrine:migrations:migrate_             Execute a migration to a specified version or the latest available version.

  _doctrine:migrations:status_              View the status of a set of migrations.

  _doctrine:migrations:version_             Manually add and delete migration versions from the version table.

  _doctrine:query:dql_                      Executes arbitrary DQL directly from the command line.

  _doctrine:query:sql_                      Executes arbitrary SQL directly from the command line.

  _doctrine:schema:create_                  Executes (or dumps) the SQL needed to generate the database schema

  _doctrine:schema:drop_                    Executes (or dumps) the SQL needed to drop the current database schema

  _doctrine:schema:update_                  Executes (or dumps) the SQL needed to update the database schema to match the current mapping metadata.

  _doctrine:schema:validate_                Validate the mapping files.

 **fos**

  _fos:elastica:populate_                   Populates search indexes from providers

  _fos:elastica:reset_                      Reset search indexes

  _fos:elastica:search_                     Searches documents in a given type and index

  _fos:user:activate_                       Activate a user

  _fos:user:change-password_                Change the password of a user.

  _fos:user:create_                         Create a user.

  _fos:user:deactivate_                     Deactivate a user

  _fos:user:demote_                         Demote a user by removing a role

  _fos:user:promote_                        Promotes a user by adding a role

 **gaufrette**

  _gaufrette:filesystem:keys_               List all the file keys of a filesystem

 **generate**

  _generate:bundle_                         Generates a bundle

  _generate:command_                        Generates a console command

  _generate:controller_                     Generates a controller

  _generate:doctrine:crud_                  Generates a CRUD based on a Doctrine entity

  _generate:doctrine:entities_              Generates entity classes and method stubs from your mapping information

  _generate:doctrine:entity_                Generates a new Doctrine entity inside a bundle

  _generate:doctrine:form_                  Generates a form type class based on a Doctrine entity

 **liip**

  _liip:imagine:cache:remove_               Remove cache for given paths and set of filters.

  _liip:imagine:cache:resolve_              Resolve cache for given path and set of filters.

 **lint**

  _lint:twig_                               Lints a template and outputs encountered errors

  _lint:yaml_                               Lints a file and outputs encountered errors

 **oneup**

  _oneup:uploader:clear-chunks_             Clear chunks according to the max-age you defined in your configuration.

  _oneup:uploader:clear-orphans _           Clear orphaned uploads according to the max-age you defined in your configuration.

 **orm**

  _orm:convert:mapping_                     Convert mapping information between supported formats.

 **router**

  _router:match_                            Helps debug routes by simulating a path info match

 **security**

  _security:check_                          Checks security issues in your project dependencies

  _security:encode-password _               Encodes a password.

 **server**

  _server:run_                              Runs PHP built-in web server

  _server:start_                            Starts PHP built-in web server in the background

  _server:status_                           Outputs the status of the built-in web server for the given address

  _server:stop_                             Stops PHP's built-in web server that was started with the server:start command

 **swiftmailer**

  _swiftmailer:debug_                       Displays current mailers for an application

  _swiftmailer:email:send_                  Send simple email message

  _swiftmailer:spool:send_                  Sends emails from the spool

 **translation**

  _translation:update_                      Updates the translation file

El comando _help_ muestra **información de ayuda para el comando** especificado. Si por ejemplo queremos ayuda para el comando _list_:

```
php application.php help list
```

El comando _help_ sin especificar ningún comando mostrará las **opciones globales**:

```
php application.php help
```

### Opciones globales

Puedes obtener información de cualquier comando con la opción _--help_. Por ejemplo para obtener ayuda para el comando _list_:

```
php application.php list --help
php application.php list -h
```

Puedes suprimir el output con:

```
php application.php list --quiet

php application.php list -q
```

Puedes obtener **mensajes más explicativos** (si los soporta el comando) con:

```
php application.php list --verbose

php application.php list -v
```

Para mostrar mensajes todavía más explicativos:

```
php application.php list -vv

php application.php list -vvvv
```

Si estableces los argumentos opcionales para que proporcionen un **nombre y versión a la aplicación**:

```
$application = new Application('Acme Console Application', '1.2');
```

Puedes emplear:

```
php application.php list --version

php application.php list -v
```

para obtener la siguiente información:

```
Acme Console Application version 1.2
```

Si no proporcionas ambos argumentos el output será:

```
console tool
```

Puedes también forzar el **colorido en ANSI output**:

```
php application.php list --ansi
```

o desactivarlo con el siguiente comando:

```
php application.php list --no-ansi
```

Puedes suprimir cualquier pregunta interactiva del comando que estás ejecutando con:

```
php application.php list --no-interaction
php application.php list -n
```

### Shortcuts

No tienes por qué escribir los comandos enteros. Puedes emplear _shortcuts_ **si no hay comandos que puedan colisionar**. Por ejemplo el comando de ayuda:

```
php application.php h
```

Si tienes comandos que emplean : para **comandos con namespaces**, simplemente puedes escribir el shortcut de cada parte. Si por ejemplo hemos creado el comando _demo:greet_ podemos ejecutarlo de la siguiente forma:

```
php application.php d:g Fabien
```

Si introduces un comando de consola que colisiona con otro por su inicial, no se ejecutará ningún comando y se mostrarán sugerencias posibles para elegir.