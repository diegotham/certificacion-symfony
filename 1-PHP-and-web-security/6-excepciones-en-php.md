Con la llegada de **PHP 5** se incorporó una nueva **forma orientada a objetos de manejar los errores**. Las **excepciones** se utilizan para cambiar el flujo normal de un script si ocurre un error concreto dentro de una condición. Esta condición es lo que se denomina **excepción**.

**Indice de contenido**

| | |
| -------- | -------- |
| 1. Ejemplo de excepción en PHP | 4. Excepciones múltiples |
| 2. Lanzamiento y captura de excepciones | 5. Excepciones no capturadas |
| 3. Crear un exception handler | 6. Relanzar excepciones |

### 1. Ejemplo de excepción en PHP

Vamos a ver un ejemplo sencillo con una función que calcula el área de un cuadrado:

```
$miLado = -3;
function areaCuadrado($lado){
    if ($lado < 0){
        // Lanzamos una excepción
        throw new Exception ('Debes insertar un número positivo');
    } else {
        return $lado * $lado;
    }
}
areaCuadrado($miLado);
// Devuelve: Uncaught exception 'Exception' with message 'Debes insertar un número positivo'
```

Hemos _**lanzado**_ una excepción y el código detiene su ejecución ya que se produce un **error fatal**. Podemos en cambio **_capturar_ ese error** y continuar con el script:

```
// Definimos un array con los lados de los cuadrados que queremos calcular
$misLados = array(2, -6, 4);
// Creamos un loop para calcular el área de cada cuadrado
foreach ($misLados as $lado){
    try {
        echo "El área del cuadrado es: " . areaCuadrado($lado) . "<br>";
    } catch (Exception $e) {
        echo 'Ha habido una excepción: ' . $e->getMessage() . "<br>";
    }
}
/*
Devuelve:
El área del cuadrado es: 4
Ha habido una excepción: Debes insertar un número positivo
El área del cuadrado es: 16
*/
```

Ahora en lugar de parar el script, continúa y **captura la excepción**.

### 2. Lanzamiento y captura de excepciones

Una excepción puede ser lanzada ("_thrown_") y opcionalmente capturada ("_catched_"). El código debe estar dentro de un bloque _**try**_ para capturar las **excepciones**. Cada bloque **try** debe tener al menos un bloque **catch** o **finally**.

*   **Catch**. Pueden usarse múltiples bloques catch para atrapar diferentes clases de excepciones. Si se **lanza una excepción** en el bloque _**try**_, el código siguiente a la declaración no se ejecutará, y **PHP** buscará un primer bloque **catch**. Si no se **captura una excepción** se emititá un **error fatal** "Uncaught Exception..." (a no ser que se haya definido un manejador con _[set_exception_handler()](#ExcepcionesNoCapturadas)_).
*   **Finally**. Puede especificarse después o en lugar de bloques _**catch**_. El código dentro de un bloque _**finally**_ siempre se ejecutará **después** de los bloques _**try**_ y _**catch**_, independientemente de si se ha lanzado una excepción, y **antes** de que se continúe con el flujo normal del script.

El objeto lanzado debe ser siempre una instancia de la **clase Exception** o una **subclase de Exception** (sino PHP emite error fatal).

Lo **que ocurre cuando se dispara una excepción** es lo siguiente:

*   El **estado actual del código** se guarda.
*   La **ejecución del código cambia** a una función _**exception handler**_ predefinida.
*   **Dependiendo de la situación**, el _**handler**_ podría entonces resumir la ejecución desde el estado de código guardado, terminar la ejecución del script o continuar con el script desde una parte distinta del código.

Cuando se lanza una excepción desde una función o método de clase, va hasta la función o método que lo llamó. Continúa así hasta que alcanza el primer nivel o la **excepción es capturada**. Si llega hasta el primer nivel sin ser capturada, se produce un **error fatal**. Por ejemplo, tenemos una función que lanza una excepción, llamamos a ésta desde otra y finalmente llamamos a esta segunda desde el script principal:

```
function unaFuncion(){
    throw new Exception('Mensaje desde unaFuncion().');
}
function otraFuncion(){
    unaFuncion();
}
try {
    otraFuncion();
} catch (Exception $e){
    echo 'Excepción capturada: '.$e->getMessage()."<br>";
}
// Devuelve: Excepción capturada: Mensaje desde unaFuncion(). 
```

Cuando hemos llamado a otraFuncion(), intentamos calcular **cualquier excepción**. Aunque _otraFuncion()_ no lance una de forma directa, sí lo hace _unaFuncion()_. Ocurriría lo mismo si se añadiera otro nivel, y _unaFuncion()_ llamara a otra que lanzara una excepción. Este funcionamiento puede producir que no se sepa de donde viene la excepción exactamente, por eso existe la función [Exception::getTraceAsString](http://php.net/manual/es/exception.gettraceasstring.php), que devuelve la pila de una excepción como una cadena de caracteres.

```
// Clase Exception propia
class MyCustomException extends Exception {}
// Función cualquiera
function hacerAlgo() {
    try {
        // Lanzamos excepción InvalidArgumentException
        throw new InvalidArgumentException("Lo estás haciendo mal", 112);
    } catch(Exception $e) {
        // Lanzamos excepción propia
        throw new MyCustomException("Algo ha ocurrido", 911, $e);
    }
}
try {
    hacerAlgo();
    } catch(Exception $e) {
    do {
        printf("%s:%d %s (%d) [%s]\n", $e->getFile(), $e->getLine(), $e->getMessage(),
            $e->getCode(), get_class($e));
    } while($e = $e->getPrevious());
}
```

En el ejemplo anterior, _hacerAlgo()_ lanza una primera excepción **InvalidArgumentException** desde un try, que es capturada. La propia captura lanza otra excepción, esta vez de una clase propia **MyCustomException**. Ahora sí es capturada desde el try en _hacerAlgo()_, y se imprimen los datos de esta última excepción MyCustomException: getFile(), getLine(), getMessage(), etc. El while devuelve true porque hay un **Exception error** anterior, el InvalidArgumentException, por lo que continúa el loop e imprime los datos de éste. Finaliza porque no ha habido más Exception errors.

Para asegurar la regla de que cada _throw_ debe tener un _catch_, lo ideal es **crear un exception handler**.

### 3. Crear un exception handler

Para crear un _**exception handler**_ se crea una clase especial con funciones que pueden ser llamadas cuando una excepción ocurre en PHP. La clase debe ser una **extensión de Exception**. La **clase Exception** personalizada hereda las propiedades de **Exception** y se pueden añadir funciones adicionales. 

```
class customException extends Exception {
    public function errorMessage() {
        // Mensaje de error
        $errorMsg = 'Error en la línea '
        .$this->getLine().' en el archivo '
        .$this->getFile() .': <b>'
        .$this->getMessage().
        '</b> no es una dirección de email válida';
        return $errorMsg;
    }
}
```

Probamos la nueva clase **customException**:

```
// Ponemos un email no válido para forzar la excepción
$email = "ejemplo@ejemplo/.com";
// Iniciamos el bloque try
try {
    // Comprobar si el email es válido
    if(filter_var($email, FILTER_VALIDATE_EMAIL) === FALSE) {
        // Lanza una excepción si el email no es válido
        throw new customException($email);
    }
}
// Iniciamos el bloque catch
catch (customException $e) {
    // Muestra el mensaje que hemos customizado en customException:
    echo $e->errorMessage();
}
```

La clase **customException** es heredada de **Exception** solo que le hemos añadido la función _errorMessage()_. Al ser una **clase heredada de Exception**, hemos podido utilizar las funciones _getLine()_, _getFile()_ y _getMessage()_. 

### 4. Excepciones múltiples

En un script se pueden usar **excepciones múltiples para comprobar diferentes condiciones**. Se pueden utilizar [estructuras de control](http://diego.com.es/estructuras-de-control-en-php) como if/else o switch o anidar diferentes excepciones. Cada una de estas excepciones pueden usar diferentes clases **Exception** y devolver errores distintos:

```
$email = "ejemplo@ejemplo.com";
try {
    // Comprobar si el email es válido
    if(filter_var($email, FILTER_VALIDATE_EMAIL) === FALSE) {
        // Lanza una excepción si el email no es válido
        throw new customException($email);
    }
    // Comprueba la palabra ejemplo en la dirección email
    if(strpos($email, "ejemplo") !== FALSE) {
        throw new Exception("$email es un email de ejemplo");
    }
}
catch (customException $e) {
    echo $e->errorMessage();
}
catch(Exception $e) {
    echo $e->getMessage();
}
```

### 5. Excepciones no capturadas

En ocasiones no se necesita crear excepciones en todo el código del script atrapándolo todo en bloques try/catch. Sin embargo, las **excepciones no capturadas** (_uncaught exceptions_) muestran un error detallado al usuario, lo cual no es nada bueno en un **entorno de producción**.

Hay una forma de **centralizar el manejo de todas las excepciones no capturadas** de forma que puedas controlar su salida desde un sólo sitio. Para ello se utiliza [set_exception_handler()](http://php.net/manual/en/function.set-exception-handler.php), que establece una **función de gestión de excepciones definida por el usuario**:

```
set_exception_handler('exceptionHandler');
function exceptionHandler($e){
    // Mensaje público
    echo "Ha habido un error";
    // Mensaje semi-escondido
    echo "<!--Excepción sin capturar: " . $e->getMessage() . "--><br>";
}
throw new Exception('Hola');
throw new Exception('Que tal');
// Devuelve: Ha habido un error
// y como comentario HTML: <!--Excepción sin capturar: Hola-->
```

El script aborta la ejecución con la primera excepción, por lo que no ejecuta la segunda. Este es un comportamiento esperado en las **excepciones no capturadas**. Si prefieres que el script continúe después de una excepción, es necesario usar un bloque try/catch.

### 6. Relanzar excepciones

A veces cuando se lanza una excepción se prefiere manejarla de forma diferente a como se hace por defecto. Para ello se puede lanzar una excepción una segunda vez dentro de un bloque _**catch**_. 

Una de las situaciones en las que se hace esto es cuando se quiere **esconder errores del sistema a los usuarios**, y mostrarles un mensaje más amigable:

```
class customException extends Exception {
    public function errorMessage() {
        // Mensaje de error
        $errorMsg = $this->getMessage().' no es una dirección de email válida.';
        return $errorMsg;
    }
}
$email = "ejemplo@ejemplo.com";
try {
    try {
        // Comprobar la palabra ejemplo en el email:
        if(strpos($email, "ejemplo") !== FALSE) {
            // Lanzar una excepción si el email no es válido:
            throw new Exception($email);
        }
    }
    catch(Exception $e) {
        // Relanzar excepción:
        throw new customException($email);
    }
}
catch (customException $e) {
    // Mostrar mensaje customizado:
    echo $e->errorMessage();
}
```

Si el email contiene la palabra "ejemplo", la excepción se relanza. Si una **excepción** no es capturada en su bloque _**try**_ actual, buscará por un bloque _**catch**_ en niveles más altos.