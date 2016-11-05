La idea detrás de Symfony: **no te cierres a Symfony, crea aplicaciones que se adapten a tus necesidades**.

Symfony respeta los estándares de PHP: **PHPUnit**, convenciones de nombres para las clases, etc. Además, Symfony también permite usar ciertas piezas de su Software por partes (inyector de dependencias, administrador de traducciones, administración de formularios, etc) sin la necesidad de utilizar el framework en su totalidad.

Symfony es tan interoperable que en su núcleo ya utiliza herramientas externas: **ORM Doctrine**, **Swiftmailer**, etc. Symfony y su comunidad han desarrollado innovaciones en el mundo de PHP que se han obtenido de otros lenguajes, como la idea de inyección de dependencias de Java. También, en búsqueda de la mejora de la productividad de los desarrolladores, crearon la Web Debug Toolbar, ideas también de otros frameworks.

Cientos de sitios web y aplicaciones utilizan Symfony de alguna forma: Yahoo!, Dailymotion, Opensky.com, Drupal, phpBB, etc. Desde el comienzo, Symfony se ha centrado en la rapidez, con un énfasis especial en el rendimiento de las aplicaciones. Gracias a su **inyector de dependencias** y al **EventDispatcher**, Symfony es totalmente configurable para crear aplicaciones con su versión completa, con algunos de sus componentes o mediante un microframework.

**Indice de contenido**

1.  ¿Qué es PSR?
2.  PSR-1, Estándares básicos de código
3.  PSR-2, Guía de estilo de código
4.  PSR-4, Autoloader
5.  PSR-7, Mensajes de interface HTTP

### ¿Qué es PSR?

PSR significa **PHP Standards Recommendation**. PHP nunca había tenido estándares para escribir código. Desde la conferencia de [php[tek] de 2009](http://tek.phparch.com/) comenzaron a discutirse las opciones para la interoperatibilidad entre proyectos. La conclusión fue establecer una serie de estándares.

Actualmente estos estándares los marca el FIG, [Framework Interoperability Group](http://www.php-fig.org/). Aunque el nombre explícitamente hace referencia a frameworks, desarrolladores que representan todo tipo de proyectos han sido aceptados como [miembros de voto](http://www.php-fig.org/members/). El objetivo del FIG es crear un diálogo entre representantes de proyectos, con el objetivo de encontrar formas de trabajar juntos (para la interoperatibilidad).

**Symfony** sigue los estándares definidos en los documentos [PSR-0](http://www.php-fig.org/psr/psr-0/) (**deprecated**, la alternativa es PSR-4), [PSR-1](http://www.php-fig.org/psr/psr-1/), [PSR-2](http://www.php-fig.org/psr/psr-2/) y [PSR-4](http://www.php-fig.org/psr/psr-4/).

### PSR-1, Estándares básicos de código

PSR-1 se centra en un estándar básico de código. No detalla demasiado, simplemente marca unas normas para asegurar una mínima interoperabilidad entre código PHP:

*   Sólo se deben usar las etiquetas <?php y <?=.
*   Sólo se debe usar UTF-8 sin BOM para código PHP.
*   Separar acciones de efectos secundarios (generar output, acceder a una base de datos, modificación de configuraciones iniciales, uso explícito de require o include, enviar errores o excepciones, modificar variables globales o estáticas, leer o escribir un archivo, etc) de las declaraciones estructurales (clases, funciones, constantes, etc).
*   Los espacios de nombres y las clases deben cumplir el estándar PSR-0.
*   Las constantes de las clases deben declararse en mayúsculas con guiones bajos como separadores CONSTANTE_DE_CLASE.
*   Los nombres de los métodos deben declararse en notación _camelCase_.

### PSR-2, Guía de estilo de código

Esta guía expande a PSR-1, y su objetivo es reducir la dificultad a la hora de interpretar código de diferentes autores. 

*   El código debe seguir el estándar PSR-1.
*   El código debe usar 4 espacios como indentación, no tabuladores.
*   No debe haber un límite estricto en la longitud de las líneas. El límite flexible es de 120 caracteres, aunque las líneas deberían ser de 80 caracteres o menos.
*   Debe haber una línea en blanco tras la declaración del _namespace_, y otra línea en blanco después de las declaraciones _use_.
*   La apertura de llaves en clases debe ir en la siguiente línea, y el cierre debe ir en la línea siguiente después del _body_.
*   La apertura de llaves en métodos debe ir en la siguiente línea, y el cierre debe ir en la línea siguiente después del _body_.
*   La visibilidad se debe declarar en todas las propiedades y métodos. _abstract_ y _final_ deben declararse antes que la visibilidad. _static_ debe declararse después de la visibilidad.
*   Las palabras de estructuras de control deben tener un espacio después, las llamadas a métodos y funciones no deben.
*   La apertura de llaves en estructuras de control deben ir en la misma línea, y el cierre de llaves debe ir en la siguiente línea después del _body_.
*   La apertura de paréntesis para estructuras de control no deben tener un espacio después, y el cierre de paréntesis para estructuras de control no deben tener un espacio antes.

### PSR-4, Autoloader

Este PSR describe una especificación para **clases autoloading** desde directorios de archivos. Puede usarse además de cualquier otra especificación de autoloading, como PSR-0\. Este PSR también describe donde poner los archivos que serán autocargados de acuerdo a la especificación.

*   El término class se refiere a clases, interfaces, traits y otras estructuras similares.
*   Un nombre _fully qualified class_ tiene la siguiente forma:

    \<Namespace>(\<SubnamespaceNames>)*\<ClassName>

    - El nombre _fully qualified class_ debe tener un nombre de **namespace superior**, también conocido como "_vendor namespace_".
    - El nombre _fully qualified class_ debe tener uno o más nombres subnamespace.
    - El nombre _fully qualified class_ debe tener una clase al final.
    - Las barras bajas no tienen un significado especial en ninguna porción del nombre _fully qualified class_.
    - Caracteres alfabéticos en el nombre _fully qualified class_ pueden ser cualquier combinación de mayúsculas y minúsculas.
    - Todos los nombres de clases deben referenciarse teniendo en cuenta mayúsculas y minúsculas.
*   Cuando se carga un archivo que corresponde a un nombre _fully qualified class_:

    - Series de uno o más nombres de namespaces y subnamespaces, sin incluir el **namespace superior** en el nombre _fully qualified class_ ("_namespace prefix_"), corresponde al menos a un **directorio base**.
    - Los nombres de subnamespaces después del "_namespace prefix_" corresponde a un subdirectorio dentro del **directorio base**, en el cual los separadores del namespace representan separadores de directorios. El nombre del subdirectorio debe coincidir con el de los nombres de los subnamespaces.
    - El nombre de la clase al final corresponde al nombre de archivo que termina en _.php_. El nombre del archivo debe coincidir con el nombre de la clase al final.
*   La implementación de autoloaders no deben lanzar excepciones, no deben lanzar errores de ningún nivel, y no deberían devolver ningún valor.

### PSR-7, mensajes de interface HTTP

Este PSR describe **interfaces** comunes para representar **mensajes HTTP** tal y como se describen en [RFC 7230](http://tools.ietf.org/html/rfc7230) y [RFC 7231](http://tools.ietf.org/html/rfc7231), y URIs para usar con mensajes HTTP descritos en [RFC 3986](http://tools.ietf.org/html/rfc3986). Los mensajes HTTP son la base del desarrollo web. Los **navegadores** y **clientes HTTP como cURL** crean mensajes request que se envían a un servidor web, y éste proporciona un mensaje response. El código del servidor recibe el mensaje HTTP request y devuelve un mensaje HTTP response.

Este nuevo PSR tiene un gran potencial para la interoperatibilidad y estandarización en PHP. Esto es especialmente importante en **middleware**: funciones que manipulan el proceso request-response. En el futuro, un middleware escrito con estas nuevas interfaces podría usarse en cualquier framework.

Hoy en día un gran número de proyectos utilizan las clases Request y Response de Symfony (a través del componente [HttpFoundation](http://symfony.com/components/HttpFoundation)), incluyendo **Laravel** y **Drupal 8**. Esto ha provocado una base para la estandarización de mensajes HTTP en los últimos años, antes de que la comunidad estuviera de acuerdo en crear una interface oficial.

El [PSR HTTP Message bridge](https://github.com/symfony/psr-http-message-bridge) es una librería que puede convertir objetos Request y Response de Symfony en objetos compatibles con PSR-7 y viceversa. Puedes leer como hacer las transiciones [aquí](http://symfony.com/blog/psr-7-support-in-symfony-is-here).