Los **componentes de Symfony** son un conjunto de **librerías PHP reusables**. Implementan características comunes para desarrollar aplicaciones en PHP. Se puede usar cualquiera de estas librerías en aplicaciones propias independientemente del framework Symfony, pues no tienen **dependencias** obligatorias.

Para instalarlos se puede utilizar **composer**:

```
composer require symfony/component
```

Lista de componentes:

*   [Asset](http://symfony.com/components/Asset): Gestiona la generación de URLs y el control de versiones de los _**assets**_ como CSS, JavaScript e imágenes.
*   [BrowserKit](http://symfony.com/components/BrowserKit): Simula el comportamiento de un **navegador**.
*   [ClassLoader](http://symfony.com/components/ClassLoader): Carga las **clases** de tu proyecto automáticamente si siguen algunos de los estándares PHP.
*   [Config](http://symfony.com/components/Config): Ayuda a encontrar, cargar, combinar, autorellenar y validar **valores de configuración**.
*   [Console](http://symfony.com/components/Console): facilita la creación de interfaces de línea de **comandos**.
*   [CssSelector](http://symfony.com/components/CssSelector): convierte selectores **CSS** a expresiones **XPath**.
*   [Debug](http://symfony.com/components/Debug): proporciona herramientas para **depurar** fácilmente código PHP.
*   [DependencyInjection](http://symfony.com/components/DependencyInjection): permite estandarizar y centralizar la forma en que los **objetos** se construyen en tu aplicación.
*   [DomCrawler](http://symfony.com/components/DomCrawler): facilita la navegación **DOM** para documentos **HTML** y **XML**.
*   [EventDispatcher](http://symfony.com/components/EventDispatcher): implementa el patrón **Mediator** de forma simple y efectiva para hacer proyectos **extensibles**.
*   [ExpressionLanguage](http://symfony.com/components/ExpressionLanguage): proporciona un motor que puede **compilar y evaluar expresiones**.
*   [FileSystem](http://symfony.com/components/Filesystem): proporciona utilidades básicas para el **sistema de archivos**.
*   [Finder](http://symfony.com/components/Finder): **encuentra** archivos y directorios con una interfaz intuitiva.
*   [Form](http://symfony.com/components/Form): proporciona herramientas para crear, procesar y reusar **formulatios HTML**.
*   [HttpFoundation](http://symfony.com/components/HttpFoundation): define una forma orientada a objetos de manejar los **mensajes HTTP**.
*   [HttpKernel](http://symfony.com/components/HttpKernel): proporciona los cimientos para crear frameworks **HTTP-based**.
*   [Intl](http://symfony.com/components/Intl): proporciona código para manejar los casos en los que falta la **extensión intl**.
*   [OptionsResolver](http://symfony.com/components/OptionsResolver): ayuda a **configurar objetos** con arrays de opciones.
*   [Process](http://symfony.com/components/Process): ejecuta **comandos en subprocesos**.
*   [PropertyAccess](http://symfony.com/components/PropertyAccess): proporciona la funcionalidad de leer y escribir desde/a un **objeto o array** mediante un _string_.
*   [Routing](http://symfony.com/components/Routing): asigna una petición HTTP a un conjunto de variables de configuración.
*   [Security](http://symfony.com/components/Security): proporciona una infraestructura para sistemas de autorización sofisticados.
*   [Serializer](http://symfony.com/components/Serializer): convierte objetos en un formato específico (XML, JSON, YAML, ...) y viceversa.
*   [Stopwatch](http://symfony.com/components/Stopwatch): proporciona una manera de perfilar código.
*   [Templating](http://symfony.com/components/Templating): proporciona las herramientas necesarias para construir cualquier tipo de sistema de plantillas.
*   [Translation](http://symfony.com/components/Translation): proporciona herramientas para internacionalizar tu aplicación.
*   [Validator](http://symfony.com/components/Validator): proporciona herramientas para validar clases.
*   [VarDumper](http://symfony.com/components/VarDumper): su objetivo es reemplazar la **función** _var_dump()_ por una más moderna y sofisticada _dump()_.
*   [Yaml](http://symfony.com/components/Yaml): carga y vuelca archivos YAML.