**Indice de contenido**

| | |
| -------- | -------- |
| 1\. Introducción | 8. IteratorAggregate |
| 2. ¿Por qué utilizar iteradores SPL? | 9. AppendIterator |
| 3. ArrayIterator | 10. FilterIterator |
| 4. DirectoryIterator | 11. LimitIterator |
| 5. ArrayObject | 12. NoRewindIterator |
| 6. ArrayAccess | 13. SplFileObject |
| 7. Iterator | 14. SimpleXMLIterator |

### 1. Introducción

La **biblioteca SPL**, **Standard PHP Library**, es un conjunto de **interfaces y clases para PHP**. Fue diseñada para recorrer estructuras agregadas: arrays, resultados de bases de datos, árboles XML, listados de directorios o cualquier otro tipo de listado.

SPL trabaja con **iteradores**. La **iteración** es el proceso de recorrer una lista de elementos. Un **iterador** es simplemente un objeto que recorre una lista. La biblioteca viene con un gran listado de clases, para verlas todas, puedes emplear la función _spl_classes()_:

```
foreach(spl_classes() as $key => $value)
{
    echo $key.' -> '.$value.'<br>';
}
```

### 2. ¿Por qué utilizar Iteradores SPL?

Los **objetos iteradores** realizan la misma función que cuando se itera con arrays, pero a diferencia de éstos, pueden albegar una gran cantidad de datos y organizarlos más eficientemente.

El **loop foreach** hace una copia de cualquier array que se le pase, por lo que si son muchos los datos que se tienen que iterar, hacer una copia de cada array cada vez que se usen en un foreach puede resultar muy costoso. Los **iteradores SPL encapsulan las listas de datos** y muestran un elemento cada vez haciéndolo mucho más eficiente. 

Los iteradores también facilitan el **lazy loading**, es decir, devuelven los datos sólo cuando se necesita. También se pueden manipular los datos antes de devolverlos al usuario. 

**Usar o no usar iteradores SPL** es una decisión que depende del **proyecto** en el que se esté trabajando. Normalmente cuando se trabaja con una gran cantidad de datos es más conveniente usarlos.

### 3. ArrayIterator

El constructor de la clase **ArrayIterator** recibe un array como parámetro y proporciona métodos con los que iterar mediante el objeto creado:

```
$animales = ["perro", "gato", "liebre", "toro", "conejo"];
// Se crea un nuevo ArrayIterator y se pasa el array
$iterador = new ArrayIterator($animales);
// Se hace loop con el objeto
foreach ($iterador as $key => $value) {
    echo $key . ":  " . $value . "<br>";
}
/*
0:  perro
1:  gato
2:  liebre
3:  toro
4:  conejo
*/
```

Normalmente se utiliza **ArrayObject**, una **clase que permite trabajar con objetos como si fueran arrays**. Esta clase crea directamente un **ArrayIterator** cuando se usa un _loop foreach_ o cuando se llama al método **ArrayIterator::getIterator()**. 

Conviene apuntar que tanto ArrayObject como ArrayIterator son **objetos**, y no se pueden usar funciones de arrays en ellos.

ArrayIterator está limitado a arrays unidimensionales, de un sólo nivel. Para poder utilizarlo en arrays multidimensionales se puede utilizar **RecursiveArrayIterator**. 

Una forma de **mostrar los elementos anidados en un array con un foreach** es la siguiente:

```
$animales = [
    "perro" => ["Sparky"],
    "gato" => ["Bear"],
    "liebre" => ["Bunny"],
    "toro" => ["Hunk"],
    "conejo" => "Juliet"
];
// Loop a través del array
foreach ($animales as $key => $value) {
    // Comprobar si tiene subnivel
    if (is_array($value)) {
        foreach ($value as $k => $v) {
            echo $k . ": " . $v . "<br>";
        }
    }
    else {
        echo $key . ": " . $value . "<br>";
    }
}
/*
0: Sparky
0: Bear
0: Bunny
0: Hunk
conejo: Juliet
*/
```

Una forma más elegante de hacerlo es utilizando **RecursiveArrayIterator**:

```
$animales = [
    "perro" => ["Sparky"],
    "gato" => ["Bear"],
    "liebre" => ["Bunny"],
    "toro" => ["Hunk"],
    "conejo" => "Juliet"
];
$iterador = new RecursiveArrayIterator($animales);
foreach(new RecursiveIteratorIterator($iterador) as $key => $value) {
    echo $key . ": " . $value . "<br>";
}
/*
0: Sparky
0: Bear
0: Bunny
0: Hunk
conejo: Juliet
*/
```

Es necesario crear una instancia de **RecursiveIteratorIterator** y pasarle el objeto **RecursiveArrayIterator** que se ha creado con anterioridad. 

Cuando se trabaja con arrays multidimensionales conviene usar **RecursiveArrayIterator**. **RecursiveIteratorIterator** es el **decorador** que permite iterar por los diferentes niveles, itera el array de RecursiveArrayIterator e itera también cualquier otro array que se vaya encontrando como valor. Se puede **obtener la profundidad de los niveles de un array** con **RecursiveIteratorIterator::getDepth()**. Hay que tener en cuenta que **los objetos también son iterables**, por lo que si un elemento del array es un objeto, será iterado.

### 4. DirectoryIterator

Para **recorrer directorios y sus archivos en PHP** ya existen funciones específicas como _scandir()_ o _glob()_, pero también es posible utilizar **DirectoryIterator**.

```
// Se crea un nuevo objeto DirectoryIterator
$directorio = new DirectoryIterator("/Mi/directorio");
// Se hace un loop a través del directorio
foreach ($directorio as $item) {
    echo $item . "<br>";
}
```

Con este **iterador**, al igual que con cualquiera de los **iteradores SPL**, se pueden utilizar excepciones de esta forma:

```
try {
    $directorio = new DirectoryIterator("/un/directorio/que/no/existe");
    foreach ($directorio as $item) {
        echo $item . "<br>";
    }
}
catch (Exception $e) {
    echo get_class($e) . ": " . $e->getMessage();
}
/*
UnexpectedValueException: DirectoryIterator::__construct(/un/directorio/que/no/existe):
failed to open dir: No such file or directory
 */
```

Existen métodos muy útiles para tener información básica de cualquier directorio: **DirectoryIterator::isDot()**, **DirectoryIterator::getType()** o **DirectoryIterator::getSize()**. Se puede combinar también **DirectoryIterator** con otros iteradores como **FilterIterator** o **RegexIterator** para devolver archivos con unos criterios específicos:

```
class FileExtensionFilter extends FilterIterator
{
    // Lista de extensiones permitidas
    protected $ext = ["php", "txt"];
    // Método abstract que debe ser implementado en la subclase
    public function accept() {
        return in_array($this->getExtension(), $this->ext);
    }
}
// Crear un nuevo iterador:
$directorio = new FileExtensionFilter(new DirectoryIterator("/mi/directorio"));
// Mostrar sólo los archivos php y txt
foreach ($directorio as $key){
    echo $key . "<br>";
}
```

**SPL** también proporciona **RecursiveDirectoryIterator**, que se puede usar de la misma forma que **RecursiveArrayIterator**. RecursiveDirectoryIterator puede facilitar el trabajo de iterar directorios recursivamente, además de que queda un código mucho más limpio. La única pega es que no muestra directorios vacíos, sólo los que tienen archivos:

```
// Se crea un nuevo objeto RecursiveDirectoryIterator
$iterador = new RecursiveDirectoryIterator("/mi/directorio/");
// Se crea instancia de RecursiveIteratorIterator
foreach (new RecursiveIteratorIterator($iterador) as $item) {
    echo $item . "<br>";
}
```

### 5. ArrayObject

**ArrayObject** permite el **recorrido externo de arrays** y **crear instancias de ArrayIterator**. Podemos ver los métodos de la clase de esta forma:

```
foreach(get_class_methods(new ArrayObject()) as $key=>$method)
{
    echo $key.' -> '.$method.'<br>';
}
```

Los métodos de los que dispone ArrayObject son los siguientes:

* 0 -> __construct
* 1 -> offsetExists
* 2 -> offsetGet
* 3 -> offsetSet
* 4 -> offsetUnset
* 5 -> append
* 6 -> getArrayCopy
* 7 -> count
* 8 -> getFlags
* 9 -> setFlags
* 10 -> asort
* 11 -> ksort
* 12 -> uasort
* 13 -> uksort
* 14 -> natsort
* 15 -> natcasesort
* 16 -> unserialize
* 17 -> serialize
* 18 -> getIterator
* 19 -> exchangeArray
* 20 -> setIteratorClass
* 21 -> getIteratorClass

**ArrayIterator** tiene los 15 primeros métodos iguales a **ArrayObject**, se diferencian en los seis últimos. **ArrayIterator** tiene: rewind(), current(), key(), next(), valid() y seek(), lo que le permite iterar por sí solo. **ArrayObject** necesita crear una instancia de **ArrayIterator** para iterar, y lo hace mediante _getIterator()_.

Veamos un ejemplo:

```
$animales = array('perro', 'gato', 'conejo', 'cerdo', 'caballo');
// Se crea el objeto array:
$objeto = new ArrayObject($animales);
// Podemos añadir otro elemento al array
$objeto->append('canguro');
// También podemos ordenarlo como queramos, por ejemplo, alfabéticamente:
$objeto->natcasesort();
// Se itera a través del array:
for($iterador = $objeto->getIterator();
    // Comprobar si es válido
    $iterador->valid();
    // Moverse al siguiente elemento
    $iterador->next())
{
    // Mostrar el key y value actuales
    echo $iterador->key() . ' => ' . $iterador->current() . '<br />';
}
```

En el ejemplo anterior se ha recorrido externamente el objeto del array con _getIterator()_. Podríamos haber usado un _**foreach**_ y el método _getInstance()_ se hubiera llamado automáticamente. Los métodos _key()_ y _current()_ pertenecen a la instancia de **ArrayIterator.**

**ArrayObject::getIterator** devuelve una instancia de **ArrayIterator** desde la instancia original de **ArrayObject**.

Al ordenar los resultados con _natcasesort()_ podemos ver que el resultado ha ordenado los valores alfabéticamente, pero los keys se han mantenido tal cual:

```
4 => caballo
5 => canguro
3 => cerdo
2 => conejo
1 => gato
0 => perro

```

Podemos emplear la función _count()_ para contar el número de items:

```
$animales = array('perro', 'gato', 'conejo', 'serpiente', 'caballo');
$objeto = new ArrayObject($animales);
echo $objeto->count(); // Devuelve 5
```

También es posible **remover un ítem del array** anterior empleando _offsetUnset($elemento)_, donde _$elemento_ es el ítem que queremos eliminar. Vamos a eliminar el único que no es mamífero:

```
$animales = array('perro', 'gato', 'conejo', 'serpiente', 'caballo');
$objeto = new ArrayObject($animales);

$objeto->offsetUnset(3);

for($iterador = $objeto->getIterator(); $iterador->valid(); $iterador->next()){
    echo $iterador->key() . " -> " . $iterador->current() . "<br>";
}
/*
0 -> perro
1 -> gato
2 -> conejo
4 -> caballo
 */
```

Vemos que el array no se reindexa, desaparece serpiente pero los demás _keys_ mantienen su número. Para comprobar que existe un offset se emplea _offsetExists()_:

```
$animales = array('perro', 'gato', 'conejo', 'serpiente', 'caballo');
$objeto = new ArrayObject($animales);

if ($objeto->offsetExists(3)) echo "Offset existe"; // Devuelve Offset existe
```

También podemos **cambiar el valor de cualquier elemento**, con _offsetSet()_. Vamos a insertar un nuevo animal mamífero en el lugar que ha quedado vacío:

```
$animales = array('perro', 'gato', 'conejo', 'serpiente', 'caballo');

try {
    $objeto = new ArrayObject($animales);

    $objeto->offsetSet(3, "oso");

    for($iterador = $objeto->getIterator(); $iterador->valid(); $iterador->next())
    {
        echo $iterador->key() . ' -> ' . $iterador->current() . '<br>';
    }
}
catch (Exception $e)
{
    echo $e->getMessage();
}
```

También existe _offsetGet()_ para **obtener el valor de un elemento del array**:

```
$animales = array('perro', 'gato', 'conejo', 'serpiente', 'caballo');

try {
    $objeto = new ArrayObject($animales);

    echo $objeto->offsetGet(4); // Devuelve caballo
}
catch (Exception $e)
{
    echo $e->getMessage();
}

```

Si queremos **copiar un array para editar o hacer comparaciones**, podemos emplear _getArrayCopy()_:

```
$animales = array('perro', 'gato', 'conejo', 'serpiente', 'caballo');
$objeto = new ArrayObject($animales);
$copiaArray = $objeto->getArrayCopy();
```

Hay que tener en cuenta que lo anterior es una **copia del array**, no es una copia del objeto, por lo que dará **error** si queremos hacer algo como _**$copiaArray->getIterator()**_. Para ello deberemos crear de nuevo un objeto con la copia:

```
$animales = array('perro', 'gato', 'conejo', 'serpiente', 'caballo');
$objeto = new ArrayObject($animales);
$copiaObjeto = new ArrayObject($objeto->getArrayCopy());
```

La clase también viene con algunas _**flags**_ o banderas: **STD_PROP_LIST** y **ARRAY_AS_PROPS**, que permiten configurar el registro y la salida de los datos en la clase **ArrayObject**.

STD_PROP_LIST nos dice como leer, y ARRAY_AS_PROPS nos dice como debemos escribir los elementos creados en el array de la clase **ArrayObject**. Si guardamos elementos a través de la notación normal de **array[]**, siempre funcionará de la misma forma, independientemente de los flags:

```
$a = new ArrayObject();
$a['key'] = 'Valor del elemento array';
$a->propiedad = 'Valor de la propiedad de objeto';
var_dump($a);
echo "Foreach:" . "<br>";
foreach ($a as $wa){
    echo $wa . "<br>";
}
/*
object(ArrayObject)[1]
  public 'propiedad' => string 'Valor de la propiedad de objeto' (length=31)
  private 'storage' =>
    array (size=1)
      'key' => string 'Valor del elemento array' (length=24)
Foreach:
Valor del elemento array
*/
```

Cuando configuramos **ARRAY_AS_PROPS** significa que cuando guardas **propiedades** no se guardarán en el objeto como se haría con objetos normales, se guardarán como **elementos internos**. Esto hace que aparezcan también en un _**foreach()**_. Si no se configura esta bandera, se guardará la propiedad en el objeto del array (como en el ejemplo anterior). Veamos el ejemplo con ARRAY_AS_PROPS:

```
$b = new ArrayObject(array(), ArrayObject::ARRAY_AS_PROPS);
$b['key'] = 'Valor del elemento array';
$b->propiedad = 'Valor de la propiedad de objeto';
var_dump($b);
echo "Foreach:" . "<br>";
foreach ($b as $we){
    echo $we . "<br>";
}
/*
object(ArrayObject)[1]
  private 'storage' =>
    array (size=2)
      'key' => string 'Valor del elemento array' (length=24)
      'propiedad' => string 'Valor de la propiedad de objeto' (length=31)
Foreach:
Valor del elemento array
Valor de la propiedad de objeto
*/
```

La bandera **STD_PROP_LIST** decide qué devolver bajo ciertas condiciones, sobre todo con _**var_dump()**_, aunque existen otros casos. Si se configura **ArrayObject** con STD_PROP_LIST, devolverá las propiedades que se han establecido en **ArrayObject** o **ArrayIterator**. Si no se han establecido, var_dump() devolverá una lista de los elementos internos que se hayan guardado. 

```
$c = new ArrayObject(array(), ArrayObject::STD_PROP_LIST);
$c['key'] = 'Valor del elemento array';
$c->propiedad = 'Valor de la propiedad de objeto';

var_dump($c);
echo "Foreach:" . "<br>";
foreach ($c as $wi){
    echo $wi . "<br>";
}
/*
object(ArrayObject)[1]
  public 'propiedad' => string 'Valor de la propiedad de objeto' (length=31)
  private 'storage' =>
    array (size=1)
      'key' => string 'Valor del elemento array' (length=24)
Foreach:
Valor del elemento array
*/
```

### 6. ArrayAccess

**ArrayAccess** es una interface de PHP que permite dictar cómo se comportará PHP cuando **un objeto tenga una sintaxis de array** (corchetes después del objeto). ArrayAccess sólo tiene 4 funciones a implementar: _offsetExists_, _offsetGet_, _offsetSet_ y _offsetUnset_. Para implementar la interface sólo hay que declarar estos métodos en la clase:

```
class Animal implements ArrayAccess {
    protected $animales;
    public $lugar;
    public function offsetExists($index){
        return isset($this->animales[$index]);
    }
    public function offsetGet($index){
        if($this->offsetExists($index)) {
            return $this->animales[$index];
        }
        return false;
    }
    public function offsetSet($key, $value){
        if($key) {
            $this->animales[$key] = $value;
        } else {
            $this->animales[] = $value;
        }
        return true;
    }
    public function offsetUnset($index){
        unset($this->animales[$index]);
        return true;
    }
    public function getAnimales() {
        return $this->animales;
    }
}

```

Podemos usar las propiedades y métodos de forma normal como en cualquier objeto, pero además podemos usar la sintaxis de array para manipular _$animales_:

```
$animal = new Animal;
$animal->lugar = "Asia y Oceania";
$animal[] = "Oso panda";
$animal[] = "Koala";
$animales = $animal->getAnimales();
foreach ($animales as $anim){
    echo $anim . "<br>";
}
/*
Devuelve:
Oso panda
Koala
*/
```

**ArrayAccess** es extendida por [ArrayObject](http://php.net/manual/es/class.arrayobject.php), por eso cuando se crea un objeto ArrayObject está disponible esta funcionalidad, además de [IteratorAggregate](#IteratorAggregate) (que veremos después), **Countable** y **Serializable**.

### 7. Iterator

Hay ocasiones en las que los iteradores no cubren unas necesidades específicas y es necesario crear un **iterador customizado**. **SPL** proporciona **interfaces** con las que poder hacerlo.

Para que un **objeto sea transitable en un loop foreach** debe ser una instancia de **Traversable**. No es posible implementar esta interface directamente, por lo que se utilizan las interfaces _**Iterator**_ o _**IteratorAggregate**_.

_**Iterator**_ permite crear objetos para ser iterados directamente o para crear iteradores externos. Especifica 5 métodos que hay que implementar: _rewind()_, _current()_, _key()_, _next()_ y _valid()_. Vamos a construir por partes la clase Animal:

```
class Animal implements Iterator {
    // Puntero que indica la posición actual
    protected $position = 0;
    // Array de animales
    protected $animales = [
        "Perro",
        "Gato",
        "Oso",
        "León"
    ];
```

El método _rewind()_ devuelve el puntero al principio para reiniciar la iteración:

```
public function rewind(){
    echo "Reiniciando <br>";
    $this->position = 0;
}
```

El método _current()_ devuelve el valor actual del elemento en la posición actual:

```
public function current(){
    echo "Posición actual <br>";
    return $this->animales[$this->position];
}
```

El método _key()_ devuelve el valor actual del puntero:

```
public function key(){
    echo "Key <br>";
    return $this->position;
}
```

El método _next()_ mueve el puntero a la próxima posición:

```
public function next(){
    echo "Next <br>";
    ++$this->position;
}
```

Y finalmente el método _valid()_ devuelve un booleano indicando si hay datos en la posición actual:

```
    public function valid(){
        echo "Valido <br>";
        return isset($this->animales[$this->position]);
    }
}
```

Instanciamos la clase y hacemos un foreach:

```
$animales = new Animal;
foreach ($animales as $key=>$value){
    echo $key . ":" . $value . "<br>";
}
```

Cuando la iteración comienza, PHP llama a _rewind()_ para llevar el puntero al inicio. Entonces comprueba si hay algún **dato válido** en ese punto llamando a _valid()_. Si es **true**, llama a _current()_ para mostrar el **valor** en ese punto de la iteración. Como estamos usando el $key también en el foreach, PHP llama a _key()_ para obtener el valor del **key** actual. PHP llamará entonces a _next()_ para **avanzar el puntero** y a _valid()_ para comprobar si es **válido** este nuevo elemento. El ciclo continúa hasta que _valid()_ devuelve **false**.

Iterator posibilita el poder implementar iteradores muy potentes. Para el ejemplo anterior que es un simple array, sería mejor emplear **IteratorAggregate**.

### 8. IteratorAggregate

**IteratorAggregate** requiere implementar sólo un método, _getIterator()_. Este método devuelve un iterador externo que se usará para la **iteración**. Si escribimos el ejemplo anterior empleando _**IteratorAggregate**_:

```
class Animal implements IteratorAggregate {
    protected $animales = [
        "Perro",
        "Gato",
        "Oso",
        "León"
    ];
    // Devuelve un Iterator de los datos
    public function getIterator(){
        echo "getIterator <br>";
        return new ArrayIterator($this->animales);
    }
}
$animales = new Animal();
foreach ($animales as $key=>$value){
    echo $key . ":" . $value . "<br>";
}
```

Al principio de la iteración, PHP llama a _getIterator()_, que devuelve un iterador, en este caso el iterador SPL ArrayIterator. PHP entonces hace las llamadas pertinentes en ese iterador. Básicamente dejas los datos en manos del iterador externo que hace el trabajo por tí.

**Iterator** e **IteratorAggregate** tienen la misma funcionalidad, **Iterator** permite definir el comportamiento de la iteración para ocasiones en las que se necesita mayor customización e **IteratorAggregate** es un recurso más simple y rápido.

### 9. AppendIterator

Cuando se tienen múltiples elementos sobre los que se quiere iterar de forma conjunta se emplea **AppendIterator**. El siguiente ejemplo crea dos objetos **ArrayObject** y un iterador **AppendIterator**.

```
$domesticos = array("perro", "gato", "conejo");
$salvajes = array("oso", "tiburón", "tigre");
$objetoDom = new ArrayObject($domesticos);
$objetoSal = new ArrayObject($salvajes);
$iterador = new AppendIterator();
$iterador->append($objetoDom->getIterator());
$iterador->append($objetoSal->getIterator());
foreach ($iterador as $animal){
    echo $animal . "<br>";
}

```

Con **AppendIterator** se pueden recolectar elementos desde diferentes objetos y mostrarlos de vez.

Un ejemplo algo más complejo es en un supuesto **CMS** donde cada **tipo** (enlace, artículo, etc) extiende una **clase abstracta Elemento** (ya que todos los tipos en este CMS han de ser del mismo elemento), se crean colecciones de elementos que coleccionan N elementos y nos proporcionan un iterador para iterar sobre los tipos que se puedan añadir. Vamos a crear por partes las clases:

*   Clase abstracta Element y las clases Link y Article:

```
abstract class Element {
    public $title;
}
class Link extends Element {
    public function __construct($title) {
        $this->title = $title;
    }
}
class Article extends Element {
    public function __construct($title) {
        $this->title = $title;
    }
}
```

*   Clase abstracta ElementCollection:

```
abstract class ElementCollection implements IteratorAggregate {
    private $elements;
    public function __construct() {
        $this->elements = new ArrayObject();
    }
    public function addElement(Element $module) {
        $this->elements->append($module);
    }
    public function getIterator() {
        return $this->elements->getIterator();
    }
}
```

*   Clase ArticleCollection:

```
class ArticleCollection extends ElementCollection {
    public function __construct() {
        parent::__construct();
        $this->collectArticles();
    }
    private function collectArticles() {
        // Crear artículos
        $a = new Article("Primer artículo");
        $b = new Article("Segundo artículo");
        $this->addElement($a);
        $this->addElement($b);
    }
}
```

*   Y finalmente la clase LinkCollection:

```
class LinkCollection extends ElementCollection {
    public function __construct() {
        parent::__construct();
        $this->collectLinks();
    }
    private function collectLinks() {
        // Crear enlaces
        $a = new Link("Primer enlace");
        $b = new Link("Segundo enlace");
        $this->addElement($a);
        $this->addElement($b);
    }
}
```

Ahora vamos a crear dos colecciones y utilizamos AppendIterator para usar _append()_ en las dos y unirlas. Ahora podemos iterar sobre las dos colecciones y mostrar el elemento común _$title_.

```
$article = new ArticleCollection();
$link = new LinkCollection();
$it = new AppendIterator();
$it->append($article->getIterator());
$it->append($link->getIterator());
foreach($it as $module){
    echo $module->title . "<br>";
}
/*
Devuelve:
Primer artículo
Segundo artículo
Primer enlace
Segundo enlace
*/
```

### 10\. FilterIterator

**FilterIterator** es una clase abstracta que nos permite devolver sólo los elementos que queramos, mediante el método _accept()_.

```
class Filter extends FilterIterator {
    // FilterIterator necesita un iterador como parámetro
    public function __construct(Iterator $it){
        parent::__construct($it);
    }
    // Cuando iteremos sobre el array se llamará a la siguiente función
    // para comprobar si aceptar o no la key actual
    public function accept(){
        // Sólo queremos los animales que empiecen por c:
        if(preg_match("/^c/", $this->getInnerIterator()->current())){
            return true;
        }
        return false;
    }
}
$animales = array("perro", "conejo", "pantera", "cerdo", "gato", "caballo");
$objeto = new ArrayObject($animales);
$iterador = new Filter($objeto->getIterator());
foreach ($iterador as $key=>$value){
    echo "$key : $value" . "<br>";
}
```

Nos ha devuelto sólo conejo, cerdo y caballo. FilterIterator es muy útil si se quiere comprobar el elemento antes de devolverlo. Observar que se obtienen los mismos resultados si en lugar de emplear ArrayObject se emplea ArrayIterator (de esta forma no hay que llamar a getIterator):

```
$objeto = new ArrayIterator($animales);
$iterador = new Filter($objeto);
```

### 11. LimitIterator

Cuando **tenemos muchos resultados** pero sólo queremos mostrar parte de ellos podemos crear un **LimitIterator**, una clase que controla cuantos elementos hay en el iterador y muestra el número que especifiques en el constructor:

```
$animales = array("perro", "conejo", "pantera", "cerdo", "gato", "caballo");
$objeto = new ArrayObject($animales);
/*
El LimitIterator toma un iterador como primer parámetro, después
dónde deseas empezar y finalmente cuántos deseas mostrar
*/
$iterador = new LimitIterator($objeto->getIterator(), 2, 3);
foreach($iterador as $key=>$value){
    echo "$key : $value" . "<br>";
}
/*
Devuelve:
2 : pantera
3 : cerdo
4 : gato
*/
```

Vamos a crear ahora tres páginas donde mostramos en cada una sólo un **animal**:

```
$animales = array("perro", "conejo", "pantera", "cerdo", "gato", "caballo");
$objeto = new ArrayObject($animales);
$pagina = 0;
$porPagina = 1;
for ($i = 0; $i < 5; $i++){
    echo "Pagina: $pagina <br>";
    $iterador = new LimitIterator($objeto->getIterator(), $pagina, $porPagina);
    foreach ($iterador as $key=>$value){
        echo "$key : $value <br>";
    }
    $pagina++;
}
/*
Devuelve:
Pagina: 0
0 : perro
Pagina: 1
1 : conejo
Pagina: 2
2 : pantera
Pagina: 3
3 : cerdo
Pagina: 4
4 : gato
*/
```

### 12. NoRewindIterator

**NoRewindIterator** es un iterador que no llama a _rewind()_, por lo que sólo se puede usar una vez:

```
$animales = array("perro", "conejo", "pantera");
$objeto = new ArrayObject($animales);
$iterador = new NoRewindIterator($objeto->getIterator());
foreach ($iterador as $animal){
    echo $animal . "<br>";
}
foreach ($iterador as $animal){
    echo $animal . "<br>";
}
foreach ($iterador as $animal){
    echo $animal . "<br>";
}
/*
Devuelve:
perro
conejo
pantera
*/
```

Sólo devuelve el _**array**_ una vez. Si hubiéramos hecho el foreach con _$objeto_ en lugar de con _$iterador_, el array se hubiera mostrado tres veces.

### 13. SplFileObjet

**SplFileObject** proporciona una interface orientada a objetos para manejar archivos.

Para iterar sobre un archivo se puede hacer así:

```
$lineas = file("archivo.txt");
foreach ($lineas as $num => $lin){
    echo "Linea: $num -> $lin" . "<br>";
}
```

Con **SplFileObject** podemos tratar el archivo como si fuera un objeto y usar iteradores SPL para recorrer el archivo:

```
try {
    $file = new SplFileObject("archivo.txt");
    while ($file->valid()){
        echo $file->current() . "<br>";
        $file->next();
    }
} catch (Exception $e){
    echo $e->getMessage();
}
```

[SplFileObject](http://php.net/manual/es/class.splfileobject.php) también dispone de muchas otras funciones disponibles para archivos: _getFilename_, _getPath_, _getPerms_, _getSize_, _isWritable_, _isDir_, _openFile_, etc.

### 14. SimpleXMLIterator

El iterador **SimpleXMLIterator** acepta un **documento XML** y lo recorre como cualquier iterador con otra estructura agregada. Extiende a **SimpleXMLElement**, y es un **iterador recursivo**. Si empleamos el siguiente código podremos ver las funciones de las que dispone:

```
foreach (get_class_methods('SimpleXMLIterator') as $key => $metodo){
    echo $key . '->'. $metodo . "<br>";
}
```

La **lista de funciones de SimpleXMLIterator** es la siguiente:

* 0 -> rewind
* 1 -> valid
* 2 -> current
* 3 -> key
* 4 -> next
* 5 -> hasChildren
* 6 -> getChildren
* 7 -> __construct
* 8 -> asXML
* 9 -> saveXML
* 10 -> xpath | 11 -> registerXPathNamespace
* 12 -> attributes
* 13 -> children
* 14 -> getNamespaces
* 15 -> getDocNamespaces
* 16 -> getName
* 17 -> addChild
* 18 -> addAttribure
* 19 -> __toString
* 20 -> count

Vamos a poner un ejemplo. El string XML es como sigue:

```
$xmlString = <<<XML
<?xml version = "1.0" encoding="UTF-8" standalone="yes"?>
<document>
<animal>
<categoria id="2">
<tipo>Koala</tipo>
<nombre>Bruce</nombre>
</categoria>
</animal>
<animal>
<categoria id="4">
<tipo>Gato</tipo>
<nombre>Whiskas</nombre>
</categoria>
</animal>
</document>
XML;
```

Ahora vamos a emplear SimpleXMLIterator sobre el string XML:

```
<?php
try {
    $iterador = simplexml_load_string($xmlString, 'SimpleXMLIterator');
    foreach(new RecursiveIteratorIterator($iterador, 1) as $nombre => $dato){
        echo "$nombre -- $dato <br>";
    }
} catch (Exception $e){
    echo $e->getMessage();
}
/*
Devuelve:
animal --
categoria --
tipo -- Koala
nombre -- Bruce
animal --
categoria --
tipo -- Gato
nombre -- Whiskas
*/
?>
```

**SimpleXMLIterator** es recursivo, por lo que hemos necesitado un iterador mediante **RecursiveIteratorIterator**, que itera sobre un iterador recursivo. 

Normalmente necesitaremos más control:

```
try {
    $sxi = new SimpleXMLIterator($xmlString);
    foreach($sxi as $nodo){
        foreach($nodo as $key=>$value){
            echo $value->nombre . "<br>";
        }
    }
} catch (Exception $e){
    echo $e->getMessage();
}
/*
Devuelve:
Bruce
Whiskas
*/
```

En el ejemplo anterior hemos instanciado la clase **SimpleXMLIterator**. Después hemos tenido que emplear dos loops foreach para devolver el nombre correcto de los animales.

Podemos hacerlo de otra forma:

```
try {
    $sxs = simplexml_load_string($xmlString, 'SimpleXMLIterator');
    for ($sxs->rewind(); $sxs->valid(); $sxs->next()){
        if($sxs->hasChildren()){
            foreach ($sxs->getChildren() as $el=>$val){
                echo $val->nombre . "<br>";
            }
        }
    }
} catch (Exception $e){
    echo $e->getMessage();
}
/*
Devuelve:
Bruce
Whiskas
*/
```

Pero esta forma todavía es más lenta y engorrosa. En lugar de los dos métodos anteriores se puede usar el método _**SimpleXMLIterator::xpath()**_, que permite una iteración directa sobre un árbol XML especificando el xpath al nodo. Similar a un directorio de un ordenador, el xpath del nombre de los animales es: _animal/categoria/nombre_.

Especificar el xpath de un particular nodo permite que podamos ahorrarnos muchas iteraciones del árbol XML, lo que ahorra ciclos CPU y hace scripts más rápidos. El siguiente código devolverá lo mismo que los dos anteriores, pero utilizará menos recursos:

```
try {
    $sxi = new SimpleXMLIterator($xmlString);
    $xpath = $sxi->xpath('animal/categoria/nombre');
    foreach ($xpath as $key=>$value){
        echo $value . "<br>";
    }
} catch (Exception $e){
    echo $e->getMessage();
}
/*
Devuelve:
Bruce
Whiskas
*/
```
