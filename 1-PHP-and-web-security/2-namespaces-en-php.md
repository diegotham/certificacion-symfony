Los namespaces o **espacios de nombres** permiten crear aplicaciones complejas con mayor flexibilidad evitando problemas de conflictos entre clases y mejorando la legibilidad del código.

**Indice de contenido**

| | |
| -------- | -------- |
| 1. Introducción | 5. Importar un namespace |
| 2. Namespaces dinámicos | 6. Apodar un namespace |
| 3. La constante __NAMESPACE__ | 7. Espacio global en los namespaces |
| 4. La palabra reservada namespace | 8. Reglas de resolución de los namespaces |

### 1. Introducción

Un namespace no es más que un directorio para **clases**, **traits**, **interfaces**, **funciones** y **constantes**. Se crean utilizando la palabra reservada _namespace_ al principio del archivo, antes que cualquier otro código, a excepción de la **estructura de control declare**.

Supongamos la siguiente clase:

```
// MiClase.php
class MiClase {
// ...codigo
}
```

Si queremos instanciar la clase desde otro archivo bastará con escribir:

```
include 'MiClase.php';

$miClase = new MiClase;
```

Con un proyecto sencillo no habría dificultades con esta metodología. Los problemas pueden venir cuando el proyecto aumenta y puede ocurrir que coincidan clases, funciones o constantes de PHP o de **librerías de terceros** con las del propio proyecto.

Supongamos ahora que _coche.php_ está en un nuevo directorio:

```
// Directorio: Proyecto/Prueba/MiClase.php
namespace Proyecto\Prueba;

class MiClase {
//...
}
```

Para **utilizar la clase**:

```
include 'Proyecto/Prueba/MiClase.php';

$miClase = new Proyecto\Prueba\MiClase;
```

Siempre se emplea como _namespace_ el **directorio de la clase**. No es obligatorio pero es lo más recomendable.

Los **namespaces** proporcionan una forma de agrupar clases, interfaces, funciones y constantes relacionadas.

#### **Varios namespaces en un archivo**

**Al igual que se debe crear sólo una clase por archivo, también se debe crear sólo un namespace**. En caso de definir varios namespaces por archivo, conviene utilizar corchetes:

```
<?php
namespace MiEspacio {
const CONECTAR_OK = 1;
class Conexion() { //... }
function conectar() { // ...}
}

namespace OtroEspacio {
//...
}

// Para incluir código que no tiene namespace en archivos donde sí existe código
// con algún namespace, se deja vacío el nombre:

namespace {
// Código del archivo sin namespace
}
```

### 2. Namespaces dinámicos

Para **acceder dinámicamente a un namespace**:

```
namespace Proyecto;
// Definimos una clase, una función y una constante bajo el namespace Proyecto
class MiClase {
    function __construct()
    {
        echo __METHOD__ . "<br>";
    }
}
function miFuncion()
{
    echo __FUNCTION__ . "<br>";
}
const MICONSTANTE = "Constante para namespaces <br>";
// En el mismo archivo llamándolos directamente:
$x = new MiClase; // Devuelve: Proyecto\MiClase::__construct
miFuncion(); // Devuelve: Proyecto\miFuncion
echo MICONSTANTE; // Devuelve: Constante para namespaces
// Si en cambio se inicia la clase, función o contante desde un string:
$x = 'MiClase';
$objeto = new $x; // Fatal error Class MiClase not found
$y = 'miFuncion';
$y(); // Fatal error: Call to undefined function miFuncion()
echo constant('MICONSTANTE'); // Warning: constant(): Couldn't find constant MICONSTANTE

```

La razón por la que no funcionan mediante llamamientos dinámicos es porque dependen de si las llamadas se traducen durante la compilación o no, lo que se aclara en la última sección: _8\. Reglas de resolución de los namespaces_. Continuando con el ejemplo anterior:

```
// Si obtenemos la clase con el namespace completo desde un string:
$x = 'Proyecto\MiClase';
$objeto = new $x; // Devuelve: Proyecto\MiClase::__construct
$y = 'Proyecto\miFuncion';
$y(); // Devuelve: Proyecto\miFuncion
echo constant('Proyecto\MICONSTANTE') . "<br>"; // Devuelve: Constante para namespaces
// No importa incluir o no la barra invertida inicial:
$x = '\Proyecto\MiClase';
$objeto = new $x; // Devuelve: Proyecto\MiClase::__construct
$x = 'Proyecto\MiClase';
$objeto = new $x; // Devuelve: Proyecto\MiClase::__construct
// Aunque sí que importa si se define directamente la clase en el mismo archivo:
$x = new \Proyecto\MiClase; // Devuelve: Proyecto\MiClase::__construct
$x = new Proyecto\Miclase; // Fatal error: Class 'Proyecto\Proyecto\Miclase' not found
// Esto último también se clarifica en la sección sobre las reglas de resolución de los namespaces, concretamente en la regla número 3
```

**Importante** escapar con barra invertida los namespaces dentro de una cadena de comillas dobles:

```
<?php
$x = "proyecto\nombre"; // \n es una nueva línea con comillas dobles
$x = "proyecto\\nombre"; // ahora si que lo detectará correctamente
```

### 3. La constante __NAMESPACE__

\_\_NAMESPACE\_\_ es una constante mágica que devuelve un string con el nombre del namespace actual:

```
<?php
namespace MiProyecto;

echo "Namespace actual: " . __NAMESPACE__;
```

Esta constante se puede utilizar para **construir nombres dinámicamente**:

```
<?php
namespace MiProyecto;

function obtener($nombreClase){
    $x = __NAMESPACE__ . '\\' . $nombreClase;
    return new $x;
}
```

### 4. La palabra reservada namespace

La palabra reservada **namespace** es equivalente a **self** en las **clases**. Se utiliza para solicitar un elemento del namespace actual:

```
<?php
namespace MiProyecto;

namespace\miFuncion(); // llama a MiProyecto\miFuncion()
namespace\MiClase::metodo(); // llama al método estático metodo() de la clase MiClase. Equivale a MiProyecto\Miclase::metodo()
namespace\CONSTANTE // llama a la constante CONSTANTE como MiProyecto\CONSTANTE
```

### 5. Importar un namespace

Si tenemos que utilizar una clase varias veces en un archivo, podemos evitar tener que escribir el _namespace_ tantas veces **importándolo**, así después solo habrá que escribir la clase:

```
include 'Proyecto/Prueba/MiClase.php';

use Proyecto\Prueba\MiClase;

$miClase = new MiClase;
```

**La importación se realiza durante la compilación** y no durante la ejecución, por eso se declara en el **ámbito global** de un archivo. 

### 6. Apodar un namespace

También es posible utilizar un **alias** para utilizar una clase bajo un nombre diferente: 

```
include 'Proyecto/Prueba/MiClase.php';

use Proyecto\Prueba\MiClase as Clase;

$miClase = new Clase; // instancia un objeto de la clase Proyecto\Prueba\MiClase

// No es posible con nombres dinámicos
$x = 'MiClase';
$objeto = new $x; // instancia de la clase MiClase. No detecta el apodo

```

### 7. Espacio global en los namespaces

Cuando se emplean **namespaces**, hay que tener en cuenta las **clases globales**. En un archivo donde se emplea un namespace, si se crea una clase **Datetime()** se instanciará la clase del mismo:

```
class Datetime extends \Datetime {
    public function getTimestamp()
    {
        echo "¡Hola!";
    }
}

$propia = new Datetime();
echo $propia->getTimestamp(); // Devuelve !Hola!
```

Para **instanciar la clase global** es necesario emplear \ delante de Datetime() al instanciarla:

```
class Datetime extends \Datetime {
    public function getTimestamp()
    {
        echo "¡Hola!";
    }
}

$propia = new \Datetime();
echo $propia->getTimestamp(); // Devuelve el timestamp en formato Unix
```

### 8. Reglas de resolución de los namespaces

**Definiciones importantes**:

*   **Unqualified name** - identificador sin separador: MiProyecto
*   **Qualified name** - identificador con separador: MiProyecto\Prueba
*   **Fully qualified name** - nombre cualificado que comienza con un separador: \MiProyecto\Prueba

**Reglas**:

1.  Las llamadas a clases, funciones o constantes con **fully qualified name** se resuelven durante la compilación. Ej: new \MiProyecto\Prueba será la clase MiProyecto\Prueba.
2.  Los **qualified** y **unqualified names** se traducen durante la compilación según las reglas de importación (como la generación de **alias**). Ej: MiProyecto\Prueba as Prueba, una llamada a Prueba\Test\miFuncion se traduce como MiProyecto\Prueba\Test\miFuncion.
3.  Dentro de un _namespace_, todos los nombres **qualified** no traducidos llevan antepuesto el _namespace_ actual. Ej: si una llamada a Prueba\Test\miFuncion se lleva a cabo en el _namespace_ MiProyecto, se traduce como MiProyecto\Prueba\Test\miFuncion.
4.  Los _namespaces_ **unqualified** se traducen durante la compilación según las reglas de compilación. Ej: si MiProyecto\Prueba\Test se importa como Test, new Test() se traduce como new MiProyecto\Prueba\Test().
5.  Dentro de un _namespace_ (MiProyecto\Prueba), las llamadas a **funciones** **unqualified** se resuelven durante la ejecución. Ej: si se llama a miFuncion():
    - 1\. Se busca en el namespace actual MiProyecto\Prueba\miFuncion
    - 2\. Se intenta encontrar y llamar a la función global miFuncion
6.  Dentro de un namespace, MiProyecto\Prueba, los _namespaces_ **qualified** y **unqualified** se resuelven durante la ejecución. Ej:
    Si se llama a new Test():
    - 1\. Se busca la clase en el namespace actual MiProyecto\Prueba\Test
    - 2\. Se intenta cargar automáticamente MiProyecto\Prueba\Test
    Si se llama a new Otro\otraFuncion():
    - 1\. Se busca la clase en el namespace actual MiProyecto\Prueba\Otro\otraFuncion()
    - 2\. Se intenta cargar automáticamente MiProyecto\Prueba\Otro\otraFuncion()
7.  Para referirse a **cualquier clase global** en el **_namespace_ global**, se debe emplear el **fully qualified name** new \miFuncion().