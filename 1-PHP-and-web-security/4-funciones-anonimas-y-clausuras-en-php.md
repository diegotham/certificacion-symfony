Las **funciones anónimas** son funciones sin nombre:

```
function() {
    return "Hola";
};
```

Se puede **llamar a una función anónima** de dos formas: utilizando la sintaxis de funciones variables o llamarla desde otra función en forma de argumento.

Empleando la sintaxis de funciones variables, el valor de la variable es la función:

```
$saludo = function() {
    return "Hola que tal";
};

echo $saludo(); // Devuelve: Hola que tal
```

Añadiendo la función en otra:

```
function decir ($algo) {
    echo $algo();
}

decir(function(){
    return "Esto es algo";
});
```

La función _decir()_ es una **función regular normal**, cuyo argumento es _$algo_. _decir()_ imprimirá lo que devuelva la función _$algo()_. Al ejecutar _decir()_ le pasamos el argumento: una **función anónima** que devuelve "Esto es algo".

Una **función anónima** no es más que una función que no tiene nombre, y por tanto no está enlazada a ningún identificador.

Una **clausura** o **closure** es una función anónima que captura el _scope_ actual, y proporciona acceso a ese _scope_ cuando se invoca el closure.

Las **clausuras** permiten **usar variables** mediante la palabra _use_:

```
$colorCoche = 'rojo';

$mostrarColor = function() use ($colorCoche) {
    echo "El color del coche es $colorCoche";
};

$mostrarColor(); // Mostrará El color del coche es rojo
```

Cuando se emplea _use_, se **hereda una variable del ámbito padre**. Esto no es lo mismo que usar **variables globales**. El **ámbito padre** de una clausura es el ámbito de la función en el que la clausura fue declarada (no desde la función desde la que se llamó).

Si se altera el valor de la variable _$colorCoche_ dentro de la **clausura**, no afectará a la variable original:

```
$colorCoche = 'rojo';

$mostrarColor = function() use ($colorCoche) {
    $colorCoche = 'azul';
};

$mostrarColor();
echo $colorCoche; // Mostrará rojo
```

Si se quiere que su valor se altere, se pasa por referencia añadiéndole un **ampersand** **&** delante:

```
$colorCoche = 'rojo';

$mostrarColor = function() use (&$colorCoche) {
    $colorCoche = 'azul';
};

$mostrarColor();
echo $colorCoche; // Mostrará azul
```

También es posible **añadir argumentos en las clausuras**:

```
// Las clausuras también aceptan argumentos normales
$mensaje = "¿Qué tal?";
$saludar = function ($saludo) use ($mensaje) {
    echo $saludo . ', ' . $mensaje;
};
$saludar("Hola"); // Devuelve: Hola, ¿Qué tal?
```

La **principal utilidad de las funciones anónimas** es como **parámetro** de los [callbacks](http://php.net/manual/es/language.types.callable.php), que son funciones llamadas por otra función que usa a la primera como parámetro. Se llama a un **callback** cuando ocurre un **evento**. Hay algunas **funciones PHP que usan un callback como argumento**, como _array_map_, _array_filter_ o _array_walk_. La más sencilla es [_call_user_func_](http://php.net/manual/es/function.call-user-func.php), cuyo primer argumento es el callback y el segundo son los argumentos del primero:

```
$saludo = function($nombre)
{
    printf("Hola %s\r\n", $nombre);
};

$saludo('Carlos'); // Devuelve Hola Carlos
call_user_func($saludo, "PHP"); // Devuelve Hola PHP
```

Se puede llamar a **funciones simples** o a **métodos** de un objeto:

```
function mostrarColor() {
    echo 'Rojo';
}
class Numeros {
    static function mostrarNumero()
    {
        echo "5";
    }
}

call_user_func('mostrarColor');  // Muestra rojo

call_user_func(array('Numeros', 'mostrarNumero')); // Muestra 5
// Esta forma de llamar al método es estática.
// Si el método no fuera static, con esta forma generaría un error
// Strict Standards, non-static method should not be called statically

// Llamada a un método estático o no estático:
$numeros = new Numeros;
call_user_func(array($numeros, 'mostrarNumero')); // Muestra 5

// Llamada a un método estático:
call_user_func('Numeros::mostrarNumero'); // Muestra 5
```

Otra función que también emplea **callbacks** es _[array_walk](http://php.net/manual/es/function.array-walk.php)_, que aplica **una función a cada miembro de un array**:

```
class Coche
{
    protected $coches = array();
    public function add($coche, $potencia)
    {
        $this->coches[$coche] = $potencia;
    }
    public function mostrarCoches()
    {
        $callback = function ($potencia, $coche) {
            echo $coche . " -> " . $potencia . "\n";
        };
        array_walk($this->coches, $callback);
    }
}
$coche = new Coche;
$coche->add('Audi', '200');
$coche->add('BMW', '220');
$coche->add('Mercedes', 250);
// Empleamos el método mostrarCoches()
$coche->mostrarCoches(); // Devuelve: Audi -> 200 BMW -> 220 Mercedes -> 250
```

En la función _array_walk_, el primer parámetro es un **array**, y el segundo un **callback** que suele aceptar dos **parámetros**, el primero es el **valor** (_$potencia_) y el segundo la **clave** (_$coche_). El **callback** también puede aceptar sólo un parámetro:

```
<?php
$multiplicador = 2;

$numeros = [2, 4, 6, 8];

array_walk($numeros, function($numero) use ($multiplicador){
    echo $numero * $multiplicador . "\n";
}); // Devuelve: 4 8 12 16
```

En el artículo de **funciones anónimas** de la **documentación PHP** hay un buen ejemplo de la relación entre clausuras y ámbitos, es el siguiente:

```
class CarroDeLaCompra {
    const PRECIO_BROCOLI = 1.00;
    const PRECIO_PIMIENTO = 0.40;
    const PRECIO_CALABACIN = 0.60;

    protected $productos = array();

    public function añadir($producto, $cantidad)
    {
        $this->productos[$producto] = $cantidad;
    }
    public function obtenerTotal($impuesto)
    {
        $total = 0.00;

        $callback = function ($cantidad, $producto) use ($impuesto, &$total)
        {
            $precioUnidad = constant(__CLASS__ . "::PRECIO_" .
                strtoupper($producto));
            $total += ($precioUnidad * $cantidad) * ($impuesto + 1.0);
        };

        array_walk($this->productos, $callback);

        return round($total, 2);
    }
}
$miCarro = new CarroDeLaCompra();
// Añadir artículos al carro de la compra
$miCarro->añadir('brocoli', 2);
$miCarro->añadir('pimiento', 4);
$miCarro->añadir('calabacin', 3);
// Devolver el total con un impuesto del 5%
echo $miCarro->obtenerTotal(0.05) . "<br>";
// Resultado: 5.67 
```

Se puede usar **$this** en las **funciones anónimas**, además de funciones como _func_num_args()_, _func_get_arg()_ y _func_get_arg()_ desde dentro de la clausura.

Los **closures** son realmente objetos, instancias de la clase [Closure](http://php.net/manual/es/class.closure.php). Esta clase dispone de los métodos _bind()_ y _bindTo()_, que permiten duplicar una clausura con un objeto vinculado y ámbito de clase nuevos.

```
public Closure Closure::bindTo (object $newthis [,mixed $newscope = "static" ])
```

Lo que hace es crear un nuevo _**closure**_ en el scope del nuevo objeto _$newthis_, por lo que tendrá los valores de este nuevo objeto:

```
class ClaseA {
    function __construct($val) {
        $this->val = $val;
    }
    function getClosure() {
        // Devuelve la clausura vinculada a este objeto y el ámbito
        return function() { return $this->val; };
    }
}

$objeto1 = new ClaseA(1);
$objeto2 = new ClaseA(2);

// Creamos el closure de forma normal
$closure = $objeto1->getClosure();
echo $closure(), "\n"; // Devuelve 1

// Ahora creamos el closure desde el anterior, vinculándolo a otro objeto
$closure = $closure->bindTo($objeto2);
echo $closure(), "\n"; // Devuelve 2
```

Para ver un poco mejor sus posibilidades en la práctica, vamos a crear una claseUno con propiedades privadas:

```
class ClaseUno
{
    private $privadaUno = "Valor Uno";
    private $privadaDos = "Valor Dos";
    public function obtenerPropiedades()
    {
        return 'Las propiedades privadas son: '.
        $this->privadaUno . ' y ' . $this->privadaDos;
    }
}
$claseUno = new ClaseUno();
echo $claseUno->obtenerPropiedades() . "\n"; // Devuelve Valor 1 Valor 2
```

En el mismo archivo vamos a crear ClaseDos, donde vamos a acceder a esas propiedades privadas mediante _binding_:

```
class ClaseDos
{
    public function obtenerPropiedadUno()
    {
        $claseUno = new claseUno();
        $closure = function(ClaseUno $claseUno) {
            return $claseUno->privadaUno;
        };
        $bind = Closure::bind($closure, null, $claseUno);
        return $bind($claseUno);
    }
    public function obtenerPropiedadDos()
    {
        $claseUno = new ClaseUno();
        $closure = function(ClaseUno $claseUno) {
            return $claseUno->privadaDos;
        };

        $bind = Closure::bind($closure, null, $claseUno);
        return $bind($claseUno);
    }
}
$claseDos = new ClaseDos();
echo $claseDos->obtenerPropiedadUno() . "\n";
echo $claseDos->obtenerPropiedadDos() . "\n";
```

_bind()_ es una versión estática de _bindTo()_.

```
public static Closure Closure::bind (Closure $closure, object $newthis [, mixed $newscope = "static" ])
```

**Ejemplo**:

```
class ClaseA {
    private static $soyUno = "Valor Uno";
    private $soyDos = "Valor Dos";
}
$closureUno = static function() {
    return ClaseA::$soyUno;
};
$closureDos = function() {
    return $this->soyDos;
};
$primerBind = Closure::bind($closureUno, null, 'ClaseA');
$segundoBind = Closure::bind($closureDos, new ClaseA(), 'ClaseA');
echo $primerBind(), "\n"; // Devuelve: Valor Uno
echo $segundoBind(), "\n"; // Devuelve: Valor Dos
```