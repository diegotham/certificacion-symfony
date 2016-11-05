Para el **manejo de la subida de archivos** se emplea el campo de formulario FileType, que representa un campo _input file_. Este campo se compone de las siguientes especificaciones:

### Opciones

#### multiple

Type: **boolean**. Default: **false**

Si se establece en true, el usuario podrá subir múltiples archivos a la vez.

### Opciones sobreescritas

#### compound

Type: **boolean**. Default: **false**

Esta opción especifica si el type contiene child types o no. Esta opción se maneja internamente para types incorporados, por lo que no hay necesidad de configurarla explícitamente.

#### data_class

Type: **string**. Default: **File**

Esta opción establece el mapeador de datos apropiado para archivos a usar por el type.

#### empty_data

Type: **mixed**. Default: **null**

Esta opción determina que valor devolverá el campo cuando el valor enviado esté vacío.

### Opciones heredadas

Estas opciones se heredan de **[FormType](http://symfony.com/doc/current/reference/forms/types/form.html)**: disabled, error_bubbling, error_mapping, label, label_attr.

### Ejemplo de subida de archivo en Symfony

Suponemos que tenemos una **clase User** donde el usuario puede subir su **currículum en pdf**. La definición de **UserType** es:

```
// src/AppBundle/Form/UserType.php

class UserType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('name', TextType::class, array('label' => 'Nombre'))
            ->add('curriculum', FileType::class, array('label' => 'Currículum (PDF file)'))
            ->add('save', SubmitType::class, array('label' => 'Crear curriculum'))
        ;
    }
```

Cuando el formulario se envía, el campo _curriculum_ será una instancia de **UploadedFile**. Puede usarse para mover el archivo _curriculum_ a un directorio:

```
public function uploadAction()
{
    // ...

    if ($form->isValid()) {
        $someNewFilename = ...

        $form['attachment']->getData()->move($dir, $someNewFilename);

        // ...
    }

    // ...
}
```

El método _move()_ recibe un **directorio** y un **nombre de archivo** como argumentos. Puedes generar el nombre del archivo así:

```
// utilizamos el nombre original del archivo
$file->move($dir, $file->getClientOriginalName());

// obtener un nombre aleatorio e intentar adivinar la extensión (es más seguro)
$extension = $file->guessExtension();
if (!$extension) {
    // la extensión no se ha podido adivinar
    $extension = 'bin';
}
$file->move($dir, rand(1, 99999).'.'.$extension);
```

Utilizar el nombre original con _getClientOriginalName()_ no es seguro y puede manipularse por el usuario. Además, pueden contener caracteres que no están permitidos en nombres de archivos. Se debe **sanitizar el nombre antes de usarlo** directamente.

Una forma de hacer esto y más de forma automática es empleando [Doctrine para la subida de archivos](http://diego.com.es/subida-de-archivos-en-doctrine).