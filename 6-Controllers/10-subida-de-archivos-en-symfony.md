Para explicar **cómo subir archivos en una aplicación Symfony**, vamos a poner el ejemplo de subir un currículum en pdf, con un model y un formulario muy sencillos para centrarnos en el controller.

Tenemos una entidad Usuario en la aplicación, y queremos añadir un PDF con el currículum para cada usuario.

```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

class User
{
    // ...

    /**
     * @ORM\Column(type="string")
     *
     * @Assert\NotBlank(message="Por favor inserta tu currículum")
     * @Assert\File(mimeTypes={"aplication/pdf"})
     */
    private $curriculum;

    public function getCurriculum()
    {
        return $this->curriculum;
    }

    public function setCurriculum($curriculum)
    {
        $this->curriculum = $curriculum;

        return $this;
    }
}
```

Nótese que el tipo de columna en **currículum** es _string_ en lugar de _binary_ o _blob_ ya que sólo guarda el nombre del **archivo PDF** en lugar de los contenidos del archivo.

Ahora añadimos el campo curriculum en el formulario que maneja la entidad User:

```
// src/AppBundle/Form/UserType.php
namespace AppBundle;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;
use Symfony\Component\Form\Extension\Core\Type\FileType;

class UserType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            // ...
            ->add('curriculum', FileType::class, array('label' => 'Currículum (PDF file)'))
            // ...
        ;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'AppBundle\Entity\User',
        ));
    }
}
```

Actualizamos la plantilla que renderiza el formulario para mostrar el nuevo campo curriculum (el código exacto de la template a añadir depende en el método empleado por tu aplicación para customizar el form rendering):

```
{# app/Resources/views/user/cv.html.twig #}
<h1>Añadir currículum de usuario</h1>

{{ form_start() }}
    {# ... #}
    {{ form_row(form.curriculum) }}
{{ form_end() }}
```

Finalmente, llegamos a la parte del controller:

```
// src/AppBundle/Controller/UserController.php
namespace AppBundle\Controller;

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;
use AppBundle\Entity\User;
use AppBundle\Form\UserType;

class UserController extends Controller
{
    /**
     * @Route("/user/nuevo", name="add_new_cv")
     */
    public function newAction(Request $request)
    {
        $usuario = new User();
        $form = $this->createForm(UserType::class, $usuario);
        $form->handleRequest($request);

        if($form->isValid())
        {
            // La variable $file guardará el PDF subido
            /** @var Symfony\Component\HttpFoundation\File\UploadedFile $file */
            $file = $usuario->getCurriculum();

            // Generar un numbre único para el archivo antes de guardarlo
            $fileName = md5(uniqid()).'.'.$file->guessExtension();

            // Mover el archivo al directorio donde se guardan los curriculums
            $cvDir = $this->container->getparameter('kernel.root_dir').'/../web/uploads/cv';
            $file->move($cvDir, $fileName);

            // Actualizar la propiedad curriculum para guardar el nombre de archivo PDF
            // en lugar de sus contenidos
            $usuario->setCurriculum($fileName);

            // ... persist la variable $usuario o cualquier otra tarea

            return $this->redirect($this->generateUrl('app_product_list'));
        }

        return $this->render('user/cv.html.twig', array(
            'form' => $form->createView(),
        ));
    }
}
```

Hay una serie de cosas a tener en cuenta en el controller:

*   Cuando se sube el archivo, la propiedad _curriculum_ contiene el contenido del **archivo PDF**. Ya que esta propiedad sólo guarda el nombre del archivo, debes establecer su nuevo nombre antes de persistir los cambios en la entidad.
*   En aplicaciones Symfony, los archivos subidos son objetos de la clase **UploadedFile**, que proporciona métodos para las operaciones más comunes cuando se trata con archivos subidos.
*   Una regla de seguridad fundamental es nunca confiar en archivos subidos por un usuario. La clase **Uploaded** proporciona métodos para obtener la extensión original (_getExtension()_), el tamaño original (_getClientSize()_) y el nombre de archivo original (_getClientOriginalName()_). Sin embargo, estos no son considerados muy seguros porque un atacante podría alterar esa información. Esa es la razón por la que siempre es mejor generar un nombre único y emplear el método guessExtension() para permitir que Symfony adivine la extensión correcta de acuerdo al MIME type.
*   La clase **UploadedFile** también proporciona un método _move()_ para guardar el archivo en el directorio que se quiera. Definir este directorio como una opción de configuración es una buena práctica que simplifica el código: $this->container->getParameter('cv_dir').

Ahora puedes utilizar el siguiente enlace al currículum en PDF del usuario:

```
<a href="{{ asset('uploads/cv/' ~ user.curriculum) }}">Ver currículum en PDF</a>
```