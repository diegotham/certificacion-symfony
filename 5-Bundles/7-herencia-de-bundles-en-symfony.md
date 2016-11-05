Cuando se trabaja con **bundles de terceros**, es probable que te encuentres en una situación en la que quieres sobreescribir un archivo de ese bundle con un archivo de uno de tus propios bundles. **Symfony** proporciona una forma conveniente de sobreescribir cosas como controllers, templates, y otros archivos en el directorio _/Resources_. 

Por ejemplo queremos extender **FOSUserBundle**, ya sea el _layout.html.twig_ o alguno de sus _controladores_. Suponemos que tu bundle se llama **UserBundle** y ahí están los archivos que sobreescribirán al bundle original. Pondremos al FOSUserBundle como padre de tu bundle:

```
// src/UserBundle/UserBundle.php
namespace Acme\UserBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class UserBundle extends Bundle
{
    public function getParent()
    {
        return 'FOSUserBundle';
    }
}
```

Ahora tan sólo creando archivos con el mismo nombre ya puedes sobreescribir diferentes partes del **FOSUserBundle**. A pesar del nombre _getParent_, no hay ninguna relación de herencia entre los controladores, simplemente es una forma de **sobreescribir un bundle existente**.

### Controllers

Por ejemplo ahora queremos añadir alguna funcionalidad a _registerAction_ del **RegistrationController** que está dentro de **FOSUserBundle**. Para hacerlo, simplemente crea tu propio archivo _RegistrationController.php_, sobreescribe el método original del bundle, y cambia su funcionalidad:

```
// src/UserBundle/Controller/RegistrationController.php
namespace UserBundle\Controller;

use FOS\UserBundle\Controller\RegistrationController as BaseController;

class RegistrationController extends BaseController
{
    public function registerAction()
    {
        $response = parent::registerAction();

        // ... hacer cosas personalizadas
        return $response;
    }
}
```

Dependiendo en cuánto necesitas cambiar ese comportamiento, podrías llamar a _parent::resgisterAction()_ o reemplazar completamente su lógica con la tuya.

Sobreescribir los bundles de esta forma sólo funciona si el bundle hace referencia al controller empleando la sintaxis estándar _FOSUserBundle:Registration:register_ en routes y templates. Esta es la mejor práctica.

### Routing

El routing nunca se importa de forma automática en **Symfony**. Si quieres incluir los routes de cualquier bundle, debes importarlos manualmente desde algún lado a tu aplicación (por ejemplo en _app/config/routing.yml_). La forma más fácil de sobreescribir el routing de un bundle es no importarlo nunca. En lugar de importarlo, simplemente copia el archivo de routing en tu aplicación y modifícalo.

### Services y configuración

Para **sobreescribir o extender un service** hay dos opciones. Primero, puedes establecer el parámetro del nombre de clase del service con el nombre de tu propia clase en app/config/config.yml. Esto sólo es posible si el nombre de la clase se define como parámetro en la configuración del service del bundle que contiene el service. Por ejemplo, para sobreescribir la clase empleada en el service **translator** de Symfony, sobreescribirías el parámetro _translator.class_. Conociendo exactamente qué parámetro sobreescribir puede llevar su tiempo. Para el translator, el parámetro es definido y empleado en el archivo _Resources/config/translation.xml_ en el **FrameworkBundle**.

```
# app/config/config.yml
parameters:
    translator.class: Acme\HelloBundle\Translation\Translator
```

Si la clase no está disponible como parámetro, hay que asegurarse que la clase siempre se sobreescribe cuando se emplea tu bundle o si necesitas modificar algo más allá del nombre de clase, deberías emplear un compiler pass:

```
// src/Acme/DemoBundle/DependencyInjection/Compiler/OverrideServiceCompilerPass.php
namespace Acme\DemoBundle\DependencyInjection\Compiler;

use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class OverrideServiceCompilerPass implements CompilerPassInterface
{
    public function process(ContainerBuilder $container)
    {
        $definition = $container->getDefinition('original-service-id');
        $definition->setClass('Acme\DemoBundle\YourService');
    }
}
```

En este ejemplo obtienes la **definición del service original** y estableces su clase con tu propia clase. 

Si quieres hacer algo que va mas allá de simplemente sobreescribir la clase, como añadir una llamada a un método, sólo puedes emplear el método del compiler pass.

### Entidades

Debido a la forma en que funciona Doctrine, no es posible sobreescribir el mapeo de entidades de un bundle. Sin embargo, si un bundle proporciona una superclase mapeada (como la entidad User en el FOSUserBundle) se pueden sobreescribir atributos y asociaciones. Puedes ver más sobre la herencia en Doctrine [aquí](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/inheritance-mapping.html#overrides).

### Formularios

Los form types son referenciados con su nombre de clase fully-qualified:

```
$builder->add('name', CustomType::class);
```

Esto significa que no puedes sobreescribirlo creando una subclase de CustomType y registrarlo como server, y etiquetarlo con form.type (aunque esto se pudiera hacer en versiones anteriores).

En cambio, deberías emplear un form type extension para modificar el form type existente. Para más información, puedes leer sobre [como crear un form type extension](http://symfony.com/doc/current/cookbook/form/create_form_type_extension.html).

### Validación de metadatos

Symfony carga todos los archivos de validación de configuración de todos los bundles y los combina en un árbol de validación de metadatos. Esto significa que puedes añadir nuevas restricciones a una propiedad, pero no puedes sobreescribirlas. Para solucionar esto, el bundle de terceros necesita tener configuración para [grupos de validación](http://symfony.com/doc/current/book/validation.html#book-validation-validation-groups). Por ejemplo, el **FOSUserBundle** tiene su configuración. Para crear tu propia validación, añade las restricciones a un nuevo grupo de validación:

```
# src/Acme/UserBundle/Resources/config/validation.yml
FOS\UserBundle\Model\User:
    properties:
        plainPassword:
            - NotBlank:
                groups: [AcmeValidation]
            - Length:
                min: 6
                minMessage: fos_user.password.short
                groups: [AcmeValidation]
```

Ahora, actualiza la configuración del **FOSUserBundle**, de forma que utilice tus grupos de validación en lugar de los originales.

### Traducciones

Los traducciones no están relacionadas con los bundles, sino con los **dominios**. Eso significa que puedes sobreescribir las traducciónes desde cualquier archivo de traducción, siempre que esté en el [dominio correcto](http://symfony.com/doc/current/components/translation/introduction.html#using-message-domains). 

El **último archivo de traducción** siempre gana. Eso significa que necesitas asegurar que el bundle que contiene tus traducciones se carga después de cualquier bundle al cual le estás sobreescribiendo las traducciones. Esto se hace en el **AppKernel**. 

Los archivos de traducción no son conscientes de la **herencia entre bundles**. Si quieres sobreescribir traducciones del bundle padre, asegúrate que el bundle padre se carga antes que el bundle hijo en la clase AppKernel.

El archivo que siempre gana es el que está en _app/Resources/translations_, ya que esos archivos siempre se cargan los últimos.