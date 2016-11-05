Si tu código interactúa con una **base de datos**, como por ejemplo que lea datos o guarde datos en ella, necesitas ajustar tus tests para que tenga esto en cuenta. Hay muchas formas de hacerlo. En un [unit test](https://diego.com.es/unit-tests-en-symfony), puedes crear un _mock_ de un repositorio y usarlo para devolver los objetos esperados. En un [functional test](https://diego.com.es/functional-tests-en-symfony), podrías necesitar preparar una base de datos de prueba con valores predefinidos para asegurar que tu test siemple tiene los mismos datos con los que trabajar.

Si quieres testear directamente las consultas, puedes leer [Cómo testear repositorios Doctrine](http://symfony.com/doc/current/cookbook/testing/doctrine.html).

#### Imitar el repositorio en un Unit test

Si quieres comprobar código que depende de un **repositorio Doctrine** aislado, necesitas imitar (_mock_) el repositorio. Normalmente inyectas el **EntityManager** en tu clase y lo utilizas para obtener el repositorio. Esto hace las cosas un poco más difíciles ya que necesitas imitar tanto el EntityManager como tu clase repositorio. 

Es posible también inyectar tu repositorio directamente registrándolo como un [factory service](http://symfony.com/doc/current/components/dependency_injection/factories.html). Es un poco más costos de hacer pero hace que testear sea más fácil ya que sólo necesitas imitar el repositorio.

Suponemos que la clase que queremos testear es la siguiente:

```
// src/AppBundle/Salary/SalaryCalculator.php
namespace AppBundle\Salary;

use Doctrine\Common\Persistence\ObjectManager;

class SalaryCalculator
{
    private $entityManager;

    public function __construct(ObjectManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function calculateTotalSalary($id)
    {
        $employeeRepository = $this->entityManager
            ->getRepository('AppBundle:Employee');
        $employee = $employeeRepository->find($id);

        return $employee->getSalary() + $employee->getBonus();
    }
}
```

Ya que el **ObjectManager** se inyecta en la clase a través del constructor, es fácil **pasar un objeto mock dentro de un test**:

```
// tests/AppBundle/Salary/SalaryCalculatorTest.php
namespace Tests\AppBundle\Salary;

use AppBundle\Salary\SalaryCalculator;
use AppBundle\Entity\Employee;
use Doctrine\ORM\EntityRepository;
use Doctrine\Common\Persistence\ObjectManager;

class SalaryCalculatorTest extends \PHPUnit_Framework_TestCase
{
    public function testCalculateTotalSalary()
    {
        // Primero, imitamos el objeto a usar en el test
        $employee = $this->getMock(Employee::class);
        $employee->expects($this->once())
            ->method('getSalary')
            ->will($this->returnValue(1000));
        $employee->expects($this->once())
            ->method('getBonus')
            ->will($this->returnValue(1100));

        // Ahora, imitamos el repositorio de forma que devuelve el mock del empleado
        $employeeRepository = $this
            ->getMockBuilder(EntityRepository::class)
            ->disableOriginalConstructor()
            ->getMock();
        $employeeRepository->expects($this->once())
            ->method('find')
            ->will($this->returnValue($employee));

        // Por último, imitamos el EntityManager para devolver el mock del repositorio
        $entityManager = $this
            ->getMockBuilder(ObjectManager::class)
            ->disableOriginalConstructor()
            ->getMock();
        $entityManager->expects($this->once())
            ->method('getRepository')
            ->will($this->returnValue($employeeRepository));

        $salaryCalculator = new SalaryCalculator($entityManager);
        $this->assertEquals(2100, $salaryCalculator->calculateTotalSalary(1));
    }
}
```

En este ejemplo estás construyendo los mocks desde dentro, creando al empleado que devuelve el **Repository** el cual a su vez es devuelto por el **EntityManager**. De esta forma no hay ninguna clase real en el testing.

#### Cambiar la configuración de la base de datos para functional tests

Si tienes functional tests es mejor que interactúen con una **base de datos real**. La mayoría de las veces es preferible utilizar una **conexión a la base de datos dedicada** para asegurar que no se sobreescriban datos introducidos al desarrollar la aplicación y para poder limpiar la base de datos antes de cada test.

Para hacerlo puedes especificar una configuración de base de datos que sobreescriba la configuración por defecto:

```
# app/config/config_test.yml
doctrine:
    # ...
    dbal:
        host:     localhost
        dbname:   testdb
        user:     testdb
        password: testdb
```

Asegura que la base de datos se ejecuta en _localhost_ y que tiene definidas las credenciales de la base de datos y del usuario.

#### Testear la interacción de varios clientes

Si necesitas simular la interacción entre diferentes clientes (por ejemplo en un **chat**), crea diferentes clientes:
```
// ...

$harry = static::createClient();
$sally = static::createClient();

$harry->request('POST', '/say/sally/Hello');
$sally->request('GET', '/messages');

$this->assertEquals(Response::HTTP_CREATED, $harry->getResponse()->getStatusCode());
$this->assertRegExp('/Hello/', $sally->getResponse()->getContent());
```

Esto funciona salvo cuando tu código mantiene un **estado global** o si depende en una librería de terceros que tiene algún tipo de estado global. En esos casos, puedes aislar los clientes:
```
// ...

$harry = static::createClient();
$sally = static::createClient();

$harry->insulate();
$sally->insulate();

$harry->request('POST', '/say/sally/Hello');
$sally->request('GET', '/messages');

$this->assertEquals(Response::HTTP_CREATED, $harry->getResponse()->getStatusCode());
$this->assertRegExp('/Hello/', $sally->getResponse()->getContent());
```

Los clientes aislados ejecutan transparentemente sus requests en un **proceso PHP dedicado**, evitando por tanto efectos secundarios. Ya que un cliente aislado es más lento, puedes mantener a un cliente en el proceso principal y aislar el resto.