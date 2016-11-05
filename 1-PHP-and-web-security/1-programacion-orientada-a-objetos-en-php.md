La programación orientada a objetos (**Object Oriented Programming OOP**) es un **modelo de lenguaje de programación** organizado por **objetos** constituidos por **datos y funciones**, entre los cuales se pueden crear relaciones como herencia, cohesión, abstracción, polimorfismo o encapsulamiento. Esto permite que haya una gran flexibilidad y se puedan crear **objetos** que pueden heredarse y transmitirse sin necesidad de ser modificados continuamente.

**Indice de contenido**

| | |
| -------- | -------- |
| 1. Clases y Objetos | 6. Propiedades y métodos estáticos |
| 2. Interacciones entre objetos | 7. Traits |
| 3. Public, protected y private | 8. Type hinting |
| 4. Paamayim Nekudotayim | 9. Métodos mágicos |
| 5. Constantes | |

### 1. Clases y objetos

Una **clase** define los datos y la lógica de un objeto. La lógica se divide en funciones (**métodos**) y variables (**propiedades**).

Para definir una propiedad:

```
class Coche {
    public $color;
}
```

Para definir un método:

```
class Coche {
    public function getColor()
    {
        // Contenido de la función
    }
}
```

Para **crear un objeto** hay que **instanciar una clase**: 

```
class Coche {
 // Contenido de la clase
}

$miCoche = new Coche(); // Objeto
```

La **clase** es como una **plantilla** que define características y funciones. El **objeto** agrupa los datos de la clase y permite utilizarlos desde una unidad.

Podemos definir las características de un coche de la siguiente forma:

```
class Coche {
    public $color;
    public $potencia;
    public $marca;
}

$miCoche = new Coche();
$miCoche->color = 'rojo';
$miCoche->potencia = 120;
$miCoche->marca = 'audi';
```

De momento sólo hemos definido **propiedades** del objeto. Si queremos mostrar alguna característica ahora:

```
//...
echo 'Color del coche: ' . $miCoche->color; // Muestra Color del coche: rojo
```

Ahora vamos a definir un **método que devuelve una propiedad**:

```
class Coche {
    //...

    public function getColor()
    {
        return $this->color;
    }
}
//...
```

El método _**getColor()**_ nos permite devolver el color del objeto instanciado. La variable _**$this**_ se puede utilizar en cualquier método, y hace referencia al objeto que hemos instanciado, en este caso $miCoche. Podemos obtener el mismo resultado que antes para mostrar el color del coche pero ahora utilizando un **método** para mostrar la **propiedad **color:

```
//...
echo 'Color del coche: ' . $miCoche->getColor();
```

Las **propiedades** pueden tener **valores por defecto**:

```
class Coche {
    public $color = 'rojo';
//...
}
```

De esta forma siempre que se instancie un objeto de la clase **Coche**, éste será de color rojo a no ser que se modifique después. Ahora creamos también el objeto _$otroCoche_ y le ponemos otras características:

```
class Coche {

    public $color = 'rojo';
    public $potencia;
    public $marca;

    public function getColor()
    {
        return $this->color;
    }
}

$miCoche = new Coche();
$miCoche->color = 'verde';
$miCoche->potencia = 120;
$miCoche->marca = 'audi';

$otroCoche = new Coche();
$otroCoche->color = 'azul';
$otroCoche->potencia = 100;
$otroCoche->marca = 'bmw';

```

Creamos ahora dos "_getters_" para la potencia y la marca:

```
class Coche {

    public $color = 'rojo';
    public $potencia;
    public $marca;

    public function getColor()
    {
        return $this->color;
    }
    public function getPotencia()
    {
        return $this->potencia;
    }
    public function getMarca()
    {
        return $this->marca;
    }
}
```

Y creamos una función (fuera de la clase) que devuelve las caracteristicas totales de un objeto coche:

```
function printCaracteristicas($cocheConcreto)
    {
        echo 'Color: '. $cocheConcreto->getColor();
        echo "\n";
        echo 'Potencia: '. $cocheConcreto->getPotencia();
        echo "\n";
        echo 'Marca: '. $cocheConcreto->getMarca();
    }

$miCoche = new Coche();
$miCoche->color = 'verde';
$miCoche->potencia = 120;
$miCoche->marca = 'audi';

$otroCoche = new Coche();
$otroCoche->color = 'azul';
$otroCoche->potencia = 100;
$otroCoche->marca = 'bmw';

printCaracteristicas($miCoche);
echo "\n";
printCaracteristicas($otroCoche);

```

La función _printCaracteristicas()_ requiere un argumento, _$cocheConcreto_, que es una instancia de la clase Coche que mostrará las características del propio objeto.

### 2. Interacciones entre objetos

Supongamos ahora que queremos comparar y mostrar **qué coche es más rápido** dependiendo de su potencia. Añadimos un nuevo **método** a la clase **Coche**:

```
class Coche {
    //...

    public function elCocheElegidoEsMasRapido($cocheElegido)
    {
        return $cocheElegido->potencia > $this->potencia;
    }
}
//...
```

Tenemos dos **objetos** _$miCoche_ y _$otroCoche_. La condición que mostrará que coche es más rápido:

```
//...
if ($miCoche->elCocheElegidoEsMasRapido($otroCoche)) {
    echo 'El ' . $otroCoche->marca . ' es más rápido';
} else {
    echo 'El ' . $miCoche->marca . ' es más rápido';
}

$miCoche->elCocheElegidoEsMasRapido($otroCoche);
// En este caso el Audi ($miCoche) es más rápido que el BMW ($otroCoche)
```

Podemos **comparar la igualdad entre dos objetos**, con los operadores de **comparación** (==) y de **identidad** (===):

*   Dos objetos serán **iguales** si tienen los **mismos atributos y valores**, siendo instancias de la misma clase. En ejemplo anterior, los atributos de los dos objetos deberían tener los mismos valores, por ejemplo: verde, 100, audi.
*   Dos objetos serán **idénticos** si además hacen referencia a la **misma instancia de la misma clase**. 

### 3. Public, protected y private

Anteriormente hemos definido todas las propiedades y métodos como _**public**_. Esto significa que pueden ser accedidos y alterados desde cualquier parte fuera de la clase. En el ejemplo anterior hemos podido definir _$miCoche->potencia_ sin ningún problema, y podríamos aplicarle cualquier valor (un número, un string...).

Cuando se define una propiedad como _**private**_, se indica que ésta no puede verse o modificarse a no ser que sea desde la propia clase. Si utilizamos _$miCoche->potencia_ para verla o definirla, nos dará error. Si queremos definirla, se utiliza un _**setter**_, un **método public** para definir una **propiedad private**:

```
class Coche {

    //...

    public function setPotencia($potencia)
    {
        $this->potencia = $potencia;
    }
    //...
}
```

Así podemos definir desde fuera de la clase el valor de la propiedad potencia:

```
//...
$miCoche->setPotencia(120);
```

También deberemos de tener un _**getter**_ para poder acceder al valor que tiene establecido la propiedad:

```
    //...
    public function getPotencia()
    {
        return $this->potencia;
    }
    //...
```

Con esto conseguimos, entre otras cosas, que puedan definirse **valores limitados**. En este caso, la propiedad _$potencia_ queremos que pueda ser sólo un **número**. Para ello, modificamos el _**setter**_ anterior:

```
    //...
    public function setPotencia($potencia)
    {
        if(!is_numeric($potencia)){
            throw new \Exception('Potencia no válida: '. $potencia);
        }
        // Si $potencia no es un número, devolverá un error
        $this->potencia = $potencia;
    }
    //...
```

Para entender las propiedades y métodos bajo la **palabra reservada protected**, vamos a explicar primero las relaciones entre clases mediante la **herencia**.

Una clase puede **heredar métodos y propiedades** de otra clase, y éstos se pueden sobreescribir empleando el mismo nombre que en la clase madre. Las clases sólo pueden heredar de una única clase (no es posible **herencia múltiple**), pero sí que pueden heredarse unas a otras (sí es posible **herencia multinivel**). La herencia se realiza mediante la palabra _**extends**_.

Supongamos ahora la clase **Coche** con **3 tipos de propiedades** y **CocheDeLujo** como heredera:

```
class Coche {
    private $color = 'red';
    protected $potencia = 120;
    public $marca = 'audi';
}
class CocheDeLujo extends Coche {

    // La función displayColor devolverá un error porque es private
    public function displayColor()
    {
        return $this->color;
    }
    // La función displayPotencia devolverá 120, ya que hereda el valor
    public function displayPotencia()
    {
        return $this->potencia;
    }
    // La función displayMarca devolverá audi, ya que también hereda el valor
    public function displayMarca()
    {
        return $this->marca;
    }
}
```

La clase **CocheDeLujo** podrá emplear y modificar las propiedades y métodos **protected** y **public** de la madre, pero no las **private**. Además, podrá añadir cualquier propiedad o **métodos complementarios**. Creamos varios métodos en la clase Coche, incluído _printCaracteristicas()_:

```
class Coche {
    protected $color;
    public function setColor($color)
    {
        $this->color = $color;
    }
    public function getColor()
    {
        return $this->color;
    }
    public function printCaracteristicas()
    {
        echo 'Color: '. $this->getColor;
    }
}
```

Añadimos otros métodos en CocheDeLujo, y modificamos la herencia de _printCaracteristicas()_:

```
class CocheDeLujo extends Coche {
    protected $extras;
    public function setExtras($extras)
    {
        $this->extras = $extras;
    }
    public function getExtras()
    {
        return $this->extras;
    }
    public function printCaracteristicas()
    {
        echo 'Color: '. $this->color;
        echo '<hr/>';
        echo 'Extras: ' . $this->extras;
    }
}

$miCoche = new CocheDeLujo();
$miCoche->setColor('negro');
$miCoche->setExtras('TV');

$miCoche->printCaracteristicas(); // Devuelve Color : negro Extras: TV
```

El objeto _$miCoche_, que es una instancia de la clase **CocheDeLujo**, puede determinar la nueva propiedad _$extras_ además de las anteriores. El método que se ha heredado se ha modificado para añadir la nueva característica. 

**Accesos que permiten las palabras reservadas public, protected y private en métodos y propiedades:**

Private:

*   Desde la misma clase que declara

Protected:

*   Desde la misma clase que declara
*   Desde las clases que heredan esta clase

Public:

*   Desde la misma clase que declara
*   Desde las clases que heredan esta clase
*   Desde cualquier elemento fuera de la clase

Es posible **impedir que un método pueda sobreescribirse** mediante la palabra **final**:

```
class Coche {
    final public function getColor()
    {
        echo "Rojo";
    }
}
class CocheDeLujo extends Coche {
    public function getColor()
    {
        echo "Azul";
    }
} // Dará un error fatal, getColor() no puede sobreescribirse
```

También se puede **impedir que la clase pueda heredarse** mediante la misma palabra:

```
final class Coche {
    public function getColor()
    {
        echo "Rojo";
    }
}
class CocheDeLujo extends Coche {
    // Error fatal, clase no heredable
}
```

### 4. Paamayim Nekudotayim

Se le llama **Paamayim Nekudotayim**, operador de resolución de ámbito o doble dos puntos "::" (_**double colon**_) al operador que permite acceder a **constantes** y a **métodos estáticos** además de poder **sobreescribir propiedades o métodos de una clase**. En los dos siguientes apartados se explican las **constantes** y los **métodos estáticos**.

### 5. Constantes

Las **constantes** son **valores invariables**. Se definen mediante la palabra _**const**_ y se obtienen utilizando el operador **double colon**: _$objeto::CONSTANTE _o _NombreClase::CONSTANTE_. No van precedidas del símbolo $ y se escriben en mayúscula:

```
class Coche {
    const RUEDAS = 4;
}
// Obtener el valor mediante el nombre de la clase:
echo Coche::RUEDAS . "\n";
// Obtener el valor mediante el objeto:
$miCoche = new Coche();
echo $miCoche::RUEDAS . "\n";
```

### 6. Propiedades y métodos estáticos

Las **propiedades y métodos estáticos** permiten que sean accesibles sin la necesidad de instanciar la clase. Es importante saber que no se puede acceder a una **propiedad static** con un **objeto instanciado**, aunque sí a un **método static**.

No es posible usar la **pseudo-variable $this**, ya que los métodos estáticos pueden ejecutarse sin tener una instancia del objeto, y _$this_ hace referencia al objeto creado.

```
class Coche {
    public static $color = 'rojo';

    public function mostrarColor()
    {
        return self::$color;
    }
}

print Coche::$color . "\n"; // Muestra "rojo"

// Creando el objeto miCoche:
$miCoche = new Coche();
print $miCoche->mostrarColor() . "\n"; // Muestra "rojo"
print $miCoche->color . "\n"; // Error, propiedad color no definida
print $miCoche::$color . "\n"; // Muestra "rojo"
```

En este ejemplo hemos visto que **no** se puede obtener la **propiedad** _$color _desde la instancia del objeto. En el siguiente ejemplo se puede obtener un **método** instanciando la clase o no:

```
class Coche {
    public static function mostrarColor()
    {
        echo 'Rojo';
    }
}
Coche::mostrarColor(); // Muestra Rojo
$miCoche = new Coche;
$miCoche->mostrarColor(); // Muestra Rojo
```

### 7. Traits

Los **traits** son como **clases** que agrupan funcionalidades diseñadas para ser reutilizadas en otras clases y así evitar las limitaciones de la **herencia simple**. A diferencia de las clases, **no se pueden instanciar**, tan sólo permiten utilizar sus funciones específicas en otras clases. Los **traits** permiten relaciones horizontales entre clases evitando así la verticalidad.

En el siguiente ejemplo se utiliza un **trait** para mostrar el modelo de coche. En la clase **Ventas** se incluye _use Modelo_, este **trait** tiene una función _getModelo()_ que presupone que en la clase donde se vaya a utilizar existe un método _getMarca()_ de la clase **Coche**. Los métodos de la clase **Ventas** sobreescriben los métodos del **trait**, y éstos a su vez sobreescriben la clase **Coche**:

```
class Coche {
    public function getMarca() {
        echo 'Renault ';
    }
}
trait Modelo {
    public function getModelo() {
        parent::getMarca();
        echo 'Clio';
    }
}
class Ventas extends Coche {
    use Modelo;
}

$o = new Ventas();
$o->getMarca(); // Podemos obtener este método de la clase Coche
$o->getModelo(); // Podemos obtener este método del trait Modelo
```

 Se pueden emplear **múltiples traits** añadiéndolos a la clase:

```
class Ventas extends Coche {
    use Model, Serie, Versión;
    //...
}
```

O incluso **emplear múltiples traits en otro trait** para luego usarlo en la clase:

```
trait Coche {
    public function getMarca() {
        echo 'Renault ';
    }
}
trait Modelo {
    public function getModelo() {
        echo 'Clio';
    }
}
trait CocheModelo {
    use Coche, Modelo;
}

class Venta {
    use CocheModelo;
}

$o = new Venta();
$o->getMarca();
$o->getModelo();
```

Los **traits también pueden incluir propiedades y métodos estáticos.**

### 8. Type hinting

El **Type hinting**, o la **implicación de tipos**, permite determinar el tipo de **parámetro** que un **método** va a devolver: **objeto**, **array**, **interfaz** o **callable**. No añade ningún tipo de funcionalidad, pero es muy útil por ejemplo para saber si cometes algún error al añadir argumentos con valores incorrectos.

En el siguiente ejemplo al argumento de la función _listadoCochesEnVenta(Venta $venta)_ se le exige que sea una instancia de la clase **Venta**. Si se pasara cualquier otro argumento, daría error:

```
class Venta {
    public $coche = 'Audi';
}

class Coche {
    public function listadoCochesEnVenta(Venta $venta)
    {
            echo $venta->coche;
    }
}
$venta = new Venta;
$coche = new Coche;
$coche->listadoCochesEnVenta($venta); // Devuelve Audi

```

Si se pusiera, por ejemplo _$coche->listadoCochesEnVenta('Audi')_ daría error, ya que no es una clase instanciada.

En el siguiente ejemplo, vamos a **exigir un array**:

```
class Coche {
    public function listadoCochesEnVenta(Array $listado)
    {
        foreach ($listado as $coche){
            echo $coche . "\n";
        }
    }
}
$listaCoches = ['Audi', 'BMW'];

$coche = new Coche;
$coche->listadoCochesEnVenta($listaCoches); // Devuelve el listado Audi BMW
```

Si el argumento de _listadoCochesEnVenta()_ no fuera un array, también daría error

### 9. Métodos mágicos

Los **métodos mágicos** permiten reaccionar a **eventos** que le puedan ocurrir a un objeto instanciado. Se inician con una doble barra baja, y son: **__construct()**, **__destruct()**, __call(), __callStatic(), **__get()**, **__set()**, **__isset()**, **__unset()**, __sleep(), __wakeup(), **__toString()**, __invoke(), __set_state(), __clone() y __debugInfo(). Tan se introducen algunos de ellos:

El más utilizado es el **constructor __construct()**, que se ejecuta cuando el objeto se crea, pudiendo inyectar **parámetros** y **dependecias** requeridos para crear el objeto.

```
class Coche {
    public $color;

    public function __construct($color)
    {
        $this->color = $color;
    }
}

$miCoche = new Coche('Rojo');
echo $miCoche->color; // Devuelve Rojo
```

Los constructores también se heredan. Si una clase hija define un constructor propio, para heredar el de la madre es necesario indicarlo con _**parent::__construct()**_:

```
class Coche {
    public $color;

    public function __construct($color)
    {
        $this->color = $color;
    }
}
class CocheDeLujo extends Coche {
    public $extras;

    public function __construct($color, $extras)
    {
        parent::__construct($color);
        $this->extras = $extras;
    }
}

$miCoche = new CocheDeLujo('Rojo', 'TV');
echo $miCoche->color; // Devuelve Rojo
echo $miCoche->extras; // Devuelve TV
```

El método **__destruct()** hace lo contrario, se ejecuta cuando un objeto se destruye, aunque apenas se utiliza. 

Los métodos mágicos **__get()** y **__set()** actúan como **getters** y **setters** para propiedades que no tienen **visibilidad public**.

El método **__isset() **actúa como la **función** isset()** normal, como cuando se trabaja con arrays o variables. Este método mágico permite comprobar la existencia de una **propiedad no pública**. El método **__unset()** también actúa como la **función unset()** normal pero para propiedades no públicas.

El método mágico **__toString()** permite **devolver el objeto en forma de string**:

```
class Post {
    public $title;

    public function setTitle($title)
    {
        $this->title = $title;
    }
    public function __toString()
    {
        return $this->title;
    }
}

$post = new Post;
$post->setTitle('Este es mi primer artículo');
echo $post; // Devuelve Este es mi primer artículo
```
