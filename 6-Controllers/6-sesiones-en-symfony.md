**Indice de contenido**

1.  Introducción
2.  La session API
3.  Manejo de datos de sesiones
4.  Atributos

### 1\. Introducción

El componente **Symfony HttpFoundation** tiene un potente y flexible subsistema de sesiones designado para proporcionar el manejo de sesiones a través de una **interface orientada a objetos** utilizando drivers de almacenamiento de sesiones.

Las sesiones son utilizadas a través de la implementación de [Session](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html) de la interface [SessionInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/SessionInterface.html).

Hay que asegurarse de no tener iniciada la **sesión de PHP** antes de utilizar la clase Session. Si tienes un sistema de legado de sesiones que inicia la sesión, lee [este artículo](http://symfony.com/doc/current/components/http_foundation/session_php_bridge.html).

```
use Symfony\Component\HttpFoundation\Session\Session;

$session = new Session();
$session->start();

// Establecer y obtener atributos de sesión
$session->set('name', 'Drak');
$session->get('name');

// Establecer mensajes flash
$session->getFlashBag()->add('notice', 'Profile updated');

// Devolver mensajes
foreach ($session->getFlashBag()->get('notice', array()) as $message) {
    echo '<div class="flash-notice">'.$message.'</div>';
}
```

Las **sesiones de Symfony** están diseñadas para reemplazar algunas funciones nativas de PHP. Las aplicaciones deberían evitar utilizar _session_start()_, _session_regenerate_id()_, _session_id()_, _session_name()_ y _session_destroy()_ y en su lugar utilizar la **API de Symfony**.

Es recomentable iniciar una sesión explícitamente, pero una sesión de iniciará bajo demanda, es decir, si se hace cualquier sessión request para leer o escribir datos en sesiones. Es problable que en lugar de utilizar el código de _new Session()_ tengas que acceder a la sesión directamente desde el request:

```
$session = $request->getSession();
```

Ya que si no te puede salir el siguiente error: **Failed to start the session: already started by PHP. 500 Internal Server Error - RuntimeException**. 

Las sesiones de Symfony son incompatibles con la directiva _php.ini_ **session.auto_start = 1**. Esta directiva debe desactivarse en _php.ini_, en las directivas del servidor web o en _.htaccess_.

### 2\. La session API

La clase **Session** implementa **SessionInterface**. 

Session tiene una simple API dividida en varios grupos, como sigue:

#### Session Workflow

*   [start()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_start). Inicia la sesión. Nunca utilizar _session_start()_.
*   [migrate()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_migrate). Regenera la session ID. Nunca utilizar _session_regenerate_id()_. 
*   [invalidate()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_invalidate). Limpia los datos de sesiones y regenera la session ID. Nunca utilizar _session_destroy()_. 
*   [getId()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_getId). Obtiene la session ID. Nunca utilizar _session_id()_. 
*   [setId()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_setId). Establece la session ID. Nunca utilizar _session_id()_.
*   [getName()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_getName). Obtiene el nombre de la sesión. Nunca utilizar _session_name()_.
*   [setName()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_setName). Establece el nombre de la sesión. Nunca utilizar _session_name()_.

#### Session Attributes

*   [set()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_set). Establece un atributo con una _key_. 
*   [get()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_get). Obtiene un atributo por su _key_.
*   [all()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_all). Obtiene todos los atributos como array _key => value_.
*   [has()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_has). Devuelve true si el atributo existe.
*   [replace()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_replace). Establece múltiples atributos de vez. Toma un [array asociativo](http://diego.com.es/arrays-asociativos-en-php) y establece cada pareja _key => value_.
*   [remove()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_remove). Elimina un atributo por su key.
*   [clear()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_clear). Limpia todos los atributos.

Los atributos se guardan internamente en una bolsa (**Bag**), un **objeto PHP** que actúa como un array. Existen unos pocos métodos para la administración de bolsas:

*   [registerBag()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_registerBag). Registra una [SessionBagInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/SessionBagInterface.html).
*   [getBag()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_getBag). Obtiene una SessionBagInterface por su nombre de tag.
*   [getFlashBag()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_getFlashBag). Obtiene una [FlashBagInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Flash/FlashBagInterface.html). Es un simple shortcut por conveniencia.

#### Session Metadata

*   [getMetadataBag()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Session.html#method_getMetadataBag). Obtiene la [MetadataBag](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Storage/MetadataBag.html) que contiene información sobre la sesión.

### 3. Manejo de datos de sesiones

La administración de [sesiones de PHP](http://diego.com.es/sesiones-en-php) requiere el uso de la superglobal **$_SESSION**, sin embargo, interfiere de alguna forma con el testeo del código y la encapsulación en el paradigma **OOP** (Object Oriented Programming). Para solucionar esto, Symfony utiliza bolsas de sesiones enlazadas a la sesión para encapsular conjuntos de datos de atributos o mensajes flash.

Este enfoque también mitiga la profanación de **namespaces** dentro de la superglobal $_SESSION ya que cada bolsa guarda todos sus datos bajo un único namespace. Esto permite a Symfony coexistir con otras aplicaciones o librerías que pudieran utilizar la superglobal $_SESSION y todos los datos permanecen completamente compatibles con la administración de sesiones de Symfony.

**Symfony** proporciona dos tipos de **bolsas de almacenamiento**, con dos implementaciones diferentes. Todo está construido en base a interfaces por lo que puedes extender o crear tus propias bolsas si es necesario.

[SessionBagInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/SessionBagInterface.html) tiene la siguiente **API** que es principalmente para asuntos internos:

*   [getStorageKey()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/SessionBagInterface.html#method_getStorageKey). Devuelve el key bajo el que la bolsa guardará su array en $_SESSION. Generalmente este valor se puede dejar a su valor por defecto ya que es para uso interno.
*   [initialize()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/SessionBagInterface.html#method_initialize). Esta función es llamada internamente por las clases de almacenamiento de sesión de Symfony para enlazar datos de bolsas a la sesión.
*   [getName()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/SessionBagInterface.html#method_getName). Devuelve el nombre de la bolsa de sesión.

### 4\. Atributos

El objetivo de las bolsas implementando [AttributeBagInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html) es para manejar el almacenamiento de atributos de sesión. Esto podría incluir cosas como **user ID**, y ajustes de "recordarme" y otros basados en información estática.

*   [AttributeBag](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBag.html). Esta es la implementación por defecto **estándar**.
*   [NamespacedAttributeBag](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/NamespacedAttributeBag.html). Esta implementación permite a los atributos ser guardados en un **namespace** estructurado.

Cualquier sistema de almacenamiento _key-value_ está limitado a la complejidad de los datos guardados ya que cada key debe ser única. Puedes conseguir **namespacing** introduciendo una convención de nombres a los _keys_ y así diferentes partes de tu aplicación pueden operar sin chocar. Por ejemplo _module1.foo_ y _module2.foo_. Sin embargo, a veces no es muy práctico cuando los datos de los atributos están en un array, por ejemplo un **conjunto de tokens**. En este caso, administrar el array se convierte en una carga porque tienes que sacar el array, procesarlo y guardarlo de nuevo:

```
$tokens = array(
    'tokens' => array(
        'a' => 'a6c1e0b6',
        'b' => 'f4a7b1f3',
    ),
);
```

Procesar esto podría complicarse mucho rápidamente, incluso añadiendo simplemente un **token al array**:

```
$tokens = $session->get('tokens');
$tokens['c'] = $value;
$session->set('tokens', $tokens);
```

Con **namespacing** estructurado, el _key_ se puede traducir a la estructura del array como esta usando un carácter de namespaces (por defecto /):

```
$session->set('tokens/c', $value);
```

De esta forma puedes acceder fácilmente a un _key_ dentro del _array_ guardado directa y fácilmente:

[AttributeBagInterface](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html) tiene una API simple

*   [set()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html#method_set). Establece un atributo según una _key_.
*   [get()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html#method_get). Obtiene un atributo según una _key_.
*   [all()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html#method_all). Obtiene todos los atributos como un array de _key => value_.
*   [has()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html#method_has). Devuelve true si el atributo existe.
*   [replace()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html#method_replace). Establece múltiples atributos de vez: toma un array asociativo y establece cada pareja _key => value_.
*   [remove()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html#method_remove). Elimina un atributo por su key.
*   [clear()](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/Session/Attribute/AttributeBagInterface.html#method_clear). Limpia la bolsa.