Los **traits** son un **mecanismo de reutilización de código** en lenguajes que tienen **herencia simple**, como **PHP**. El objetivo de los traits es reducir las limitaciones de la herencia simple permitiendo reutilizar métodos en varias clases independientes y de distintas jerarquías. Un trait es similar a una clase, pero su objetivo es agrupar funcionalidades específicas. Un trait, al igual que las [clases abstractas](http://diego.com.es/clases-abstractas-en-php), no se puede instanciar, simplemente facilita comportamientos a las clases sin necesidad de usar la herencia.

**Indice de contenido**

| | |
| -------- | -------- |
| 1. Ejemplo de situación con traits | 4. Conflictos entre métodos de traits |
| 2. Usar múltiples traits | 5. Usar Reflection con traits |
| 3. Orden de prededencia entre traits y clases | 6. Otras funcionalidades de los traits |

### 1. Ejemplo de situación con traits

Supongamos el siguiente ejemplo:

```
// Lector DB
class DbReader extends Mysqli{}
// Lector de archivos
class FileReader extends SplFileObject{}
```

Si por ejemplo queremos aplicar una misma funcionalidad en las dos clases, tendríamos un problema ya que ambas están ya heredando a otra clase. Queremos que las dos clases sean _singletons_ (que sólo puedan instanciarse una vez). En lugar de tener que introducir esta funcionalidad en ambas, se puede **crear un trait**:

```
trait Singleton {
    private static $instance;
    public static function getInstance(){
        if(!(self::$instance instanceof self)){
            self::$instance = new self;
        }
        return self::$instance;
    }
}
// Quitamos ahora las clases que extienden para hacer pruebas solo con los traits
class DbReader {
use Singleton;
}
class FileReader {
use Singleton;
}
```

El trait **Singleton** se implementa ahora en ambas clases y podemos utilizar el método estático _getInstance()_, que crea un objeto de la clase que está usando el trait si no se ha creado ya, y lo devuelve.

Si creamos los objetos con _getInstance()_:

```
$a = DbReader::getInstance();
$b = FileReader::getInstance();
var_dump($a);
var_dump($b);
/*
Devuelve:
object(DbReader)[1]
object(FileReader)[2]
*/
```

Vemos que $a es un objeto de **DbReader** y $b es un objeto de **FileReader**, y ambos se comportan como **singletons**. El método _getInstance()_ se ha inyectado horizontalmente en ambas clases.

El **trait** no impone ningún comportamiento en la clase, simplemente es como si copiaras los métodos del trait y los pegaras en la clase.

Un trait no puede implementar **interfaces** ni extender clases normales ni abstractas.

### 2. Usar múltiples traits

Podemos **usar varios traits en la misma clase**:

```
trait Saludar {
    function decirHola(){
        return "hola";
    }
}
trait Despedir {
    function decirAdios(){
        return "adios";
    }
}
class Comunicacion {
    use Saludar, Despedir;
}
$comunicacion = new Comunicacion;
echo $comunicacion->decirHola() . ", que tal. " . $comunicacion->decirAdios();
```

También podemos **usar traits dentro de otros traits**:

```
trait SaludoYDespedida{
    use Saludar, Despedir;
}
class Comunicacion {
    use SaludoYDespedida;
}
$comunicacion = new Comunicacion;
echo $comunicacion->decirHola() . ", que tal. " . $comunicacion->decirAdios();
```

### 3. Orden de precedencia entre traits y clases

Utilizar traits dentro de otros traits es frecuente cuando la aplicación crece y hay numerosos traits que pueden agruparse para tener que incluir menos en una clase.

Eso hace que pueda haber métodos con el mismo nombre en diferentes traits o en la misma clase, por lo que tiene que haber un orden. Existe un **orden de precedencia** de los métodos disponibles en una **clase** respecto a los de los **traits**:

1.  Métodos de un trait sobreescriben métodos heredados de una clase padre
2.  Métodos definidos en la clase actual sobreescriben a los métodos de un trait

Vamos a verlo en un ejemplo con un trait y dos clases:

*   Trait Comunicacion:

```
trait Comunicacion {
    function decirHola(){
        return "Hola";
    }
    function decirQueTal(){
        return "¿Qué tal? Soy un trait";
    }
    function decirHolaYQuetal(){
        return $this->decirHola() .  " " . $this->decirQueTal();
    }
    function preguntarEstado(){
        return $this->decirHola() . " " . parent::decirQueTal();
    }
    function decirBien(){
        return "Bien, desde el Trait Comunicación";
    }
}

```

*   Clases Estado y Comunicar:

```
class Estado {
    function decirQueTal(){
        return "¿Qué tal? Soy Estado";
    }
    function decirBien(){
        return "Bien, desde la clase Estado";
    }
}
class Comunicar extends Estado {
    use Comunicacion;
    function decirQueTal() {
        return "¿Qué tal? Soy Comunicar";
    }
}
```

Creamos una instancia de comunicar y empleamos las funciones:

```
$a = new Comunicar;
echo $a->decirHolaYQuetal() . "<br>"; // Devuelve: Hola ¿Qué tal? Soy comunicar
echo $a->preguntarEstado() . "<br>"; // Devuelve: Hola ¿Qué tal? Soy Estado
echo $a->decirBien(); // Devuelve: Bien, desde el Trait Comunicación
```

Tenemos una clase **Comunicar** que extiende a **Estado**, y ambas clases tienen el método _decirQueTal()_, pero con diferentes implementaciones. También hemos incluído el trait Comunicación en la clase Comunicar.

Primero hemos llamado a dos métodos, _decirHolaYQueTal()_ y _preguntarEstado()_. Ambos llaman a decirQueTal(), que existe en ambas clases además de en el trait. _decirHolaYQueTal()_ llama al método _decirQueTal()_ que está en la misma clase **Comunicar**. _preguntarEstado()_ llama al método _decirQueTal()_ que está en la clase padre **Estado**. Esto es así porque hemos referenciado con **_parent::_**.

Por último hemos llamado a _decirBien()_, con el que se puede comprobar que tiene precedencia el método del trait al método de la clase padre.

### 4. Conflictos entre métodos de traits

Cuando se usan **múltiples traits** es posible que haya **diferentes traits que usen los mismos nombres de métodos**. PHP devolverá un **error fatal**:

```
trait Juego {
    function play(){
        echo "Jugando a un juego";
    }
}
trait Musica {
    function play(){
        echo "Escuchando música";
    }
}
class Reproductor {
    use Juego, Musica;
}
$reproductor = new Reproductor;
$reproductor->play();
// Devuelve Fatal error: Trait method play has not been applied,
// because there are collisions with other trait methods on Reproductor
```

Esto no se resuelve de forma automática, hay que elegir el método que queremos usar dentro de la clase mediante la palabra _**insteadof**_:

```
trait Juego {
    function play(){
        echo "Jugando a un juego";
    }
}
trait Musica {
    function play(){
        echo "Escuchando música";
    }
}
class Reproductor {
    use Juego, Musica {
        Musica::play insteadof Juego;
    }
}
$reproductor = new Reproductor;
$reproductor->play(); // Devuelve: Escuchando música
```

Hemos elegido la función _play()_ del trait Musica en lugar del trait Juego en la clase Reproductor.

En el ejemplo anterior se ha elegido un método sobre el otro respecto a dos traits. Hay ocasiones en las que puedes querer mantener los dos métodos, pero evitar conflictos. Se puede introducir un nuevo nombre para un método de un trait como alias. El alias no renombra el método, pero ofrece un nombre alternativo que puede usarse en la clase. Se emplea la palabra _**as**_:

```
class Reproductor {
    use Juego, Musica {
        Juego::play as playDeJuego;
        Musica::play insteadof Juego;
    }
}
$reproductor = new Reproductor;
$reproductor->play(); // Devuelve: Escuchando música
$reproductor->playDeJuego(); // Devuelve: Jugando a un juego
```

Ahora el método _playDeJuego()_ equivaldrá a _Juego::play()_.

### 5. Usar Reflection con traits

[Reflection](http://diego.com.es/reflection-en-php) es una característica de **PHP** que nos permite analizar la estructura interna de **interfaces**, **clases** y **métodos** y poder manipularlos. Existen cuatro métodos que podemos utilizar con Reflection para los traits:

*   _**ReflectionClass::getTraits()**_. Devuelve un array con todos los traits disponibles en una clase
*   _**ReflectionClass::getTraitNames()**_. Devuelve un array con los nombres de los traits en una clase.
*   _**ReflectionClass::isTrait()**_ comprueba si algo es un trait o no.
*   _**ReflectionClass::getTraitAliases()**_ devuelve un trait con los alias como _keys_ y sus nombres originales como _values_.

### 6. Otras funcionalidades de los traits

Los traits permiten acceder a propiedades o métodos _**private**_ o **_protected_**:

```
trait Mensaje {
    public function alerta(){
        echo $this->mensaje;
    }
}
class Mensajero {
    use Mensaje;
    private $mensaje = "Esto es un mensaje";
}
$mensajero = new Mensajero();
$mensajero->alerta(); // Devuelve: Esto es un mensaje
```

Los **traits** se inyectan en la clase de forma que es como si sus métodos formaran parte de ella, por lo que **cualquier método o propiedad private o protected podrá ser accedido desde un trait**.

También podemos tener **métodos abstractos dentro de traits para forzar a las clases a implementarlos**:

```
trait Mensaje {
    private $mensaje;

    public function alerta(){
        $this->definir();
        echo $this->mensaje;
    }
    abstract function definir();
}
class Mensajero {
    use Mensaje;
    function definir(){
        $this->mensaje = "Esto es un mensaje";
    }
}
$mensajero = new Mensajero();
$mensajero->alerta(); // Devuelve: Esto es un mensaje
```

Si no se define _definir()_ en **Mensajero**, dará un **error faltal**.

Traits también pueden incluir un constructor ___construct()_ que ha de ser declarado público, pero hay que tener cuidado con posibles colisiones.
