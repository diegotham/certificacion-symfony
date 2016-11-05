Las **interfaces** de objetos son contratos que han de cumplir las **clases** que las implementan. Contienen **métodos vacíos** que obligan a una clase a emplearlos, promoviendo así un **estándar de desarrollo**.

Si una clase implementa una _**interface**_, está obligada a usar todos los métodos de la misma (y los mismos tipos de argumentos de los métodos), de lo contrario dará un **error fatal**. Pueden emplearse **más de una interface en cada clase**, y pueden **extenderse** entre ellas mediante _extends_. Una interface puede extender una o más interfaces.

Todos los métodos declarados en una interface deben ser públicos. 

Para definir una interface se utiliza la parabra _**interface**_, y para extenderla se utiliza _**implements**_. 

```
interface Automovil {
    public function getTipo();
    public function getRuedas();
}
class Coche implements Automovil {
    public function getTipo(){
        echo "Coche";
    }
    public function getRuedas(){
        echo "4";
    }
}
class Moto implements Automovil {
    public function getTipo(){
        echo "Moto";
    }
    public function getRuedas(){
        echo "2";
    }
}
```

Las clases **Coche** y **Moto** están obligadas a definir las funciones _getTipo()_ y _getRuedas()_. Así se crea un modelo para todas las **clases** de automóviles que deben aportar o definir unos métodos mínimos. Si por ejemplo _getTipo()_ tuviera un argumento de tipo _**Array**_: _getTipo(Array $tipos)_, las clases que implementen la interface deberán importar un argumento del mismo tipo.

Las interfaces también pueden definir **constantes**, que funcionan igual que las **constantes en clases**, pero no pueden ser sobreescritas por sus descendientes, ya sean clases o interfaces.

Se utilizan cuando se tienen muchas clases que tienen en común un comportamiento, pudiendo asegurar así que ciertos métodos estén disponibles en cualquiera de los objetos que queramos crear. Son especialmente importantes para la **arquitectura de aplicaciones complejas**.