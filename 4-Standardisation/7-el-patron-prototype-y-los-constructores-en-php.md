**Indice de contenido**

1.  Introducción
2.  Constructor Overloading
3.  Constructor Injection
4.  Dynamic Class Instantiation
5.  Prototype Pattern

### 1\. Introducción

La función de los constructores en las clases va más allá del código que inicia una clase cuando se instancia. La forma en que se establece un constructor puede afectar a la **usabilidad** y a la **extensibilidad** de una aplicación.

El método mágico ___construct()_ es un método de instanciamiento especial (no es marcado como estático):

```
class Pin
{
    public function __construct(){}
}
$objeto = new Pin();
```

En este ejemplo **PHP** creará un nuevo objeto, ejecutará el método ___construct()_ y asignará el objeto a la variable _$objeto_. Pero es importante saber que antes de _new Pin()_, el objeto no existía. Este hecho es el que lo hace diferente a otro tipo de métodos de instanciamiento. El método ___construct_ queda excluído de las reglas aplicables a otros métodos de instancia dentro del [principio SOLID Liskov substitution principle](http://diego.com.es/solid-principios-del-diseno-orientado-a-objetos-en-php#LiskovSubstitutionPrinciple) (LSP).

```
class Pin
{
    public function __construct(){}
}

class Pan extends Pin
{
    public function __construct(ArrayObject $arrayObj, $numero = 0)
    {
        // Hacer algo con $arrayObj y $numero
    }
}

class Pun extends Pan {
    public function __construct(Pan $pan)
    {
        // Este es el patrón proxy
    }
}
```

Lo anterior no produce ningún **warning**, pero si cambiáramos los métodos ___construct_ por cualquier otra cosa, se produciría un warning **E_STRICT**. El LSP hace referencia a subclases de un objeto particular, pero antes del método ___construct_ no existe ninguno todavía. Estas reglas no pueden aplicarse a algo que todavía no existe.

Lo que se saca de lo anterior es que la mejor práctica es que cada objeto concreto tenga un constructor con sentencias que representen bien cómo se debería instanciar ese objeto en particular. En algunos casos en los que hay herencia, heredar el constructor del padre es aceptable y útil. Además, es recomendable que cuando crees una subclase de un tipo, que el nuevo tipo deba, cuando sea apropiado, tener su propio constructor que de sentido al nuevo subtipo.

Si queremos utilizar programación orientada a objetos de una forma [SOLID](http://diego.com.es/solid-principios-del-diseno-orientado-a-objetos-en-php) se ha de evitar marcar a los constructores como _final_, hacerlos _abstract_ o ponerlos en _interfaces_.

### 2. Constructor Overloading

PHP no soporta el overloading en métodos, lo que también se aplica a los constructores. Una clase de un tipo específico puede tener sólo un constructor, por lo que los desarrolladores a veces emplean varias sentencias para acomodar múltiples posibilidades, lo que es correcto cuando se hace de forma apropiada.

```
class Db
{
    /*
     * @var string|array|DriverInterface $driver
     */
    public function __construct($driver)
    {
        if (is_string($driver)){
            $driver = $this->createDriverFromString($driver);
        } elseif (is_array($driver)){
            $driver = $this->createDriverFromArray($driver);
        }
        if (!$driver instanceof DriverInterface){
            throw new Exception();
        }
    }
}
```

Una forma apropiada suele ser minimalista pero que permita definir bien las posibilidades. El ejemplo anterior soporta tres tipos de argumentos para el constructor:

```
__construct(/* string */ $driver);
__construct(/* array */ $driver);
__construct(DriverInterface $driver);

```

El argumento es sólo uno y no cambia, pero representa los tres tipos y se puede describir en el **DocBlock** como se ha hecho en el ejemplo.

### 3. Constructor Injection

En PHP está generalmente aceptado que una buena práctica es **inyectar dependencias**, aunque la forma de inyectarlas depende de cada desarrollador.

Las formas más simples de inyectar son mediante **interface injection**, **setter injection** y **constructor injection**. En este caso nos centramos en constructor injection.

Constructor injection es un patrón para inyectar las dependencias requeridas en el constructor. Estas dependencias son normalmente otros objetos, a menudo llamados _services_. El principal beneficio de utilizar este método es que una vez que se instancia el objeto, generalmente, está totalmente preparado, lo que sigfinica que está listo para realizar sus tareas. 

```
class UserMapper
{
}
class UserRepository
{
    public function __construct(UserMapper $userMapper)
    {
        $this->userMapper = $userMapper;
    }
}
```

El ejemplo anterior muestra claramente que antes de que se pueda usar un objeto **UserRepository**, se debe primero inyectar un **UserMapper**.

Lo ideal es desarrollar una API que soporte una **inyección de dependencias** explícita a la vez que proporcione facilidad de uso en la mayoría de casos.

```
class UserMapper implements UserMapperInterface
{
}
class UserRepository
{
    protected $userMapper;
    public function __construct(UserMapperInterface $userMapper = null)
    {
        $this->userMapper = ($userMapper) ?: new UserMapper;
    }
}
```

En este ejemplo, **UserRepository** te permite inyectar la dependencia de **UserMapper**, y también instanciará UserMapper por defecto si no se había proporcionado ya.

### 4. Dynamic Class Instantiation

El siguiente ejemplo debería utilizarse sólo en algunos casos concretos donde no se han podido emplear bien otros métodos de instanciación.

```
$obj = new $className();
if(!$obj instanceof SomeBaseType)
{
    throw new \InvalidTypeException();
}
```

Es un patrón que está mal porque asume que las sentencias del constructor están libres de cualquier parámetro que se requiera. Nunca se debería usar en objetos que tienen dependencias, o en situaciones donde es posible que un subtipo pueda tener dependencias porque elimina la posibilidad de que un subtipo pueda aplicar **constructor injection**.

Otro problema es que en lugar de manejar un objeto o una lista de objetos, ahora manejas un nombre de clase o una lista de nombres de clase además del objeto o la lista de objetos. En lugar de eso sería mejor manejar sólo los objetos.

Si sabes que este tipo de objeto en particular no va a necesitar dependencias en los subtipos, puedes utilizar este patrón de instanciamiento de forma cautelar.

### 5. Prototype Pattern

El **prototype pattern** permite crear un número ilimitado de objetos de un tipo particular, con **dependencias** y cada uno son sus propias variaciones. Es un patrón muy importante si sabes que vas a tener objetos que van a ser replicados de alguna forma y que también tienen _services_ que han de ser inyectadas.

Primero creas una instancia prototipo. Esta instancia tendrá sus dependencias inyectadas y cualquier configuración instaurada. Entonces, en lugar de utilizar _new_ de nuevo, se utilizará _clone_ en el objeto, y un nuevo objeto se creará desde el objeto prototipo original. Este nuevo objeto clonado puede entonces especificarse más e inyectarse con las variaciones que hagan a este nuevo objeto único.

Consideramos el siguiente ejemplo con una conexión a una base de datos. Queremos iterar un conjunto de datos de una base de datos, y durante la iteración, presentar cada fila como un objeto que actúa como gateway de la fila (**RowObject**). Una forma de hacer esto sería obtener el array de datos de la base de datos, y durante la iteración, crear un nuevo RowObject inyectando la conexión a la base de datos:

```
class DbAdapter
{
    public function fetchAllFromTable($table)
    {
        $arrayOfData = array();
        return $arrayOfData;
    }
}

class RowGateway
{
    public function __construct(DbAdapter $dbAdapter, $tableName, $data)
    {
        $this->dbAdapter = $dbAdapter;
        $this->tableName = $tableName;
        $this->data = $data;
    }
    /*
     * Los métodos requieren acceso al database adapter
     * para poder hacer sus tareas
     */
    public function save(){}
    public function delete(){}
    public function refresh(){}
}

class UserRepository
{
    public function __construct(DbAdapter $dbAdapter){}

    public function getUsers()
    {
        $rows = array();
        foreach ($this->dbAdapter->fetchAllFromTable('users') as $rowData){
            $rows[] = new RowGateway($dbAdapter, 'user', $rowData);
        }
        return $rows;
    }
}
```

Un objeto UserRepository se construirá con un objeto database adapter. Entonces consultará la base de datos, devolviendo un array de todas las filas que devuelve la consulta. Con cada fila de datos, creará un nuevo objeto RowObject, inyectando las dependencias, configuración y los datos de la fila.

Ahora podemos preguntarnos, ¿Qué pasa si tengo una versión especial de RowGateway que quiero emplear? Esta solución puede manejarse fácilmente utilizando una instanciación dinámica como se ha visto antes.

```
class UserRepository
{
    public function __construct(DbAdapter $dbAdapter, $rowClass = 'RowGateway'){}

    public function getUsers()
    {
        $rows = array();
        foreach ($this->dbAdapter->fetchAllFromTable('users') as $rowData){
            $rowClass = $this->rowClass;
            $row = new $rowClass($dbAdapter, 'user', $rowData);
            if(!$row instanceof RowGateway){
                throw new InvalidClassType();
            }
            $rows[] = $row;
        }
        return $rows;
    }
}
```

Esto soluciona el problema parcialmente, ya que ahora podemos usar nuestra clase especializada para la implementación **RowGateway**, pero también tiene sus limitaciones. Primero estamos asumiendo incorrectamente que las sentencias del constructor de un subtipo de RowGateway son las mismas que las del tipo base. Con está asunción limitamos la posibilidad de practicar polimorfismo en los subtipos que pudieran necesitar crear.

Si se quisiera tener un objeto RowGateway que escribiera los datos en una base de datos específica, pero resfrescase los datos desde una base de datos distinta, ¿Cómo podríamos inyectar dos DbAdapters diferentes en un objeto RowGateway para conseguir este objetivo final?

La respuesta es utilizando un **Prototype Pattern**.

```
class DbAdapter
{
    // Igual que antes
}
class RowGateway
{
    public function __construct(DbAdapter $dbAdapter, $tableName)
    {
        $this->dbAdapter = $dbAdapter;
        $this->tableName = $tableName;
    }

    public function initialize($data)
    {
        $this->data = $data;
    }
    /*
     * Los métodos requieren acceso al database adapter
     * para poder hacer sus tareas
     */
    public function save(){}
    public function delete(){}
    public function refresh(){}
}

class UserRepository
{
    public function __construct(DbAdapter $dbAdapter, RowGateway $rowGatewayPrototype = null){
        $this->dbAdapter = $dbAdapter;
        $this->rowGatewayPrototype = ($rowGatewayPrototype) ? new RowGateway($this->dbAdapter, 'user');
    }

    public function getUsers()
    {
        $rows = array();
        foreach ($this->dbAdapter->fetchAllFromTable('users') as $rowData){
            $rows = $row = clone $this->rowGatewayPrototype;
            $row->initialize($rowData);
        }
        return $rows;
    }
}
```

Usando una instancia prototipo como base de todas las futuras instancias, ahora podemos permitir la habilidad de expandir esta implementación base utilizando buenas prácticas de polimorfismo en objetos para conseguir sus objetivos. Para usarlo, ahora podemos escribir:

```
class ReadWriteRowGateway extends RowGateway
{
    public function __construct(DbApadter $readDbAdapter, DbAdapter $writeDbAdapter, $tableName)
    {
        $this->readDbAdapter = $readDbAdapter;
        parent::__construct($writeDbAdapter, $tableName);
    }

    public function refresh()
    {
        // Utiliza $this->readDbAdapter en lugar de $this->dbAdapter
        // en la implementación base de RowGateway
    }
}

// Cómo usarlo
$userRepository = new UserRepository(
    $dbAdapter,
    new ReadWriteRowGateway($readDbAdapter, $writeDbAdapter, 'user')
);
$users = $userRepository->getUsers();
$user = $users[0]; // instancia de ReadWriteRowGateway con una fila específica de datos
```