Symfony administra sus lanzamientos en un modelo basado en el tiempo, y sigue una [estrategia de versiones semántica](http://semver.org/):

*   Una versión minor de Symfony (por ejemplo: 2.8, 3.2, etc) se lanza cada seis meses: una en Mayo y otra en Noviembre.
*   Una versión mayor (por ejemplo 3.0, 4.0, etc) se lanza cada dos años y es lanzada a la vez que la última versión minor de la versión mayor anterior.

### Desarrollo

El periodo de desarrollo completo para cualquier versión mayor o minor dura seis meses y se divide en dos fases:

*   **Desarrollo**: cuatro meses para añadir nuevas funcionalidades y potenciar las ya existentes.
*   **Estabilización**: dos meses para arreglar bugs, preparar el lanzamiento, y esperar a que todo el ecosistema Symfony (librerías de terceros, bundles, y proyectos que usan Symfony) se pongan al día.

Durante la etapa de desarrollo, cualquier nueva funcionalidad se puede revertir si no va a estar lista a tiempo o si no va a ser suficientemente estable para incluirse en el lanzamiento final.

### Mantenimiento

Cada versión de Symfony se mantiene durante un tiempo limitado, dependiendo del tipo de lanzamiento:

*   **Arreglos de bugs y seguridad**: Durante este periodo, todos los problemas se pueden solucionar. El final de este periodo es referenciado como el final del mantenimiento (_end of maintenance_) de una versión.
*   **Arreglos de seguridad sólo**: Durante este periodo, sólo se solucionarán problemas de seguridad. El final de este periodo es referenciado como el final de la existencia (_end of life_) de una versión.

### Versiones Symfony

#### Versiones Standard

Una versión standard minor se mantiene durante 8 meses para solucionar bugs, y durante 14 meses para temas de seguridad.

En Symfony 2.x, la versión terminó con 9 versiones minor (de la 2.0 a la 2.8). Desde la 3.0, el número de versiones minor está limitado a 5 (de X.0 a X.4).

#### Versiones Long Term Support

Cada dos años, una nueva versión **Long Term Support** (normalmente apreviada como "LTS") es publicada. Cada versión **LTS** es respaldada durante tres años para soluciones de bugs, y durante cuatro años para problemas de seguridad.

Desde [SensioLabs](http://sensiolabs.com/) también se puede comprar soporte de mayor duración para versiones anteriores.

En Symfony 2.x las versiones LTS son 2.3, 2.7 y 2.8\. Comenzando desde Symfony 3.x, sólo la última versión minor será LTS (3.4, 4.4, 5.4, etc).

#### Calendario

Calendario de las versiones Symfony:

![Calendario versiones Symfony](http://symfony.com/doc/current/_images/release-process.jpg)

En amarillo se representa la **fase de desarrollo**, en azul la **fase de estabilización** y en verde el **periodo de mantenimiento**.

La tabla de fechas y periodos de mantenimiento se puede ver en [este enlace](http://symfony.com/doc/current/contributing/community/releases.html). También pueden utilizar la herramienta que proporcionan para saber exactamente las fechas para diferentes versiones, [aquí](https://symfony.com/roadmap), incluso puedes obtener los datos como JSON con URLs como <cite>https://symfony.com/roadmap.json?version=2.x</cite>.

También es posible [subscribirse](https://symfony.com/roadmap) para notificaciones en cambios importantes de versiones.

#### Compatibilidad con versiones anteriores

La **compatibilidad con versiones anteriores de Symfony** es estricta, y permite a los desarrolladores actualizar de una versión minor de Symfony a la siguiente.

Cuando no sea posible mantener una compatibilidad con versiones anteriores, la característica, funcionalidad o el bug solucionado se guardarán para la siguiente versión mayor.

### Deprecations

Cuando la implementación de una funcionalidad no se puede reemplazar con una mejor sin romper la compatibilidad entre versiones, existe todavía la posibilidad de poner como "deprecated" una implementación antigua y añadir una nueva al mismo tiempo. 

### Rationale

El proceso de lanzamiento se adaptó para proporcionar más transparencia y que puedan ser predecibles los cambios, con los siguientes objetivos:

*   Hacer los periodos de lanzamiento más cortos.
*   Dar más visibilidad a los desarrolladores que utilizan el framework y los proyectos Open Source que usan Symfony.
*   Mejorar la experiencia de los contribuyentes de Symfony: todos saben cuándo será la siguiente versión.
*   Coordinar el timeline de Symfony con otros proyectos populares de PHP que funcionan bien con Symfony y con proyectos que utilizan Symfony.
*   Dar tiempo al ecosistema de Symfony para ponerse al día con las nuevas versiones (autores de bundles, escritores de documentación, traductores...).

El periodo de seis meses se eligió para que se lancen dos versiones al año. También permite tiempo para trabajar en nuevas funcionalidades y permite que las que no estén todavía preparadas se pospongan para la siguiente sin tener que esperar demasiado.

El sistema de mantenimiento dual se adoptó por los propios usuarios de Symfony. Los que quieren siempre tener la última versión, utilizan la versión standard: una versión se publica cada 6 meses, con un periodo de 2 meses para la actualización. Las empresas que quieren más estabilidad utilizan las versiones LTS: una nueva versión se publica cada 2 años y hay 1 año para actualizar.