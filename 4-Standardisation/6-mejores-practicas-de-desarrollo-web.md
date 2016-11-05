Las **mejores prácticas de desarrollo** son una serie de reglas que se suelen respetar entre las comunidades de **desarrollo web** que ayudan a mejorar la calidad de los proyectos. La principal razón es que los proyectos duraderos suelen tener múltiples desarrolladores y además van cambiando los desarrolladores que las mantienen con el tiempo, por lo que seguir una serie de reglas ayuda a la comunidad en general.

1.  Mejores prácticas para la legibilidad de código
2.  Otras prácticas fundamentales en el desarrollo web
3.  Desarrollo web Agile

### 1\. Mejores prácticas para la legibilidad de código

* # Comentarios y documentación

Los comentarios y la documentación son fundamentales en cualquier proyecto, ya que además de servir para otros desarrolladores que tengan que continuarlo, te sirve a tí mismo para recordar las funcionalidades y retornos de las cosas. Los **IDE's** (**Integrated Development Environment**) han permitido que los comentarios en el código sean todavía más útiles. Siguiendo ciertos estándares en los comentarios los IDE's y otras herramientas pueden proporcionar información de forma directa, ahorrando mucho tiempo.

* # Indentación consistente

Se ha de emplear una indentación consistente, que no cambie durante el proyecto. Generalmente en la actualidad se suele seguir el modelo marcado por [PSR-2](http://diego.com.es/interoperabilidad-y-psrs-en-symfony#PSR2GuiaDeEstiloDeCodigo), pero no hay ninguna obligación en hacerlo. Un ejemplo resumen de la forma de indentar según el PSR-2:

```
namespace Proveedor\Paquete;

use FooInterfaz;
use BarClase as Bar;
use OtroProveedor\OtroPaquete\BazClase;

class Foo extends Bar implements FooInterfaz
{
    public function funcionDeEjemplo($a, $b = null)
    {
        if ($a === $b) {
            bar();
        } elseif ($a > $b) {
            $foo->bar($arg1);
        } else {
            BazClase::bar($arg2, $arg3);
        }
    }

    final public static function bar()
    {
        // cuerpo del método
    }
}
```

* # Evitar comentarios obvios

Comentar el código es fundamental, pero hacerlo demasiado tampoco es bueno. Hay que llegar a un nivel medio en el que no se comente cada línea del código sino cada vez que sea realmente necesario para entender el proceso.

* # Agrupar el código

Muchas tareas requieren más de una línea de código. Es bueno mantener estas tareas en bloques separados de código, separados por una línea. Incluir un comentario al comienzo de cada bloque también enfatiza esta separación.

* # Esquema de nombrado consistente

**PHP** no es el ejemplo ideal del nombrado consistende de funciones (por ejemplo, _strpos()_, _str_split()_). Una normal fundamental es que los nombres deben tener separación entre las palabras, ya sea con un nombrado _camelCase_ o con barras bajas _get_all_, y que mantengan el formato en el proyecto.

Si un proyecto existente sigue una convención, hay que adaptarse a ella.

En **Symfony** se emplea **camelCase** para variables, funciones, métodos y argumentos, y **barras bajas** para opciones y parámetros.

* # Principio DRY

**DRY** significa Don't Repeat Yourself, también llamado DIE, Duplication Is Evil. El principio dice que "_cada funcionalidad debe tener una representación singular, sin ambigüedades y autoritaria_".

El objetivo de la mayoría de aplicaciones (y de la computación en general) es **automatizar tareas repetitivas**. La misma pieza de código no debe repetirse continuamente.

Por ejemplo, si una página web consiste en varias páginas, es muy posible que estas tengan elementos comunes (header, navigation, footer, etc). Lo ideal es que sólo existan en un archivo concreto y se incluyan de unos a otros por herencia, como ocurre en [Twig](http://twig.sensiolabs.org/). 

* # Evita los niveles excesivos

Demasiados niveles con [estructuras de control](http://diego.com.es/estructuras-de-control-en-php) (como utilizar demasiados **if** / **else**) empeoran la legibilidad del código. 

* # Limita la longitud de línea

Nuestros ojos leen mejor columnas de texto estrechas que texto demasiado amplio (por eso los periódicos tienden hacia ese formato). Es una buena práctica evitar escribir horizontalmente grandes líneas de código.

* # Organiza bien archivos y directorios

Técnicamente es posible escribir una aplicación entera en un simple archivo, pero eso sería muy difícil de leer y de mantener.

Una buena organización en diferentes archivos y directorios mejora mucho la calidad del código. Lo ideal es emplear frameworks como Symfony que estructuran el código con estándares.

* # Nombres temporales consistentes

Normalmente las variables han de ser descriptivas y contener una o más palabras. Pero eso no se aplica necesariamente en **variables temporales**, ya que pueden ser tan cortas como de un sólo carácter. Una buena práctica es utilizar nombres consistentes en las variables temporales que tienen el mismo tipo de rol, como las siguientes:

```
// $i para contadores en loops
for ($i = 0; $i < 100; $i++) {
    // $j para contadores dentro de loops
    for ($j = 0; $j < 100; $j++) {
    }
}

// $ret para variables de return
function foo() {
    $ret['bar'] = get_bar();
    $ret['stuff'] = get_stuff();

    return $ret;
}

// $k y $v en foreach (key y value)
foreach ($some_array as $k => $v) {
}

// $q, $r y $d para mysql
$q = "SELECT * FROM table";
$r = mysql_query($q);
while ($d = mysql_fetch_assocr($r)) {
}

// $fp para apuntadores en archivos (file pointers)
$fp = fopen('file.txt','w');
```

* # Pon en mayúsculas las palabras especiales en SQL

Las **interacciones con la base de datos** es una parte importante en las aplicaciones web. Si escribes **SQL**, es importante que también sea legible. A pesar de que las **palabras especiales y funciones de SQL** no son sensibles a mayúsculas y minúsculas, es una práctica común poner en mayúscula las palabras especiales (SELECT, FROM, UPDATE, SET, WHERE, LEFT JOIN, GROUP, ORDER BY, LIMIT...).

* # Separa el código de los datos

En el caso del **desarrollo web** los **datos** suelen ser el **output o salida HTML**. Cuando PHP comenzó hace algunos años, era visto principalmente como un motor de templates. Era usual tener archivos HTML grandes con algunas líneas de PHP. Con los años esto ha cambiado y los sitios han evolucionado a construcciones más dinámicas y funcionales. El código es ahora una parte enorme de las **aplicaciones web**, y no es una buena práctica mezclarlo con el lenguaje de marcado HTML.​

Esta práctica actualmente es sencilla gracias a los frameworks como **Symfony** y a los motores de templates como **Twig**.

* # Programación orientada a objetos

La **Programación Orientada a Objetos** permite crear código bien estructurado. La mayoría de los **frameworks** y **aplicaciones web** ya aplican esta forma de construcción en prácticamente la totalidad de su código.

* # Lee código open source

El código open source de frameworks y proyectos ya construidos son ejemplos de cómo se ha de construir proyectos. Los desarrolladores respetan una serie de reglas comunes que hacen que el código sea legible, consistente y bien estructurado. Actualmente la mayoría respetan las normas establecidas en el [FIG](http://www.php-fig.org/).

* # Haz refactoring

Cuando se refactoriza el código se hacen cambios pero no se cambia su funcionamiento. Se hace "limpieza" para mejorar la legibilidad y la calidad del código. Lo ideal es hacerlo lo más pronto y frecuente posible.

### 2\. Otras prácticas fundamentales en el desarrollo web

* # Debugging

Los desarrolladores tienden a escribir el código completo y después empezar a **depurar** (_debug_) y **comprobar errores**. Aunque esta forma de hacerlo puede ahorrar tiempo en proyectos pequeños, en proyectos de mayor envergadura tienden a tener demasiadas variables y funciones que requieren de atención específica. Por esta razón es mejor **depurar cada módulo cuando se termina** y no el proyecto entero. Esto ahorra tiempo a la larga, ya que no hay que buscar dónde concretamente se está produciendo el error. Para evitar que en un futuro aparezcan bugs que sean difíciles de encontrar, se emplea el testing, y más concretamente, el **Test Driven Development**.

* # Testing

El testing es una parte integral del desarrollo web que ha de ser planificada. Consiste en la comprobación de la funcionalidad de los módulos de un proyecto o del proyecto en sí. Las comprobaciones se pueden centrar en la seguridad de la aplicación, la funcionalidad, la accesibilidad en función del rango de los usuarios, en el análisis del rendimiento ([load testing](https://en.wikipedia.org/wiki/Load_testing)), etc. 

Los tests se pueden dividir en dos: **Unit Tests**, que comprueban el código por módulos, y **Functiontal Tests**, que comprueban la funcionalidad de la aplicación como si se tratara de un usuario navegando por la misma.

Una metodología muy utilizada en la actualidad es el **Test Driven Development**, que consiste en escribir el código y tests de forma simultánea, lo que te permite que compruebes el código rápidamente, ya que es automático. El proceso que sigue el TDD es el siguiente:

1.  **Antes de escribir código, primero hay que escribir un test automático**. Mientras escribes el test automático, tienes que tener en cuenta los posibles inputs, errores y outputs. De esta forma tu mente no está cerrada por un código que ya está escrito.
2.  **El test debe fallar la primera vez** que ejecutes el test automático, indicando que el código no está construído todavía.
3.  **Comienza a escribir el código**. Al haber ya tests automáticos, siempre que el test falle significa que el código no está listo. El código tendrá que modificarse hasta que pase todas las aserciones del test.
4.  **Refactorizar el código**. Una vez que el código pasa el test, puedes comenzar a limpiarlo y refactorizar. En este proceso se puede ir comprobando, si el test funciona significa que el código funciona tras la refactorización.
5.  **Inicio de nuevo el proceso**. Se inicia otra vez el proceso con una nueva funcionalidad.

Esta imagen obtenida de [Wikipedia](https://en.wikipedia.org/wiki/Test-driven_development) representa muy bien el proceso:

![Test Driven Development](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0b/TDD_Global_Lifecycle.png/1024px-TDD_Global_Lifecycle.png)

* # Design patterns

El entendimiento y la aplicación de **patrones de diseño en el desarrollo web** ha cogido especial importancia por su uso en los **frameworks** y en la construcción de **aplicaciones bien estructuradas**. Los [patrones de diseño](http://diego.com.es/patrones-de-diseno-en-php) son simplemente esquemas organizativos de código en respuesta a problemas comunes. Para emplear estos patrones es necesario dominar la **programación orientada a objetos**.

### 3\. Desarrollo web Agile

En bastantes trabajos exigen o tienen preferencia a desarrolladores que estén familiarizados con **Desarrollo Agile**. Este tipo de desarrollo incluye la entrega del proyecto testeado de forma parcial en diferentes fases (cada dos-cuatro semanas), de forma que los requisitos y soluciones evolucionan según va pasando el tiempo. Promueve la planificación adaptativa, el desarrollo evolutivo, la entrega temprana, mejoras continuas y la rapidez y flexibilidad frente al cambio. El **manifiesto Agile** se basa en 12 principios básicos:

1.  Satisfacción del cliente con la entrega temprana y continuada del proyecto.
2.  Son bienvenidos los cambios en los requisitos, incluso una vez comenzado el proyecto.
3.  El proyecto se va entregando frecuentemente (preferiblemente en semanas mejor que meses).
4.  Cooperación cercana y diaria entre los operativos y los desarrolladores.
5.  Los proyectos se construyen con individuos motivados, en los que se deposita una confianza.
6.  La conversación cara a cara es la mejor forma de comunicación.
7.  El proyecto funcional es la principal medida del progreso.
8.  Desarrollo sostenible, dispuesto a mantener un ritmo continuado.
9.  Continua atención a la excelencia técnica y al buen diseño.
10.  Simplicidad, el arte de maximizar la cantidad de trabajo sin hacer, es esencial.
11.  Equipos que se auto organizan.
12.  Adaptación regular a los cambios circunstanciales.

Cada entrega (iteración), incluye una planificación, un análisis de requisitos, diseño, codificación, tests y documentación. La idea es que en cada iteración el proyecto funcione (que no tenga errores), y que por tanto vaya ganando valor.

Este método también tiene críticas, pues no es un método que sea aplicable en todas las compañías o situaciones. Agile development puede no resultar satisfactorio en **grandes organizaciones** (por no adaptar bien los equipos a este tipo de desarrollo) o en **organizaciones gubernamentales** (uso de metodologías antiguas que frenan la aplicación correcta de Agile).