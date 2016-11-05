Una instancia de Crawler es devuelta cada vez que se hace un request con Client. El objeto Crawler permite recorrer documentos HTML o XML, seleccionar nodos, y encontrar enlaces y fomularios.

#### Traversing

Como **jQuery**, el Crawler tiene métodos para **recorrer el DOM** de un **documento HTML/XML**. El siguiente ejemplo encuentra todos los elementos _input[type=submit]_, obtiene el último en la página y selecciona su elemento padre inmediato:

```
$newCrawler = $crawler->filter('input[type=submit]')
    ->last()
    ->parents()
    ->first()
;
```

Hay muchos otros métodos disponibles:

_filter('h1.title')_. Nodos que coinciden con un selector CSS.
_filterXpath('h1')_. Nodos que coinciden con una expresión XPath.
_eq(1)_. Nodo para el index especificado.
_first()_. Primer nodo.
_last()_. Ultimo nodo.
_siblings()_. Hermanos.
_nextAll()_. Todos los hermanos siguientes.
_previousAll()_. Todos los hermanos anteriores.
_parents()_. Devuelve los nodo madre.
_children()_. Devuelve los nodos hijo.
_reduce($lambda)_. Nodos para los cuales el callable no devuelve false.

Ya que cada uno de estos métodos devuelve una instancia **Crawler** nueva, puedes especificar la selección del nodo encadenando los métodos:

```
$crawler
    ->filter('h1')
    ->reduce(function ($node, $i) {
        if (!$node->getAttribute('class')) {
            return false;
        }
    })
    ->first()
;
```

Puedes usar la función _count()_ para obtener el número de nodos guardados en un **Crawler**: _count($crawler)_.

#### Extraer información

El **Crawler** puede extraer información de los nodos:

```
// Devuelve el valor del atributo del primer nodo
$crawler->attr('class');

// Devuelve el valor del nodo del primer nodo
$crawler->text();

// Extrae un array de atributos de todos los nodos
// (_text devuelve el valor del nodo)
// devuelve un array para cada elemento del crawler,
// cada uno con su valor y href
$info = $crawler->extract(array('_text', 'href'));

// Ejecuta un lambda para cada nodo y devuelve un array de resultados
$data = $crawler->each(function ($node, $i) {
    return $node->attr('href');
});
```

#### Enlaces

Para seleccionar enlaces puedes usar los métodos de recorrido anteriores o el shorcut _selectLink()_:

```
$crawler->selectLink('Click here');
```

Esto selecciona todos los enlaces que contienen el texto dado, o imágenes clickables en las cuales el atributo _alt_ contiene el texto dado. Como los otros métodos de filtrado, esto devuelve otro objeto **Crawler**.

Una vez que has seleccionado un enlace, tienes acceso a un objeto especial **Link**, que tiene métodos de ayuda específicos para enlaces (como _getMethod()_ y _getUri()_). Para hacer click en el link, utiliza el método del cliente _click()_ y pásale un objeto **Link**:

```
$link = $crawler->selectLink('Click here')->link();
$client->click($link);
```

#### Formularios

Los formularios pueden ser seleccionados empleando sus botones, que pueden seleccionarse con el método _selectButton()_, como los enlaces:

```
$buttonCrawlerNode = $crawler->selectButton('submit');
```

Fíjate que seleccionas botones y no formularios ya que un formulario puede tener varios botones. Si recorres la API, ten en cuenta que debes buscar un botón.

El método _selectButton()_ puede seleccionar etiquetas _button_ y enviar etiquetas _input_. Utiliza varias partes de los botones para encontrarlas:

*   El valor del atributo _value_.
*   El valor de los atributos _id_ o _alt_ para imágenes.
*   El valor de los atributos _id_ o _name_ para etiquetas _button_.

Una vez que tienes un Crawler representando un botón, puedes llamar al método _form()_ para obtener una instancia Form del formulario que envuelve el nodo del botón:

```
$form = $buttonCrawlerNode->form();
```

Cuando llamamos al método _form()_, podemos también pasar un array de valores de campos que sobreescriben a los que están por defecto:

```
$form = $buttonCrawlerNode->form(array(
    'name'              => 'Fabien',
    'my_form[subject]'  => 'Symfony rocks!',
));
```

Si quieres **simular un método HTTP específico para el formulario**, pásalo como segundo argumento:

```
$form = $buttonCrawlerNode->form(array(), 'DELETE');
```

**Client** puede crear instancias de **Form**:

```
$client->submit($form);
```

Los valores de los campos pueden también pasarse en el segundo argumento del método _submit()_:

```
$client->submit($form, array(
    'name'              => 'Fabien',
    'my_form[subject]'  => 'Symfony rocks!',
));
```

Para situaciones más complejas, utiliza la instancia Form como un array para establecer el valor de cada campo individualmente:

```
// Cambiar el valor de un campo
$form['name'] = 'Fabien';
$form['my_form[subject]'] = 'Symfony rocks!';
```

Hay también una API para manipular los valores de los campos según sus tipos:

```
// Selecciona una opción o un radio
$form['country']->select('France');

// Selecciona una checkbox
$form['like_symfony']->tick();

// Sube un archivo
$form['photo']->upload('/path/to/lucas.jpg');
```

Puedes también [seleccionar valores inválidos](http://symfony.com/doc/current/components/dom_crawler.html#components-dom-crawler-invalid).

Puedes obtener los valores que serán enviados llamando al método _getValues()_ en el objeto **Form**. Los archivos subidos están disponibles en un array separado devuelto por _getFiles()_. Los métodos _getPhpValues()_ y _getPhpFiles()_ también devuelven los valores enviados, pero en formato PHP (convierte los keys con corchetes en arrays de PHP).

Añadir y eliminar formularios a una Collection

Si empleas una [Collection de formularios](http://symfony.com/doc/current/cookbook/form/form_collections.html), no puedes añadir campos a un formulario existente con _$form['task[tags][0][name]'] = 'foo';_. Esto devuelve un error _Unreachable field "..."_ porque _$form_ sólo puede usarse para establecer valores de campos existentes. Para añadir nuevos campos, tienes que añadir los valores al array:

```
// Obtiene el formulario
$form = $crawler->filter('button')->form();

// Obtiene los valores
$values = $form->getPhpValues();

// Añade campos a los valores
$values['task']['tag'][0]['name'] = 'foo';
$values['task']['tag'][1]['name'] = 'bar';

// Envía el formualrio con los campos nuevos y existentes
$crawler = $this->client->request($form->getMethod(), $form->getUri(), $values,
    $form->getPhpFiles());

// Las dos etiquetas se han añadido a la collection
$this->assertEquals(2, $crawler->filter('ul.tags > li')->count());
```

Donde task[tags][0][name] es el nombre de un campo creado con JavaScript.

Puedes eliminar un campo existente, por ejemplo una etiqueta:

```
// Obtener los valores del formualrio
$values = $form->getPhpValues();

// Eliminar la primera etiqueta
unset($values['task']['tags'][0]);

// Enviar los datos
$crawler = $client->request($form->getMethod(), $form->getUri(),
    $values, $form->getPhpFiles());

// La etiqueta se ha eliminado
$this->assertEquals(0, $crawler->filter('ul.tags > li')->count());
```