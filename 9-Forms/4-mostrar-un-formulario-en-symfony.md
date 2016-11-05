Una vez creado el **formulario**, el siguiente paso es **renderizarlo**. Esto se consigue pasando un objeto especial **form view** a la template (_$form->createView()_). Ya sabemos la forma simple y automática de mostrar todos los campos del formulario:

```
{# app/Resources/views/default/new.html.twig #}
{{ form_start(form) }}
{{ form_widget(form) }}
{{ form_end(form) }}
```

A esta sencilla forma podemos añadirle un poco más de flexibilidad:

```
{# app/Resources/views/default/new.html.twig #}
{{ form_start(form) }}
    {{ form_errors(form) }}

    {{ form_row(form.task) }}
    {{ form_row(form.dueDate) }}
{{ form_end(form) }}
```

Ya hemos visto lo que hacen las funciones _form_start()_ y _form_end()_, vamos a ver las otras dos:

*   _form_errors(form)_. Renderiza los errores del formulario (los errores de campos específicos se muestran en cada campo).
*   _form_row(form.dueDate)_. Renderiza la etiqueta, cualquier error, y el widget HTML del formulario para el campo dado, por defecto dentro de un elemento _div_.

La mayoría del trabajo está hecho con el helper _form_row_. Con el form theming se puede customizar de muchas formas la salida del _form_row_.

### Renderizar cada campo manualmente

El helper _form_row_ es muy útil porque puedes renderizar rápidamente cada campo de un formulario (y el marcado para el _row_ puede customizarse también). Pero también puedes renderizar cada campo de forma manual. El siguiente ejemplo muestra lo mismo que cuando hemos empleado el helper _form_row_.

```
{{ form_start(form) }}
    {{ form_errors(form) }}

    <div>
        {{ form_label(form.task) }}
        {{ form_errors(form.task) }}
        {{ form_widget(form.task) }}
    </div>

    <div>
        {{ form_label(form.dueDate) }}
        {{ form_errors(form.dueDate) }}
        {{ form_widget(form.dueDate) }}
    </div>

    <div>
        {{ form_widget(form.save) }}
    </div>

{{ form_end(form) }}
```

Si la **etiqueta** autogenerada no es correcta, puedes especificarla:

```
{{ form_label(form.task, 'Task Description') }}
```

Algunos **field types** tienen opciones adicionales de renderizado que pueden pasarse al **widget**. Estas opciones están documentadas en cada type, pero una opción frecuente es _attr_, que permite modificar **atributos del elemento form**. El código siguiente añadiría la clase **task_field** al campo de texto renderizado:

```
{{ form_widget(form.task, {'attr': {'class': 'task_field'}}) }}
```

Si necesitas renderizar campos de formulario manualmente puedes acceder a valores individuales para campos como **id**, **name** y **label**. Por ejemplo para obtener el **id**:

```
{{ form.task.vars.id }}
```

Para obtener el valor empleado por el nombre del atributo del campo del formulario se ha se emplear el valor _full_name_:

```
{{ form.task.vars.full_name }}
```