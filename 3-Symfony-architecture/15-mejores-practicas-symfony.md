Las **mejores prácticas para desarrollar aplicaciones web con Symfony** es una guía con recomendaciones para hacer aplicaciones más legibles y escalables respetando la filosoría del framework. En la guía se va haciendo referencia a partes de un **proyecto demo** (un sistema blog), una pequeña aplicación que reúne todos estos consejos (se puede instalar con el comando _symfony demo_ una vez tienes instalado el instalador Symfony).

**Indice de contenido**

1.  Crear el proyecto
2.  Configuración
3.  Organizar la Business Logic
4.  Controllers
5.  Templates
6.  Formularios
7.  Internacionalización
8.  Seguridad
9.  Web Assets
10.  Tests

### 1. Crear el proyecto

Anteriormente se utilizaba **Composer**, el gestor de dependencias para **aplicaciones PHP**, pero la recomendación actual es:

**, que se ha de instalar antes de crear el proyecto.**
```
symfony new blog
```

El comando anterior crea un nuevo directorio llamado **blog** que contiene un nuevo proyecto con la **última versión de Symfony**. Además el instalador comprueba si tu sistema reúne los requisitos para ejecutar aplicaciones Symfony. Sino, se mostrará una lista con los cambios que se necesitan.

#### Estructurar la aplicación

Después de crear la aplicación, al entrar en blog/ se puede ver la siguiente estructura de archivos y directorios:
```
blog/
├─ app/
│  ├─ config/
│  └─ Resources/
├─ bin/
│  └─ console
├─ src/
│  └─ AppBundle/
├─ tests/
│  └─ AppBundle/
├─ var/
│  ├─ cache/
│  ├─ logs/
   └─ sessions/
├─ vendor/
└─ web/
   ├─ app.php
   └─ app_dev.php
```

Esta jerarquía es la convención propuesta por **Symfony** para estructurar tus aplicaciones. El objetivo de cada directorio es el siguiente:

*   _app/config/_. Guarda la configuración de los diferentes entornos.
*   _app/Resources/_. Guarda todas las templates y los archivos de traducción para la aplicación.
*   _src/AppBundle/_. Guarda el código específico de Symfony (controllers y routes), tu código de dominio (como clases Doctrine) y toda la business logic.
*   _var/cache/_. Guarda los archivos cache generados por la aplicación.
*   _var/logs/_. Guarda los archivos log generados por la aplicación.
*   _var/sessions/_. Guarda los archivos de sesión generados por la aplicación.
*   _tests/AppBundle/_. Guarda tests automáticos (como Unit tests) de la aplicación.
*   _vendor/_. Directorio donde Composer instala las dependencias de la aplicación. Nunca se ha de tocar este directorio.
*   _web/_. Guarda todos los archivos front controller y los web assets, como hojas de estilo, archivos JavaScript e imágenes.

#### Bundles de la aplicación

Un bundle está diseñado para ser algo que puede reusarse. Si por ejemplo UserBundle no puede usarse en otras aplicaciones Symfony tal como está, entonces no debería formar un bundle. Si InvoiceBundle depende de ProductBundle, no existen beneficios en tener las dos funcionalidades por separado.

La recomendación es crear un bundle llamado AppBundle para la lógica de la aplicación. Esto hará la aplicación más concisa y fácil de entender. No es necesario poner un prefijo por ejemplo de tu compañía Acme: AcmeAppBundle, porque el bundle de la aplicación no se compartirá.

Otro motivo de crear un bundle es cuando se quiere sobreescribir un comportamiento de un bundle dentro de vendor (por ejemplo un controller).

Si tu aplicación no viene con un bundle autogenerado, puedes autogenerarlo con el siguiente comando:
```
php bin/console generate:bundle --namespace=AppBundle --dir=src --format=annotation --no-interaction
```

### 2\. Configuración

La configuración normalmente engloba diferentes partes de la aplicación (como **infraestructura** y **seguridad**) y diferentes entornos (**dev**, **prod**). Por esta razón **Symfony** recomienda que dividas la configuración de la aplicación en tres partes.

#### Configuración relacionada con la infraestructura

**Define las opciones de configuración relacionadas con la infraestructura en el archivo _app/config/parameters.yml_.**

El archivo por defecto _parameters.yml_ sigue esta recomendación y define las opciones relacionadas con la **base de datos** y el **servidor de email**:
```
# app/config/parameters.yml
parameters:
    database_driver:   pdo_mysql
    database_host:     127.0.0.1
    database_port:     ~
    database_name:     symfony
    database_user:     root
    database_password: ~

    mailer_transport:  smtp
    mailer_host:       127.0.0.1
    mailer_user:       ~
    mailer_password:   ~

    # ...
```

Estas opciones no se definen en el archivo app/config/config.yml porque no tienen nada que ver con el comportamiento de la aplicación. En otras palabras, a tu aplicación no le importa la localización de la base de datos o los datos para acceder a ella, siempre y cuando esté configurada correctamente.

#### Parámetros canónicos

**Define los parámetros de la aplicación en el archivo _app/config/parameters.yml.dist_.**

Cuando se establezca un nuevo parámetro en la configuración de la aplicación, deberías añadirlo también a este archivo e incluir los cambios en tu **sistema de control de versiones**. Así cuando un desarrollador actualice el proyecto o lo lance, Symfony comprobará si existe alguna diferencia entre el archivo _parameters.yml.dist_ y su archivo local _parameters.yml_. Si existe alguna diferencia, **Symfony** le pedirá un valor para el nuevo parámetro y lo añadirá a su archivo local _parameters.yml_.

#### Configuración relacionada con la aplicación

**Define las opciones de configuración relacionadas con el comportamiento de la aplicación en el archivo app/config/config.yml.**

El archivo config.yml contiene las opciones utilizadas por la aplicación para modificar su comportamiento, como el gestor de las notificaciones por email o el [feature toggle](https://en.wikipedia.org/wiki/Feature_toggle) activado. Definiendo estos valores en el archivo _parameters.yml_ añadiría una capa extra de configuración no necesaria ya que estos valores no cambian en función del servidor.

Las opciones de configuración definidas en el archivo _config.yml_ normalmente varían de un entorno a otro. Esta es la razón por la que Symfony incluye los archivos _app/config/config_dev.yml_ y _app/config/config_prod.yml_ de forma que puedes sobreescribir valores específicos para cada entorno.

#### Constantes VS opciones de configuración

Uno de los errores más comunes cuando se define la configuración de la aplicación es crear nuevas opciones para valores que nunca cambian, como el número de objetos devueltos para resultados paginados.

**Utiliza constantes para definir opciones de configuración que apenas cambian.**

Normalmente lo que se venía haciendo en las aplicaciones era incluir lo siguiente para la paginación:

```
# app/config/config.yml
parameters:
    homepage.num_items: 10
```

Lo normal es que este valor no varíe porque siempre se suele emplear el mismo número. Crear una opción de configuración para un valor que nunca vas a configurar no es necesario. La recomendación es definir estos valores como constantes de la aplicación, por ejemplo:

```
// src/AppBundle/Entity/Post.php
namespace AppBundle\Entity;

class Post
{
    const NUM_ITEMS = 10;

    // ...
}
```

La principal ventaja de definir constantes es que puedes usar sus valores en cualquier parte de tu aplicación. Cuando se usan parámetros, sólo están disponibles en lugares con acceso al **Symfony container**.

Las constantes pueden usarse por ejemplo en las templates Twig gracias a la función constant():
```
<p>
    Displaying the {{ constant('NUM_ITEMS', post) }} most recent results.
</p>
```

Y las entidades y repositorios Doctrine pueden ahora acceder fácilmente a estos valores, en cambio no pueden acceder a parámetros del container:

```
namespace AppBundle\Repository;

use Doctrine\ORM\EntityRepository;
use AppBundle\Entity\Post;

class PostRepository extends EntityRepository
{
    public function findLatest($limit = Post::NUM_ITEMS)
    {
        // ...
    }
}
```

La única desventaja notable de utilizar constantes para este tipo de valores de configuración es que no puedes redefinirlar fácilmente en los tests.

#### No hacer configuración semántica

**No definas una configuración de inyección de dependencias semántica para tus bundles.**

Los **bundles de Symfony** tienen dos opciones a la hora de manejar la configuración: configuración normal de services a través del archivo _services.yml_ y configuración semántica a través de la clase especial ***Extension**.

Aunque **la configuración semántica es más potente** y proporciona funcionalidades como **validación de la configuración**, la cantidad de recursos necesarios para definir la configuración no merece la pena para bundles que se supone que servirán como bundles de terceros.

#### Mover opciones delicadas fuera de Symfony

Cuando se trabaja con opciones delicadas, como los datos de acceso para la base de datos, se recomienda que se guarden fuera del proyecto Symfony y hacerlos disponibles a través de variables de entorno en el [Service Container](http://symfony.com/doc/current/cookbook/configuration/external_parameters.html).

### 3\. Organizar la Business Logic

En el mundo del software, [business logic](https://en.wikipedia.org/wiki/Business_logic) o **domain logic** es "_la parte del programa que codifica la reglas del proyecto que determinan como han de crearse, mostrarse cambiarse o guardarse los datos_".

En aplicaciones Symfony, business logic es todo el código personalizado que escriber para tu aplicación que no es específico del framework (routing, controllers, etc). Domain classes, Dcotrine entities y clases regulares PHP que se usan como services son buenos ejemplos de business logic.

Para la mayoría de proyectos, deberías guardar todo dentro de AppBundle. En este directorio puedes crear los directorios que hagan falta para organizar las cosas:
```
symfony-project/
├─ app/
├─ src/
│  └─ AppBundle/
│     └─ Utils/
│        └─ MyClass.php
├─ tests/
├─ var/
├─ vendor/
└─ web/
```

#### Guardar clases fuera del bundle

Si quieres puedes crear tu propio **namespace** dentro del directorio src/ y poner cosas ahí:
```
symfony-project/
├─ app/
├─ src/
│  ├─ Acme/
│  │   └─ Utils/
│  │      └─ MyClass.php
│  └─ AppBundle/
├─ tests/
├─ var/
├─ vendor/
└─ web/
```

El enfoque recomendable de utilizar el directorio _AppBundle/_ es por simplicidad. Si eres un usuario avanzado y sabes qué debería ir dentro de un bundle y qué puede estar fuera, puedes hacerlo.

#### Formato y nombramiento de los Services

La aplicación del blog necesita una utilidad que pueda transformar el título de un post (como "Hello World") en un slu g("hello-world"). El slug se usará como parte de la URL.

Creamos una clase **Slugger** dentro de _src/AppBundle/Utils_ y añadimos el método _slugify()_:

```
// src/AppBundle/Utils/Slugger.php
namespace AppBundle\Utils;

class Slugger
{
    public function slugify($string)
    {
        return preg_replace(
            '/[^a-z0-9]/', '-', strtolower(trim(strip_tags($string)))
        );
    }
}
```

Lo siguiente es definir un service para esa clase:

```
# app/config/services.yml
services:
    # Mantén los nombres de los services cortos
    app.slugger:
        class: AppBundle\Utils\Slugger
```

Tradicionalmente la convención de nombrado para un service era incluir el nombre de la clase y la localización para evitar colisiones de nombres, por lo que hubiera sido app.utils.slugger. Pero utilizando nombres de services cortos el código será más fácil de leer y utilizar.

**El nombre de los services de tu aplicación deben ser tan cortos como sea posible, pero suficientemente únicos como para que puedan diferenciarse.**

Ahora se puede utilizar el slugger personalizado en cualquier clase controller, como en **AdminController**:

```
public function createAction(Request $request)
{
    // ...

    if ($form->isSubmitted() && $form->isValid()) {
        $slug = $this->get('app.slugger')->slugify($post->getTitle());
        $post->setSlug($slug);

        // ...
    }
}
```

#### El formato de services: YAML

En las secciones anteriores se ha empleado YAML para definir los services.

**Emplea el formato YAML para definir tus services.**

Esta es una recomendación controvertida, ya que unos desarrolladores prefieren YAML y otros XML. Ambos formatos tienen el mismo rendimiento, por lo que depende del desarrollador.

Desde Symfony recomiendan YAML porque es más amigable para beginners y es más conciso.

#### Parámetros en las clases de los services

En el service anterior no hemos configurado el namespace de clase como parámetro. En el siguiente ejemplo sí:

```
# app/config/services.yml

# Definición del service con namespace como parámetro
parameters:
    slugger.class: AppBundle\Utils\Slugger

services:
    app.slugger:
        class: '%slugger.class%'
```

Esta práctica es incómoda y completamente innecesaria para tus propios services.

**No definas parámetros para las clases de tus services.**

Esta práctica se adoptaba en bundles de terceros de forma equivocada. Cuando **Symfony** introdujo el **Service Container**, algunos desarrolladores usaron esta técnica para permitir sobreescribir services. Sin embargo, sobreescribir un service simplemente cambiando el nombre de la clase es un caso bastante raro porque normalmente el nuevo service tiene direfentes argumentos en el constructor.

#### Usando un Persistence Layer

Symfony es un **framework HTTP** que sólo se preocupe por generar una **respuesta HTTP** para cada **HTTP request**. Es por ello que Symfony no proporciona una forma de comunicarse con un persistence layer (por ejemplo una base de datos, una external API). Puedes elegir cualquier libraría o estrategia que quieras para esto.

En la práctica, muchas **aplicaciones Symfony** confían en [Doctrine project](http://www.doctrine-project.org/) para definir sus modelos utilizando entidades y repositorios. Al igual que con la business logic, se recomienda guardar las entidades Doctrine dentro de AppBundle.

Las tres entidades definidas en la aplicación blog sirven de ejemplo:
```
symfony-project/
├─ ...
└─ src/
   └─ AppBundle/
      └─ Entity/
         ├─ Comment.php
         ├─ Post.php
         └─ User.php
```

De nuevo si eres un usuario avanzado puedes guardarlas en tu propio namespace en _src/. _

#### Doctrine Mapping Information

Las **entidades de Doctrine** son **objetos PHP** que se guardan en algún tipo de **base de datos**. Doctrine sólo sabe acerca de tus entidades a través de los metadatos configurados en tus **clases model**. Doctrine soporta cuatro formatos para metadatos: YAML, XML, PHP y anotaciones.

**Utiliza anotaciones para definir la información de las entidades Doctrine.**

Las anotaciones son con diferencia la forma más conveniente y ágil de configurar la información de las entidades:

```
namespace AppBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Doctrine\Common\Collections\ArrayCollection;

    /**
     * @ORM\Entity
     */
class Post
{
    const NUM_ITEMS = 10;

    /**
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string")
     */
    private $title;

    /**
     * @ORM\Column(type="string")
     */
    private $slug;

    /**
     * @ORM\Column(type="text")
     */
    private $content;

    /**
     * @ORM\Column(type="string")
     */
    private $authorEmail;

    /**
     * @ORM\Column(type="datetime")
     */
    private $publishedAt;

    /**
     * @ORM\OneToMany(
     *      targetEntity="Comment",
     *      mappedBy="post",
     *      orphanRemoval=true
     * )
     * @ORM\OrderBy({"publishedAt" = "ASC"})
     */
    private $comments;

    public function __construct()
    {
        $this->publishedAt = new \DateTime();
        $this->comments = new ArrayCollection();
    }

    // getters and setters ...
}
```

Todos los formatos tienen el mismo rendimiento, por lo que de nuevo esto depende del desarrollador.

#### Data Fixtures

El soporte a fixtures no está activado por defecto en Symfony, se ha de ejecutar el siguiente comando para instalar el bundle Doctrine fixtures:
```
composer require "doctrine/doctrine-fixtures-bundle"
```

Entonces, activa el bundle en _AppKernel.php_, pero sólo para los entornos **dev** y **test**:

```
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            // ...
            $bundles[] = new Doctrine\Bundle\FixturesBundle\DoctrineFixturesBundle();
        }

        return $bundles;
    }

    // ...
}
```

Se recomienda crear sólo una **fixture class** por simplicidad, pero es posible añadir más clases.

Suponiendo que tienes al menos una fixture class y que el acceso a la base de datos está configurado correctamente, puedes cargar los fixtures ejecutando el siguiente comando:
```
$ php bin/console doctrine:fixtures:load

Careful, database will be purged. Do you want to continue Y/N ? Y
  > purging database
  > loading AppBundle\DataFixtures\ORM\LoadFixtures
```

#### Estándares de código

El código fuente de Symfony sigue los estándares de código PSR-1 y PSR-2 definidos en la comunidad de PHP.

### 4\. Controllers

**Symfony** sigue la filosofía de "**controllers** ligeros y **models** pesados". Esto significa que los controllers deben simplemente coordinar las diferentes partes de la aplicación.

Como regla de oro se debería seguir la regla 5-10-20, donde los controllers deberían definir 5 variables o menos, contener 10 acciones o menos e incluir 20 líneas de código o menos en cada acción. Esto no es una ciencia exacta, pero sirve para saber cuándo el código se ha de refactorizar y sacar fuera del controller y ponerlo en un service.

**Haz que tu controller extienda el controller base FrameworkBundle y utiliza anotaciones para configurar el routing, caching y seguridad siempre que sea posible.**

Acoplar los controllers en la estructura subyacente del framework permite aprovechar todas sus características y mejora su productividad.

Y ya que los controllers deben ser ligeros y no contener nada más que unas líneas de código, estar horas intentando desacoplarlos del framework no resulta producente a largo plazo. El tiempo malgastado no merece la pena.

Además, emplear anotaciones para el routing, cachig y seguridad simplifica la configuración. No necesitas emplear decenas de archivos creados con formatos diferentes (YAML, XML, PHP): toda la configuración está donde la necesitas y sólo emplea un formato.

En general, esto significa que deberías desacoplar la business logic del framework a la vez que acoplar los controllers y el routing en el framework para exprimirlo al máximo.

#### Configuración del Routing

Para cargar routes definidas como anotaciones en tus controllers, añade la siguiente configuración en el principal archivo de configuración del routing:

```
# app/config/routing.yml
app:
    resource: '@AppBundle/Controller/'
    type:     annotation
```

Esta configuración cargará anotaciones de cualquier controller guardado en el directorio _src/AppBundle/Controller_ e incluso de sus subdirectorios. Así que si tu aplicación define muchos controllers, es posible reorganizarlos en subdirectorios:
```
<your-project>/
├─ ...
└─ src/
   └─ AppBundle/
      ├─ ...
      └─ Controller/
         ├─ DefaultController.php
         ├─ ...
         ├─ Api/
         │  ├─ ...
         │  └─ ...
         └─ Backend/
            ├─ ...
            └─ ...
```

#### Configuración de las templates

**No utilices la anotación @Template() para configurar la template usada por el controller.**

La anotación **@Template** es útil, pero también esconde algo de magia. Desde Symfony **recomiendan no utilizarla** porque esa magia puede no merecer la pena.

La mayoría de las veces @Template se utiliza sin ningún parámetro, lo que lo hace más difícil para saber que template está siendo renderizada. Además lo hace menos obvio para beginners que un controller debe siempre devolver un objeto **Response** (a no ser que utilices un _view layer_).

#### Cómo debe ser un controller

Considerando todo lo anterior, algo así debería ser el controller de la página principal de tu aplicación:

```
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

class DefaultController extends Controller
{
    /**
     * @Route("/", name="homepage")
     */
    public function indexAction()
    {
        $posts = $this->getDoctrine()
            ->getRepository('AppBundle:Post')
            ->findLatest();

        return $this->render('default/index.html.twig', array(
            'posts' => $posts
        ));
    }
}
```

#### Usar el ParamConverter

Si utilizas **Doctrine**, puedes opcionalmente usar **ParamConverter** para consultar automáticamente una entidad y pasarla como argumento del controller.

**Utiliza ParamConverter para consultar automáticamente entidades de Doctrine cuando es simple y conveniente.**

```
use AppBundle\Entity\Post;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

/**
* @Route("/{id}", name="admin_post_show")
*/
public function showAction(Post $post)
{
    $deleteForm = $this->createDeleteForm($post);

    return $this->render('admin/post/show.html.twig', array(
        'post'        => $post,
        'delete_form' => $deleteForm->createView(),
    ));
}
```

Normalmente esperarías un argumento _$id_ en el método _showAction_. En cambio, al crear un nuevo argumento _$post_ y utilizar type hinting con la clase Post (que es una entidad Doctrine), el **ParamConverter** automáticamente busca un objeto cuya propiedad _$id_ coincide con el valor {id}. También mostrará una página 404 si no se ha podido encontrar Post.

#### Uso más complejo

Esto funciona sin ninguna configuración ya que el **wildcard** _{id}_ coincide con el nombre de la propiedad de la entidad. Si no es true, o si tienes incluso una lógica más compleja, la forma más fácil es consultar la entidad de forma manual. En la aplicación demo se ve esta situación en **CommentController**:

```
/**
* @Route("/comment/{postSlug}/new", name = "comment_new")
*/
public function newAction(Request $request, $postSlug)
{
    $post = $this->getDoctrine()
        ->getRepository('AppBundle:Post')
        ->findOneBy(array('slug' => $postSlug));

    if (!$post) {
        throw $this->createNotFoundException();
    }

    // ...
}
```

También puedes utilizar la configuración **@ParamConverter**, que es muy flexible:

```
use AppBundle\Entity\Post;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;
use Symfony\Component\HttpFoundation\Request;

/**
* @Route("/comment/{postSlug}/new", name = "comment_new")
* @ParamConverter("post", options={"mapping": {"postSlug": "slug"}})
*/
public function newAction(Request $request, Post $post)
{
    // ...
}
```

La cuestión es que el shortcut de **ParamConverter** es perfecto para situaciones sencillas. Pero no hay que olvidar que consultar entidades directamente es realmente sencillo.

#### Pre y Post Hooks

Si necesitas ejecutar código antes o después de la ejecución de los controllers, puedes emplear el [componente EventDispatcher](http://symfony.com/doc/current/cookbook/event_dispatcher/before_after_filters.html).

### 5\. Templates

Cuando se creó PHP hace ya más de 20 años, a los desarrolladores les encantaba su simplicidad y lo bien que se mezclaba con HTML y código dinámico. Con el paso del tiempo, otros lenguajes de templates (como Twig) se crearon para hacer las cosas todavía más fáciles.

**Utiliza el formato de plantillas de Twig para tus templates.**

Las templates en PHP son más verbosas que en Twig ya que no tienen soporte nativo para funcionalidades modernas que necesitan ahora las templates, como herencia, escape automático y argumentos nombrados para filtros y funciones.

Twig es el formato por defecto en Symfony y tiene la comunidad más grande en el mundo de los motores de templates (_non-PHP_) (se utiliza en grandes proyectos como Drupal 8).

Además, Twig es el único formato de templates con soporte garantizado en Symfony 3\. De hecho, PHP podría eliminarse de los motores de templates soportados.

#### Localización de las templates

**Guarda todas las templates de tu aplicación en app/Resources/views.**

Tradicionalmente los desarrolladores de Symfony guardaban las templates de la aplicación en el directorio Resources/views de cada bundle. Para referirse a ellas se utilizaba su nombre lógico: _AcmeDemoBundle:Default:index.html.twig_.

Pero para las templates utilizadas en tu aplicación, es más conveniente guardarlas en el directorio _app/Resources/views/_. Para principiantes, esto simplifica drásticamente sus nombres:

| | |
| -------- | -------- |
| **Templates guardadas en Bundles** | **Templates guardadas en app/** |
| AcmeDemoBundle:Default:index.html.twig | default/index.html.twig |
| ::layout.html.twig | layout.html.twig |
| AcmeDemoBundle::index.html.twig | index.html.twig |
| AcmeDemoBundle:Default:subdir/index.html.twig | default/subdir/index.html.twig |
| AcmeDemoBundle:Default/subdir:index.html.twig | default/subdir/index.html.twig |

Otra ventaja es que centralizando los templates se simplifica el trabajo a los diseñadores. Así no tienen que ir mirando en diferentes directorios de diferentes bundles.

**Utiliza snake_case en minúsculas para nombres de directorios y templates.**

#### Twig Extensions

**Define las extensiones de Twig en el directorio AppBundle/Twig/ y configuralas mediante el archivo app/config/services.yml.**

La aplicación blog necesita un filtro Twig customizado **md2html** de forma que podamos transformar los contenidos del Markdown de cada post en HTML.

Para hacerlo, primero instalamos el parser [Parsedown](http://parsedown.org/) Markdown como una nueva dependencia del proyecto:
```
composer require erusev/parsedown
```

Creamos ahora un service **Markdown** que será usado después como **extensión de Twig**. La definición del service solo requiere la ruta de la clase:

```
# app/config/services.yml
services:
    # ...
    markdown:
        class: AppBundle\Utils\Markdown
```

Y la clase **Markdown** simplemente necesita definir un simple método para **transformar el contenido del Markdown en HTML**:

```
namespace AppBundle\Utils;

class Markdown
{
    private $parser;

    public function __construct()
    {
        $this->parser = new \Parsedown();
    }

    public function toHtml($text)
    {
        $html = $this->parser->text($text);

        return $html;
    }
}
```

Lo siguiente es crear una nueva extensión Twig y definir un nuevo filtro llamado **md2html** utilizando la clase **Twig_SimpleFilter**. Inyecta el nuevo service markdown en el constructor de la extensión Twig:

```
namespace AppBundle\Twig;

use AppBundle\Utils\Markdown;

class AppExtension extends \Twig_Extension
{
    private $parser;

    public function __construct(Markdown $parser)
    {
        $this->parser = $parser;
    }

    public function getFilters()
    {
        return array(
            new \Twig_SimpleFilter(
                'md2html',
                array($this, 'markdownToHtml'),
                array('is_safe' => array('html'))
            ),
        );
    }

    public function markdownToHtml($content)
    {
        return $this->parser->toHtml($content);
    }

    public function getName()
    {
        return 'app_extension';
    }
}
```

Finalmente define un nuevo service para activar esta extensión Twig en la app (el nombre del service es irrelevante porque no lo empleas en tu propio código):

```
# app/config/services.yml
services:
    app.twig.app_extension:
        class:     AppBundle\Twig\AppExtension
        arguments: ['@markdown']
        public:    false
        tags:
            - { name: twig.extension }
```

### 6\. Formularios

Los formularios son uno de los componentes menos explotados debido a su largo alcance e interminable lista de funcionalidades.

**Define los formularios como clases PHP.**

El componente Form te permite construir formularios dentro del código de tu controller. Esto está bien si no necesitas reusar el formulario en algún otro lugar. Pero para una buena organización y poder reutilizarlos, se recomienda que definas cada formulario en su propia clase PHP:

```
namespace AppBundle\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\DateTimeType;

class PostType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('title')
            ->add('summary', TextareaType::class)
            ->add('content', TextareaType::class)
            ->add('authorEmail', EmailType::class)
            ->add('publishedAt', DateTimeType::class)
        ;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\Post'
        ));
    }
}
```

**Pon las clases form type en el namespace AppBundle\Form, a no ser que utilices otras clases de formularios persolalizadas para transformar los datos.**

Para utilizar la clase, utiliza _createForm_ y pasa el nombre de la clase (fully qualified):

```
// ...
use AppBundle\Form\PostType;

// ...
public function newAction(Request $request)
{
    $post = new Post();
    $form = $this->createForm(PostType::class, $post);

    // ...
}
```

#### Registrar formularios como services

Puedes también **registrar tu propio form type como service**. No es recomendable a no ser que quieras reusar el nuevo form type en muchos sitios o embeberlo en otros formularios directamente o a través de [CollectionType](http://symfony.com/doc/current/reference/forms/types/collection.html).

Para la mayoría de formularios que se utilizan sólo para editar o crear algo, registrar el formulario como un service es excesivo, y hace más difícil averiguar exactamente la clase del formulario que se está utilizando en el controller.

#### Configuración de botones de formularios

Las clases de formularios deberían ser agnósticas respecto a dónde serán usadas. Esto hace más fácil que sean reutilizadas después:

**Añade botones en las templates, no en las clases de formularios o en los controllers.**

Se pueden añadir botones como campos en el propio formulario. Es una buena forma de simplificar la template que renderiza tu formulario. Pero si añades los botones directamente en la clase del formulario, limitarás el scope de ese formulario:

```
class PostType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            // ...
            ->add('save', SubmitType::class, array('label' => 'Crear Post'))
        ;
    }

    // ...
}
```

Este formulario puede haber sido diseñado para crear posts, pero si realmente quieres reusarlo para editar posts, la etiqueta del botón no sería la correcta. En lugar de eso, algunos desarrolladores configuran los botones de formularios en el controller:

```
namespace AppBundle\Controller\Admin;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use AppBundle\Entity\Post;
use AppBundle\Form\PostType;

class PostController extends Controller
{
    // ...

    public function newAction(Request $request)
    {
        $post = new Post();
        $form = $this->createForm(PostType::class, $post);
        $form->add('submit', SubmitType::class, array(
            'label' => 'Crear',
            'attr'  => array('class' => 'btn btn-default pull-right')
        ));

        // ...
    }
}
```

Esto es también un error importante, porque estás mezclando la **presentación** (etiquetas, clases css, etc) con código **PHP puro**. **Separation of concerns** es siempre una regla que hay que seguir, por lo que es mejor poner las cosas relacionadas con las views en el **view layer**:

```
{{ form_start(form) }}
{{ form_widget(form) }}

<input type="submit" value="Create"
           class="btn btn-default pull-right" />
{{ form_end(form) }}
```

#### Renderizar el formulario

Existen muchas varias de renderizar el formulario, que van desde renderizarlo entero en una línea o hacerlo cada parte en un campo independiente. La mejor forma depende de cuanta customización necesitas.

Una de las formas más simples, especialmente útil durante el desarrollo, es renderizar las etiquetas del formulario utilizando _form_widget()_ para renderizar todos los campos:

```
{{ form_start(form, {'attr': {'class': 'my-form-class'} }) }}
{{ form_widget(form) }}
{{ form_end(form) }}
```

Si necesitas más control sobre cómo se renderizan tus campos, deberías remover la función _form_widget(form)_ y renderizarlos individualmente. [Este artículo](http://symfony.com/doc/current/cookbook/form/form_customization.html) explica las diferentes formas de personalizarlos y cómo el formulario renderiza a un nivel global utilizando **form theming**.

#### Manejar el envío del formulario

El **envío de un formulario** normalmente es de la siguiente forma:

```
public function newAction(Request $request)
{
    // construir el formulario ...

    $form->handleRequest($request);
    if ($form->isSubmitted() && $form->isValid()) {
        $em = $this->getDoctrine()->getManager();
        $em->persist($post);
        $em->flush();

        return $this->redirect($this->generateUrl(
            'admin_post_show',
            array('id' => $post->getId())
        ));
    }
    // renderizar la template
}
```

Hay dos cosas a destacar de aquí. Primero, se recomienda que se emplee sólo una _action_ para renderizar el formulario y para manejar el envío. Por ejemplo, podrías tener un **newAction** que sólo renderizase el formulario y un **createAction** que sólo procesara el envío del formulario. Ambas acciones serán casi idénticas, por lo que es más sencillo dejar que newAction se encargue de todo.

Segundo, se recomienda usar _$form->isSubmitted()_ en la sentencia **if** para que se vea más claro. Ténicamente no es necesario, ya que _isValid()_ primero llama a _isSubmitted()_, pero sin esto, el flujo no se ve bien ya que parece que el formulario es siempre procesado (incluso con el GET request).

### 7\. Internacionalización

La internacionalización y localización adaptan las aplicaciones y sus contenidos a una región o lenguaje específicos para los usuarios. en Symfony esta es una opción incorporada que ha de ser activada antes de usarse. Para hacerlo, se descomenta **translator** de la configuración y se establece tu **locale**:

```
# app/config/config.yml
framework:
    # ...
    translator: { fallbacks: ['%locale%'] }
# app/config/parameters.yml
parameters:
    # ...
    locale:     en
```

#### Formato de los archivos de traducción

El componente Symfony Translation soporta muchos formatos de traducción diferentes: PHP, Qt, .po, .mo, JSON, CSV, INI, etc.

**Utiliza el formato XLIFF para los archivos de traducción.**

De todos los formatos de traducción disponibles, sólo XLIFF y gettext tienen soporte completo en las herramientas utilizadas por traductores profesionales. Y ya que está basado en XML, puedes validar los contenidos de XLIFF a medida que los escribes.

También hay soporte a las anotaciones dentro de los archivos XLIFF, hacíendolos más amigables para los traductores. Al fin y al cabo las buenas traducciones dependen del contexto, y las anotaciones de XLIFF permiten definir cada contexto.

El bundle JMSTranslationBundle ofrece una interfaz para ver y editar estos archivos de traducción. También tiene extractores avanzados que pueden leer tu proyecto y automáticamente actualizar los archivos XLIFF.

#### Localización de los archivos de traducción

**Guarda los archivos de traducción en el directorio app/Resources/translations/.**

Tradicionalmente, los desarrolladores de **Symfony** creaban estos archivos en cada bundle en _Resources/translations/_.

Pero como el directorio _app/Resources/_ es considerado el directorio global para los recursos de la aplicación, guardar las traducciones en _app/Resources/translations/_ las centraliza y les da prioridad sobre cualquier otro archivo de traducción. Esto te permite sobrescribir traducciones definidas en bundles de terceros.

#### Translation _keys_

**Usa siempre keys para traducciones en lugar de strings de contenido.**

Utilizar _keys_ simplifica la administración de los archivos de traducción porque puedes cambiar los contenidos originales sin tener que actualizar todos los archivos de traducción.

Las _keys_ deben siempre describir su **objetivo** y no su **localización**. Por ejemplo, si un formulario tiene un campo con la etiqueta "**Username**", un buen key sería **label.username**, no **edit_form.label.username**.

#### Ejemplo de archivo de traducción

Aplicando las buenas prácticas anteriores, un ejemplo de archivo de traducción en Inglés sería:

```
<!-- app/Resources/translations/messages.en.xlf -->
<?xml version="1.0"?>
<xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
    <file source-language="en" target-language="en" datatype="plaintext" original="file.ext">
        <body>
            <trans-unit id="1">
                <source>title.post_list</source>
                <target>Post List</target>
            </trans-unit>
        </body>
    </file>
</xliff>
```

### 8\. Seguridad

#### Auntenticación y Firewalls

Puedes configurar Symfony para autenticar a tus usuarios utilizando el método que quieras y cargando la información del usuario desde cualquier parte. Esta es una sección complicada que se trata más detenidamente en el [cookbook](http://symfony.com/doc/current/cookbook/security/index.html).

Independientemente de tus necesidades, la autenticación se configura en _security.yml_, principalmente en el key **firewalls**.

**A no ser que tengas dos sistemas de autenticación diferentes y usuarios (por ejemplo un formulario de login para el sitio principal y un sistema token para tu API únicamente) se recomienda tener sólo una entrada firewall con la key _anonymous_ activada.**

La mayoría de aplicaciones tienen sólo un sistema de autentificación y un grupo de usuarios. Por esta razón sólo es necesaria una entrada firewall. Hay excepciones, especialmente cuando tienes separadas las secciones web y API. El objetivo es mantener las cosas sencillas.

Adicionalmente, deberías usar la key _**anonymous**_ en tu firewall. Si necesitas que los usuarios hagan login en diferentes secciones de tu sitio, utiliza el área **access_control**.

**Utiliza bcrypt para codificar los passwords de los usuarios.**

Es más recomendable utilizar **bcrypt** a sha-512\. Las principales ventajas de bcrypt son la inclusión de un valor _salt_ para protegerse frente a [ataques rainbow](http://diego.com.es/encriptacion-y-contrasenas-en-php#LaImportanciaDeLosHashesSeguros) y su naturaleza, que permite hacerlo más lento para este tipo de ataques forzosos.

Con todo esto, lo siguiente es la configuración:

```
# app/config/security.yml
security:
    encoders:
        AppBundle\Entity\User: bcrypt

    providers:
        database_users:
            entity: { class: AppBundle:User, property: username }

    firewalls:
        secured_area:
            pattern: ^/
            anonymous: true
            form_login:
                check_path: security_login_check
                login_path: security_login_form

            logout:
                path: security_logout
                target: homepage

# ... access_control existe, pero no se muestra aquí
```

El código fuente de la aplicación demo muestra comentarios que explican cada sección,

#### Autorización

Symfony proporciona diferentes formas para la autorización, incluyendo la configuración **access_control** en _security.yml_, la anotación **@Security** y utilizando _isGranted_ en el service **security.authorization_checker**.

**.
Siempre que sea posible, utilizar la anotación** **.
Comprobar la seguridad directamente con el service**  **en situaciones más complejas.**

Existen también diferentes formas de centralizar la lógica de la aplicación, como con un sistema **security voter** o con **ACL**.

 **personalizado.
Para restringir el acceso a cualquier objeto de cualquier usuario a través de una interface admin, utiliza** **.**

#### La anotación @Security

Para controlar el acceso se puede utilizar la anotación @Security siempre que sea posible. Es fácil de leer y se coloca encima de cada _action_.

En la aplicación demo, necesitamos un **ROLE_ADMIN** para crear un nuevo post. Utilizando **@Security**:

```
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;
// ...

/**
* Muestra un formulario para crear una nueva entidad Post.
*
* @Route("/new", name="admin_post_new")
* @Security("has_role('ROLE_ADMIN')")
*/
public function newAction()
{
    // ...
}
```

**Utilizando expresiones para restricciones de seguridad más complejas con @Security**

Si la lógica de la seguridad de tu aplicación es algo más compleja, puedes usar expresiones dentro de @Security. En el siguiente ejemplo, un usuario sólo puede acceder al controller si su email coincide con el valor devuelto por _getAuthorEmail_ en el objeto **Post**:

```
use AppBundle\Entity\Post;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;

/**
* @Route("/{id}/edit", name="admin_post_edit")
* @Security("user.getEmail() == post.getAuthorEmail()")
*/
public function editAction(Post $post)
{
    // ...
}
```

Esto requiere el uso de **ParamConverter**, que automáticamente consulta el objeto Post y lo coloca en el argumento _$post._ Esto es lo que hace posible que se pueda usar la variable post en la expresión.

Esto tiene un inconveniente importante: una expresión en una anotación no puede ser reusada fácilmente en otras partes de la aplicación. Imagina que quieres añadir un enlace en una template que sólo será vista por autores. Ahora tendrás que repetir la expresión con Twig:

```
{% if app.user and app.user.email == post.authorEmail %}
    <a href=""> ... </a>
{% endif %}
```

La solución más fácil, si tu lógica es suficientemente simple, es añadir un nuevo método a la entidad Post que comprueba si un usuario es su autor:

```
// src/AppBundle/Entity/Post.php
// ...

class Post
{
    // ...

    /**
     * ¿Es el usuario actual el autor de este Post?
     *
     * @return bool
     */
    public function isAuthor(User $user = null)
    {
        return $user && $user->getEmail() == $this->getAuthorEmail();
    }
}
```

Ahora puedes reusar este método en **@Security**:

```
use AppBundle\Entity\Post;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;

/**
* @Route("/{id}/edit", name="admin_post_edit")
* @Security("post.isAuthor(user)")
*/
public function editAction(Post $post)
{
    // ...
}
```

Y en la **template**:

```
{% if post.isAuthor(app.user) %}
    <a href=""> ... </a>
{% endif %}
```

#### Comprobar permisos sin @Security

El ejemplo anterior con **@Security** funciona porque estamos usando **ParamConverter**, que le da a la expresión acceso a la variable post. Si no usas esto, o tienes otro uso más avanzado, puedes hacer siempre la misma comprobación en PHP:

```
/**
* @Route("/{id}/edit", name="admin_post_edit")
*/
public function editAction($id)
{
    $post = $this->getDoctrine()->getRepository('AppBundle:Post')
        ->find($id);

    if (!$post) {
        throw $this->createNotFoundException();
    }

    if (!$post->isAuthor($this->getUser())) {
        $this->denyAccessUnlessGranted('edit', $post);

        // or without the shortcut:
        //
        // use Symfony\Component\Security\Core\Exception\AccessDeniedException;
        // ...
        //
        // if (!$this->get('security.authorization_checker')->isGranted('edit', $post)) {
        //    throw $this->createAccessDeniedException();
        // }
    }

    // ...
}
```

#### Security voters

Si la lógica de tu seguridad es más compleja y no puede ser centralizada en un método como _isAuthor_, podrías usar _voters_ personalizados. Son más fáciles que ACLs y te darán la flexibilidad que necesitas en la mayoría de los casos.

Primero creamos la clase voter. El siguiente ejemplo muestra un voter que implementa la misma lógica _getAuthoerEmail_ utilizada antes:

namespace AppBundle\Security;

```
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\AccessDecisionManagerInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;
use Symfony\Component\Security\Core\User\UserInterface;
use AppBundle\Entity\Post;

// Voter class requires Symfony 2.8 or higher version
class PostVoter extends Voter
{
    const CREATE = 'create';
    const EDIT   = 'edit';

    /**
     * @var AccessDecisionManagerInterface
     */
    private $decisionManager;

    public function __construct(AccessDecisionManagerInterface $decisionManager)
    {
        $this->decisionManager = $decisionManager;
    }

    protected function supports($attribute, $subject)
    {
        if (!in_array($attribute, array(self::CREATE, self::EDIT))) {
            return false;
        }

        if (!$subject instanceof Post) {
            return false;
        }

        return true;
    }

    protected function voteOnAttribute($attribute, $subject, TokenInterface $token)
    {
        $user = $token->getUser();
        /** @var Post */
        $post = $subject; // $subject must be a Post instance, thanks to the supports method

        if (!$user instanceof UserInterface) {
            return false;
        }

        switch ($attribute) {
            case self::CREATE:
                // if the user is an admin, allow them to create new posts
                if ($this->decisionManager->decide($token, array('ROLE_ADMIN'))) {
                    return true;
                }

                break;
            case self::EDIT:
                // if the user is the author of the post, allow them to edit the posts
                if ($user->getEmail() === $post->getAuthorEmail()) {
                    return true;
                }

                break;
        }

        return false;
    }
}
```

Para activar el **security voter** en la aplicación, definimos un nuevo **service**:

```
# app/config/services.yml
services:
    # ...
    post_voter:
        class:      AppBundle\Security\PostVoter
        arguments: ['@security.access.decision_manager']
        public:     false
        tags:
           - { name: security.voter }
```

Ahora puedes usar el voter con la anotación **@Security**:

```
/**
* @Route("/{id}/edit", name="admin_post_edit")
* @Security("is_granted('edit', post)")
*/
public function editAction(Post $post)
{
    // ...
}
```

También puedes usar esto directamente con el service **security.authorization_checker** o con un shortcut más sencillo en un **controller**:

```
/**
* @Route("/{id}/edit", name="admin_post_edit")
*/
public function editAction($id)
{
    $post = ...; // query for the post

    $this->denyAccessUnlessGranted('edit', $post);

    // or without the shortcut:
    //
    // use Symfony\Component\Security\Core\Exception\AccessDeniedException;
    // ...
    //
    // if (!$this->get('security.authorization_checker')->isGranted('edit', $post)) {
    //    throw $this->createAccessDeniedException();
    // }
}
```

#### Algunos tips

*   Para la administración de usuarios puedes emplear [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle). 
*   Añadir un [Recordarme](http://symfony.com/doc/current/cookbook/security/remember_me.html) en el Login.
*   Crear [usuarios impersonales](http://symfony.com/doc/current/cookbook/security/impersonating_user.html) (especialmente útil en soporte cuando se quiere reproducir el problema de un usuario).
*   Si tu empresa emplea un método de login no soportado por Symfony, puedes usar tu propio [user provider](http://symfony.com/doc/current/cookbook/security/custom_provider.html) y [authentication provider](http://symfony.com/doc/current/cookbook/security/custom_authentication_provider.html).

### 9\. Web Assets

Los web assets son cosas como CSS, JavaScript y archivos de imágenes que hacen que el frontend de tu sitio luzca bien. Anteriormente los desarrolladores Symfony guardaban estos assets en el directorio Resources/public/ de cada bundle.

**Guarda tus assets en el directorio web/.**

Guardar los assets en diferentes bundles lo hace más difícil de administrarlos. Si todos los assets están en un directorio será más fácil para los diseñadores.

Las templates también se benefician de la centralización de los assets, porque los enlaces son más amigables:

```
<link rel="stylesheet" href="{{ asset('css/bootstrap.min.css') }}" />
<link rel="stylesheet" href="{{ asset('css/main.css') }}" />

{# ... #}

<script src="{{ asset('js/jquery.min.js') }}"></script>
<script src="{{ asset('js/bootstrap.min.js') }}"></script>
```

Ten en cuenta que _web/_ es un directorio público y cualquier cosa que se guarde en este directorio será accesible públicamente, incluyendo los archivos assets originales (como archivos Sass, LESS y CoffeeScript).

#### Usar Assetic

Assetic no se incluye directamente en la **Symfony Standard Edition**, para ver cómo instalarlo utiliza [este enlace](http://symfony.com/doc/current/cookbook/assetic/asset_management.html).

Actualmente lo más común es combinar y minimizar los archivos CSS y Javascript para mejorar el rendimiento. También es frecuente utilizar LESS o Sass, que requieren alguna forma de procesarlos para obtener archivos CSS.

Existen muchas herramientas para solucionar estos problemas, incluyendo herramientas de frontend como GruntJS.

**Utiliza Assetic para compilar, combinar y minimizar web assets, a no ser que te sea más cómodo hacerlo con herramientas de frontend como GruntJS.**

Assetic es un administrador de Assets capaz de compilar assets desarrollados con diferentes tecnologás frontend como LESS, Sass y CoffeeScript. Combinar todos tus assets con Assetic es sencillo agrupándolos de la siguiente forma:

```
{% stylesheets
    'css/bootstrap.min.css'
    'css/main.css'
    filter='cssrewrite' output='css/compiled/app.css' %}
    <link rel="stylesheet" href="{{ asset_url }}" />
{% endstylesheets %}

{# ... #}

{% javascripts
    'js/jquery.min.js'
    'js/bootstrap.min.js'
    output='js/compiled/app.js' %}
    <script src="{{ asset_url }}"></script>
{% endjavascripts %}
```

#### Aplicaciones frontend

Actualmente, tecnologías frontend como AngularJS están siendo muy populares para desarrollar aplicaciones web frontend que se comunican con una API.

Si estás desarrollando una aplicación así, deberías utilizar una herramienta recomendada por la tecnología en concreto, como Bower y GruntJS. Deberías desarrollar tu aplicación frontend de forma separada al backend de Symfony (incluso separando los repositorios si quieres).

#### Algunos tips

*   Puedes usar Assetic para [minimizar archivos CSS y JavaScript con Uglify](http://diego.com.es/comprimir-archivos-css-js-con-uglifycss-y-uglifyjs).
*   [Comprimir imágenes con Assetic](http://symfony.com/doc/current/cookbook/assetic/jpeg_optimize.html) para reducir su tamaño antes de enviarlas al usuario.

### 10\. Tests

Existen dos tipos de tests. **Unit testing** permite testear el input y output de funciones específicas. **Functional testing** funciona como un navegador al que le vas mostrando páginas, haciendo click en links, rellenar formularios, etc.

#### Unit Tests

Los Unit tests se utilizan para testear la "**business logic**", la cual ha de estar en clases que son independientes de **Symfony**. Por esta razón, Symfony no recomienda ningún tipo de herramienta concreta en este campo. Sin embargo, las herramientas más populares son **PHPUnit** y **PHPSpec**.

#### Functional Tests

Crear buenos functional tests puede resultar complicado y algunos desarrolladores se saltan este paso por completo, lo cual es un error. Definiendo algunos functional tests puedes detectar errores antes de hacer deploy:

**Define functional tests que por lo menos comprueben que las páginas de tu aplicación cargan correctamente.**

Un functional tests puede ser tan sencillo como esto:

```
// tests/AppBundle/ApplicationAvailabilityFunctionalTest.php
namespace Tests\AppBundle;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class ApplicationAvailabilityFunctionalTest extends WebTestCase
{
    /**
     * @dataProvider urlProvider
     */
    public function testPageIsSuccessful($url)
    {
        $client = self::createClient();
        $client->request('GET', $url);

        $this->assertTrue($client->getResponse()->isSuccessful());
    }

    public function urlProvider()
    {
        return array(
            array('/'),
            array('/posts'),
            array('/post/fixture-post-1'),
            array('/blog/category/fixture-category'),
            array('/archives'),
            // ...
        );
    }
}
```

Este código comprueba que todas las URL dadas cargan satisfactoriamente, lo que significa que su código de estado HTTP está entre 200 y 299\. Esto puede parecer que no es tan útil, pero viendo el poco esfuerzo que hace falta, merece la pena tenerlo en la aplicación.

En el software de computación, este tipo de test es llamado [smoke testing](https://en.wikipedia.org/wiki/Smoke_testing_(software)), y consite en "tests preliminares para revelar simples fallos suficientemente grandes como para frenar el lanzamiento de una aplicación".

#### Hardcore URLs en functional tests

¿Por qué no se ha utilizado el service URL generator?

**Las Hardcore URLs se utilizan en functional tests en lugar de utilizar el URL generator.**

Considera el siguiente test funcional que utiliza el **router service** para generar la URL de la página testeada:

```
public function testBlogArchives()
{
    $client = self::createClient();
    $url = $client->getContainer()->get('router')->generate('blog_archives');
    $client->request('GET', $url);

    // ...
}
```

Este código funcionaría, pero tiene una importante desventaja. Si el desarrollador cambia erróneamente el directorio de la route **blog_archives**, el test también funcionará, pero la URL antigua no funcionará. Esto significa que cualquier bookmark para esa URL se romperá y perderás page ranking en buscadores.

#### Testing JavaScript

El cliente de functional tests por defecto funciona muy bien, pero no puede usarse para testear el comportamiento de JavaScript. Si necesitas esto, considera utilizar la librería Mink desde dentro de PHPUnit.

Si tienes mucho JavaScript, es mejor utilizar herramientas propias para el testeo de JavaScript.

Tip: considera utilizar librerías para generar datos para los fixtures con [Faker](https://github.com/fzaninotto/Faker) y [Alice](https://github.com/nelmio/alice).