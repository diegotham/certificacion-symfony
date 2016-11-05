El deployment puede ser una tarea compleja de instalar dependiendo de la instalación, el servidor y los requisitos de la aplicación. En esa sección se ven los requisitos e ideas más comunes a la hora de **lanzar una aplicación Symfony**.

### Lo básico del deployment

Los pasos típicos a la hora de lanzar una aplicación Symfony son:

1.  Subir el código al servidor de producción
2.  Instalar las dependencias en el vendor (normalmente a través de composer)
3.  Realizar migraciones de base de datos o tareas similares para actualizar cualquier estructura de datos
4.  Limpiar (y opcionalmente reavivar) la caché

El deployment puede incluir también otras tareas, como:

*   Etiquetar una versión particular de tu código como un lanzamiento en tu repositorio de control de versiones
*   Crear una **staging area** temporal para construir tu instalación actualizada "offline"
*   Ejecutar tests disponibles para asegurar el código o la estabilidad del servidor
*   Remover archivos innecesarios del directorio _/web_ para mantener tu entorno de producción limpio
*   Limpiar sistemas de caché externos (como [Memcached](http://memcached.org/) o [Redis](http://redis.io/))

### Cómo lanzar una aplicación Symfony

Hay diferentes formas de lanzar una aplicación Symfony. Comienza con unas pocas **estrategias básicas de deployment** y parte de ahí.

#### Transferencia de archivos básica

La forma más básica de lanzar la aplicación es subir los archivos manualmente a través de **ftp/scp** o métodos similares. Esto tiene desventajas como que pierdes el control en el sistema a medida que progresa la actualización. Este método también requiere que tomes algunos pasos manuales después, que lo vemos un poco más adelante.

#### Utilizar control de versiones

Si utilizas un **control de versiones** (como **Git** o **SVN**), puedes simplificar el proceso teniendo tu instalación como copia de tu repositorio. Cuando quieras actualizar es tan simple como enviar las últimas actualizaciones a través del control de versiones.

Este proceso hace que sea más fácil actualizar tus archivos, pero todavía necesitas tener en cuenta otros pasos después, que se ve más adelante.

#### Utilizar build scripts y otras herramientas

Existen también herramientas para ayudar a facilitar el arduo proceso de deployment. Algunas de ellas se han adecuado específicamente para Symfony: [Capistrano](http://capistranorb.com/) con el Symfony plugin, [sf2debpkg](https://github.com/liip/sf2debpkg), [Magallanes](https://github.com/andres-montanez/Magallanes), [Fabric](http://www.fabfile.org/), [Deployer](http://deployer.org/), [Bundles](http://knpbundles.com/search?q=deploy), o desde la consola como con [Ant](http://blog.sznapka.pl/deploying-symfony2-applications-with-ant/).

Desde la documentación de Symfony también se informa detalladamente de como hacer deploy con los proveedores **PaaS (Platform as a Service)** más comunes: [Microsoft Azure](http://symfony.com/doc/current/cookbook/deployment/azure-website.html), [Heroku](http://symfony.com/doc/current/cookbook/deployment/heroku.html) y [Platform.sh](http://symfony.com/doc/current/cookbook/deployment/platformsh.html).

### Tareas Post-deployment comunes

Después de subir el código, hay varias cosas comunes que se han de realizar:

*   **Comprobar** requisitos. Comprobar si tu servidor satisface los requisitos, ejecutando:

```
php bin/symfony_requirements
```

*   **Configurar** el archivo _app/config/parameters.yml_

Este archivo no ha de subirse, sólo manejarse con las utilidades proporcionadas con Symfony

*   **Instalar/Actualizar** los Vendors

Los vendors pueden actualizarse antes de transferir el código fuente o después. De cualquiera de las formas, se hace con el siguiente comando:

```
composer install --no-dev --optimize-autoloader
```

La bandera _--optimize-autoloader_ **mejora el rendimiento del autoloader de Composer** de forma significativa construyendo un "mapa de clases". La bandera _--no-dev_ asegura que paquetes en desarollo no se instalen en el entorno de producción.

Si obtienes un error "class not found" durante este paso, podrías necesitar establecer _export SYMFONY_ENV=prod_ antes de ejecutar el comando, de forma que los scripts post-install-cmd se ejecuten en el entorno de producción. 

*   **Limpiar** la Cache de Symfony

Asegura que limpias y reavivas la cache:

```
php bin/console cache:clear --env=prod --no-debug
```

*   **Volcar** los assets de Assetic

Si utilizas Assetic, también querrás volcar los assets:

```
php bin/console assetic:dump --env=prod --no-debug
```

*   **Otras** cosas

Ejecutar migraciones de bases de datos, limpiar la cache APC, ejecutar assets:install (aunque esto ya se hace en composer install), añadir/editar CRON jobs, utilizar un CDN para los assets...