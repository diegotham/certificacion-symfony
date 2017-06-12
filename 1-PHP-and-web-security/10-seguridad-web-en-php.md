**Índice de contenido**

1\. Ataques XSS Cross-Site Scripting
2\. Ataques CSRF Cross-Site Request Forgeries
3\. Seguridad en sesiones
4\. Encriptación y contraseñas

## 1\. Ataques XSS Cross-Site Scripting

### ¿Qué es Cross-Site Scripting?

**XSS** ocurre cuando un atacante es capaz de inyectar un **script**, normalmente **Javascript**, en el _**output**_ de una aplicación web de forma que se ejecuta en el **navegador del cliente**. Los ataques se producen principalmente por **validar incorrectamente datos de usuario**, y se suelen inyectar mediante un **formulario web** o mediante un **enlace alterado**. 

Los desarrolladores muchas veces subestiman el potencial peligro que conlleva este tipo de ataques. Si un atacante puede inyectar Javascript en el output de una aplicación web y ejecutarlo, podrá **ejecutar cualquier código Javascript en el navegador de un usuario**. Algunos de los **objetivos que se quieren conseguir con ataques XSS** son: robar cookies y sesiones de usuarios, modificar el sitio web, realizar HTTP requests con la sesión del usuario, redireccionar a usuarios a sitios dañinos, atacar al navegador o instalar malware, reescribir o manipular extensiones de navegador, etc. 

Desde la **perspectiva del navegador**, el script es originado por la aplicación web por lo que se asume como fuente fiable. Esta es una de las causas que permiten que estos ataques se puedan llevar a cabo.

### Ejemplo de ataque XSS

Supongamos por ejemplo el siguiente **formulario**:

```
<form action="post.php" method="post">
    <input type="text" name="comment" value="">
    <input type="submit" name="submit" value="Submit">
<form>
```

En el formulatio hay una **caja de texto** para datos de entrada y un **botón de enviar**. Una vez que el formulario es enviado, enviará los datos a _post.php_ para procesarlos. Supongamos que lo único que se hará con los datos en post.php será mostrarlos con un echo:

```
echo $_POST['comment'];
```

Sin ningún tipo de **filtrado**, el atacante puede enviar el siguiente script a través del formulario, lo que generará un popup en el navegador con el mensaje "Hackeado":

```
<script>alert("hacked")</script>
```

El script anterior tan sólo envía un mensaje de alerta, pero con Javascript ya hemos visto que se pueden hacer multitud de cosas que pueden dañar al usuario, especialmente relacionadas con el **robo de cookies y sesiones**.

### Tipos de ataques XSS

Los **ataques Cross-Site Scripting** pueden agruparse en dos grandes categorías, dependiendo de **cómo envían el código malicioso**: non-persistent XSS y persistent XSS. 

* # Non-persistent XSS

Los ataques **non-persistent XSS** o **reflected XSS** no almacenan el código malicioso en el servidor sino que lo pasan y presentan directamente a la víctima. Es el método más popular de ataque XSS. El ataque se lanza desde una fuente externa, mediante email o un sitio de terceros.

```
// Palabra de búsqueda
echo "Has buscado la palabra: " . $_GET["query"];
// Resultados de búsqueda ...
// ...código...
```

El código anterior sería de un **motor de búsquedas**, donde la palabra que se ha buscado se devuelve a presentar al usuario. El problema es que la variable **$_GET["query"]** no es validada o escapada, por lo que un atacante podría enviar el siguiente link a la víctima:

```
http://example.com/search.php?query=<script>alert("hackeado")</script>
```

* # Persistent XSS

El **código malicioso** ya ha superado la barrera del proceso de validación y está almacenado en un **almacén de datos**. Puede ser un **comentario**, un archivo **log**, un mensaje de **notificación**, o cualquier otro tipo de sección del sitio web que solicita algún _**input**_ al usuario. Cuando esta información en particular se presenta en el sitio web, el código malicioso se ejecuta.

Si suponemos un sistema de comentarios en el que los comentarios se almacenan en un archivo llamado comentarios.txt:

```
file_put_contents("comentarios.txt", $_POST["comentario"], FILE_APPEND);
```

En cualquier parte del sitio web donde se vayan a mostrar los comentarios:

```
echo file_get_contents("comentarios.txt");
```

Cuando el atacante envía un comentario, inserta código malicioso en comentarios.txt. Cuando los comentarios se muestran, éste código se ejecutará sin ser validado o escapado de ninguna forma.

### Prevenir ataques XSS

Tan fácil como un atacante puede atacar un sitio web no protegido contra **ataques Cross-Site Scripting**, un desarrollador puede defenderse de éstos. La prevención ha de tenerse siempre en cuenta incluso antes de escribir el propio código.

La regla o política más básica que ha de tenerse siempre en cuenta es simple: **NUNCA confíes de datos que vienen de usuarios o de cualquier otra fuente externa**. Cualquier dato debe ser validado o escapado para su _**output**_. 

Las medidas a tomar se pueden dividir en tres: **data validation**, **data sanitization** y **output escaping**.

* # Data validation

La **validación de datos** es el proceso de asegurarse que tu aplicación analiza el tipo de datos correctos. Si tu script PHP espera un _**integer**_ de un input, cualquier otro tipo de dato debe de rechazarse. Cada dato debe ser validado cuando se recibe para asegurarse que es del **tipo correcto**, y rechazado si no pasa ese **proceso de validación**.

Si quieres validar un número de teléfono, por ejemplo, deberás rechazar cualquier string que contenga letras, porque sólo consistirá en dígitos. También puedes tener en cuenta la longitud que deberán tener estos dígitos. Siendo más permisivo, se pueden aceptar algunos otros símbolos como +, (), y - que a veces se utilizan al indicar números de teléfono:

```
// Comprobar un número de teléfono en Estados Unidos:
$telefono = '1-909-466-4344';
if (preg_match('/^((1-)?\d{3})-\d{3}-\d{4}/', $telefono)){
    echo "El teléfono $telefono es válido";
} else {
    echo "El teléfono $telefono NO es válido";
}
```

* # Data sanitization

La **sanitización de datos** se centra en manipular los datos para asegurarse que son seguros, eliminando cualquier parte indeseable y normalizándolos en la forma correcta. Por ejemplo, si se espera un texto _**string**_ de los usuarios, puedes querer evitar cualquier tipo de markup HTML:

```
// Sanitizar comentario de usuario
$comentario = strip_tags($_POST["comentario"]);
```

A veces la **validación de datos** y su **sanitización** pueden ir de la mano.

```
$telefono = "1234567";
$telefono = preg_replace('/[^\d]/', "", $telefono);
$length = strlen($telefono);
if ($length = 7 || $length = 10 || $length = 11){
    echo "$telefono es un formato válido";
}
```

* # Output escaping

Para proteger la integridad de los **datos que se devuelven**, el _**output data**_, se debe **escapar** cualquier dato que se devuelve al usuario. Esto evita que el navegador malinterprete alguna **secuencia especial de caracteres**: 

```
echo "Has buscado la palabra: " . htmlspecialchars($_GET["query"]);
```

Puede emplearse también la función _**htmlentities()**_. La diferencia entre ambas es que _**htmlspecialchars()**_ sólo traduce los símbolos **&**, **""**, **''**, **<** y **>** en **entidades HTML**, en cambio _htmlentities()_ traduce todos los caracteres posibles que tengan su equivalencia en HTML. Normalmente vale con _htmlspecialchars()_ a no ser que uses algún tipo de codificación diferente a **ISO-8859-1** o **UTF-8**. 

### Ejemplo de prevención contra ataques XSS

Mezclando un poco las tres **formas de prevenir ataques XSS**, vamos a ver un sencillo sistema de comentarios:

```
// Validar el comentario
$comentario = trim($_POST["comentario"]);
if(empty($comentario)){
    exit("Debes proporcionar un comentario");
}
// Sanitizar comentario
$comentario = strip_tags($comentario);
// El comentario ya se puede guardar de forma segura
file_put_contents("comentarios.txt", $comentario, FILE_APPEND);
// Escapar comentarios antes de mostrarlos
$comentarios = file_get_contents("comentarios.txt");
echo htmlspecialchars($comentarios);
```

Primero nos aseguramos de que no se guardan **comentarios vacíos**. Después se sanitizan los datos eliminando cualquier posible **etiqueta HTML** que pudiera contener. Finalmente, los comentarios se devuelven **filtrados**. La función _strip_tags_ hace que no sea posible insertar enlaces en los comentarios, ya que éstos utilizan una etiqueta que será eliminada. Para que puedan insertarse se puede utilizar _htmlentities_ o _htmlspecialchars_ en su lugar.

Hay que tener en cuenta que ninguna solución es fiable al 100%, y que es conveniente estar al tanto de novedades respecto a los **ataques Cross-Site Scripting** ya que van evolucionando a medida que lo hacen las plataformas que los facilitan (navegadores, HTML...).

## 2. Ataques CSRF Cross-Site Request Forgeries

Los **ataques Cross-Site Request Forgeries**, más comúnmente llamados **ataques CSRF**, se producen cuando el atacante provoca que el usuario ejecute una acción de forma no intencionada en una aplicación en la que había iniciado sesión.

Por ejemplo cuando, estando el usuario logeado en su sitio favorito, procede a **hacer click en un enlace que parece inofensivo**. En el fondo, su información de perfil está siendo actualizada con la dirección email del atacante. El atacante puede ahora usar la opción de recuperar la contraseña en el sitio web para cambiar la contraseña de la cuenta mediante email.

Cualquier acción que pueda realizar un usuario cuando está logeado en un sitio web la puede realizar también el atacante, ya sea **actualizar su perfil**, añadir **objetos a la cesta de la compra**, postear **mensajes en un foro** o cualquier otra cosa.

### Como funcionan los ataques CSRF

Para ver cómo funcionan es mejor verlos en acción. Vamos a simular un sistema de usuarios con su **página de acceso** (_login.php_), un **script que procese el inicio y cierre de sesión** (_process.php_), y una página que simula un ataque (_veneno.html_).

El código para _login.php_:

```
<?php
session_start();
?>
<html>
<body>
<?php
if (isset($_SESSION["user"])) {
    echo "<p>Bienvenido de vuelta, " . $_SESSION["user"] . "!<br>";
    echo '<a href="process.php?action=logout">Logout</a></p>';
}
else {
    ?>
    <form action="process.php?action=login" method="post">
        <p>El nombre de usuario es: admin</p>
        <input type="text" name="user" size="20">
        <p>La contraseña es: test</p>
        <input type="password" name="pass" size="20">
        <input type="submit" value="Login">
    </form>
    <?php
}
?>
</body>
</html>
```

El script de _login.php_ primero inicia los datos se sesión con _sessión_start()_, después comprueba si el usuario ya está logeado, y si no es así, muestra un **formulario de login**.

El archivo de proceso _process.php_ es el siguiente:

```
<?php
session_start();
switch($_GET["action"]) {
    case "login":
        if ($_SERVER["REQUEST_METHOD"] == "POST") {
            $user = (isset($_POST["user"])) &&
            ctype_alnum($_POST["user"]) ? $_POST["user"] : null;
            $pass = (isset($_POST["pass"])) ? $_POST["pass"] : null;
            $salt = '$2a$07$my.salt.mUy.Secr3t0';

        if (isset($user, $pass) && (crypt($user . $pass, $salt) ==
                crypt("admintest", $salt))) {
                $_SESSION["user"] = $_POST["user"];
            }
        }
        break;
    case "logout":
        $_SESSION = array();
        session_destroy();
        break;
}
header("Location: login.php");
?>
```

El archivo _process.php_ de nuevo inicia los datos de sesión, y después comprueba a ver si existe alguna acción con la que trabajar. En el caso de _**login**_, se realiza una pequeña **validación de _input_** con el operador ternario de PHP junto con las funciones _ctype_alnum()_ y _crypt()_. En el caso de _**logout**_, se destruye la sesión. Finalmente se redirige al usuario a _login.php_.

El **archivo malicioso** en cuestión es _veneno.html_:

```
<html>
<body>
<p>Esta página es veneno!!</p>
<img src="process.php?action=logout" style="display:none;">
</body>
</html>
```

Si visitas _login.php_, te logeas, y después visitas _veneno.html_, automáticamente se cerrará tu sesión aunque no hayas hecho click en el botón de _**logout**_. El navegador envía un _**request**_ al servidor para acceder al script _process.php_, esperando que sea realmente una imagen. El script en _process.php_ no tiene forma de diferenciar entre un _**request**_ válido iniciado por el usuario que hace click en el link de _**logout**_ o procedente de un **archivo malicioso**.

El archivo _veneno.html_ se puede alojar en un servidor totalmente diferente al que estás logeado, y funcionaría igual, ya que la página del atacante está haciendo un **request en tu nombre** usando la **sesión que tienes abierta**. No importa ni siquiera si el sitio web en el que estás logeado está en una red privada, el request se enviará desde tu dirección IP como si hubieras hecho el request tú mismo, haciendo que sea difícil rastrear la fuente maligna.

Si permites que los usuarios puedan **enlazar imágenes como imagen de perfil**, sin **escapar** apropiadamente y **sanitizar** los datos del usuario, podría realizarse incluso desde tu propio sitio web.

Como es lógico, cerrar la sesión de un usuario no es lo que se suele hacer para hacer daño. Se puede utilizar un **iframe escondido** (en lugar de una imagen) con un **formulario que automáticamente envía cuando se carga la página**, lo que haría que fuera posible cualquiera de los ataques mencionados al principio.

### Como evitar ataques CSRF

Para asegurarse que una acción está realmente siendo llevada a cabo por el usuario en lugar de un tercero, hay que asociarlo con algún tipo de **identificador único** que puede ser verificado después, llamado _**token**_. Para prevenir el ataque, podemos modificar _login.php_ así:

```
$_SESSION["token"] = md5(uniqid(mt_rand(), true));
echo '<a href="process.php?action=logout&csrf=' . $_SESSION["token"] . '">Logout</a></p>';
```

Y para **verificar el identificador**, podemos modificar _process.php_ como sigue:

```
case "logout":
    if (isset($_GET["csrf"]) && $_GET["csrf"] == $_SESSION["token"]) {
        $_SESSION = array();
        session_destroy();
    }
    break;
```

Con estas modificaciones, _veneno.html_ ya no puede cerrar la sesión porque se le ha añadido una nueva tarea al atacante: tener que **adivinar el token aleatorio**.

Para **proteger formularios**, se suele incluir el **identificador dentro de un campo escondido**, siendo enviado con el resto de los datos del formulario:

```
<input type="hidden" name="csrf" value="<?php echo $_SESSION["token"]; ?>">
```

## 3\. Seguridad en sesiones

Aunque con las **sesiones** y las **cookies** no se pueda quebrantar la **seguridad de la aplicación** de forma directa, mediante el robo de sesiones se pueden comprometer las **cuentas de los usuarios**, y si éstos tienen permisos especiales las consecuencias pueden ser peor de lo esperado.

La mayoría de las veces PHP guardará una cookie en el ordenador del cliente llamada **PHPSESSID** (puede cambiarse al nombre que se desee) cuando se usen sesiones. Esta **cookie** guardará un valor, un **identificador de sesión**, que está asociado con algún tipo de datos en el servidor. Si el usuario tiene una **session ID válida**, los datos asociados con la sesión se incluirán en el **superglobal** _array_ **$_SESSION**. Las sesiones pueden también transferirse a través de URL. En ese caso sería algo como _**?PHPSESSID=id_aqui**_.

La **sessión ID** funciona de forma parecida a la **llave de un cajón en un banco**, con la que puedes acceder a lo que sea que haya en ese cajón. La llave de tu cajón puede ser robada al igual que puede serlo la sessión ID de usuarios (robada o interceptada).

Cuando el atacante roba una session ID y trata de usarla para entrar en la aplicación como si fuera el verdadero usuario se le llama **session hijacking**. Cuando el atacante establece la session ID para la sesión de un usuario se denomina **session fixation**. Estos ataques no se pueden evitar en su totalidad pero se pueden tomar medidas para prevenirlos.

El problema de la seguridad en sesiones aumenta cuando se utiliza un **hosting compartido**, es decir, usar el **mismo servidor** que otros usuarios. En un server **Linux**, por defecto las sesiones de guardan en el directorio _/tmp_, que guarda archivos temporales y ha de ser legible y escribible para todo el mundo. Si tus sesiones se guardan ahí, otros usuarios podrían encontrar tus datos.

### Opciones de configuración

La **administración de sesiones HTTP** es una parte fundamental de la seguridad web. Algunas de las configuraciones más importantes que pueden manipularse en PHP son las siguientes:

*   _session.cookie_lifetime = 0_. El cero tiene un significado especial, le dice a los navegadores que no guarden una cookie permanentemente. Así pues, si se cierra el navegador, la session ID se borra inmediatamente. Se desaconseja variar este valor, y si se quiere una **aplicación con autologin**, existen alternativas mucho más seguras.
*   _session.use_cookies = On_ y _session.use_only_coockies = On_. Aunque las HTTP cookies tienen algunos problemas, es la forma más recomendable de manejar sesiones.
*   _session.use_strict_mode = On_. Esto previene al **módulo de sesión** iniciar una **sessión ID** sin inicializar. El módulo de sesión sólo acepta **session ID** válidas generadas por él mismo, rechazando cualquier sesson ID proporcionada por usuarios. Sessión ID injection puede hacerse con cookie injection a través de JavaScript. Se recomienda mantelerla en On.
*   _session.cookie_httponly = On_. Rechaza el **acceso a la cookie de sesión vía JavaScript**. Esto previene el **robo de cookies** a través de **JavaScript injection**. Se puede usar **session ID** como _**CSRF protection key**_, pero no es recomendable. 
*   _session.cookie_secure = On_. Permite el acceso a la cookie session ID sólo cuando el protocolo es HTTPS. Si tu aplicación es sólo HTTPS, es recomendable tener activada esta opción.
*   _session.gc_maxlifetime = [elegir el menor posible]_. Número de segundos tras los cuales los datos serán considerados basura y limpiados con posterioridad. Garbage Collection puede ocurrir al inicio de sesión mediante probabilidad. Esta opción no garantiza la eliminación de sesiones antiguas. Aunque el desarrollador no ha de fiarse del todo de esta configuración, se recomienda ajustarla al menor valor posible. Es mejor ajustar _**session.gc_probability**_ y _**session.gc_divisor**_ para que sesiones obsoletas se eliminen con la frecuencia apropiada. 
*   _session.use_trans_sid = Off_. El tener esta opción de manejo de session ID transparentes desactivada mejora la seguridad de sesiones eliminando la posibilidad de session ID injection y session ID leak. De todas formas, pueden emplearse session ID transparentes si es necesario.
*   _session.referer_check = [tu url original]_. Si está activala la opción anterior, session.use_trans_sid, el uso de esta opción es recomendable, ya que reduce el riesgo de ID injection.
*   _session.cache_limiter = nocache_. Asegúrate de que los **contenidos HTTP** no son cacheados para **sesiones autentificadas**. "_private_" suele usarse para cuando no hay ningún dato de seguridad en el contenido HTTP, y "_public_" cuando no contiene ningún dato privado en general.
*   _session.hash_function = "sha256"_. Una **función hash** más fuerte generará una **session ID** más fuerte. Cuanto más complejo sea el cifrado mejor: sha384, sha512...

El **módulo de sesión no puede garantizar que la información guardada en una sesión sólo sea vista por el usuario que creó la sesión**. Es necesario tomar más medidas para proteger totalmente la confidencialidad de la sesión.

Existen diferentes formas de que se **filtre una session ID exitente a terceros**. Si se produce una _**leaked session ID**_, posibilita a la tercera persona acceder a todos los recursos asociados con esa ID. Formas de que esto ocurra:

*   **URLs llevando una session ID**. Si enlazas a un sitio externo desde una URL que contiene una session ID, ésta aparecerá en el _**referrer log**_ del sitio externo.
*   **Un atacante más activo puede estar atento al tráfico de red**. Si éste no está encriptado, las **session IDs** pueden viajar en texto plano por la red. La solución es implementar **SSL en el servidor** y hacerlo obligatorio a los usuarios. Se debería usar **HSTS**.

Cuando la opción _**session.use_strict_mode**_ está On, una session ID no iniciada se rechaza y se crea una nueva session ID. Esto protege ataques que fuerzan a los usuarios a usar una session ID. Por ejemplo, un atacante puede pasar URLs que contienen una session ID: http://example.com/page.php?PHPSESSID=23411\. Si session.use_trans_sid está activado, la víctima iniciará la sesión con la session ID proporcionada por el atacante. _**session.use_strict_mode**_ reduce el riesgo, aunque no asegura. 

Para la autentificación de usuarios es muy recomendable añadir _**session_regenerate_id()**_, y debe ser llamada antes de establecer información de autentificación a **$_SESSION**. Esta función se asegura de que nuevas sesiones contienen información de autentificación almacenada sólo en una nueva sesión.

La **session ID debería ser regenerada** por lo menos cada vez que el usuario es identificado. No se ha de confiar en la expiración de la session ID. Los atacantes pueden acceder a las session ID de las víctimas periódicamente para evitar la expiración. Es recomendable implementar algún sistema propio para manejar sesiones antiguas.

_**session_regenerate_id()**_ por defecto no elimina una sesión antigua, y puede estar disponible para su uso. Para ello ha de ser destruída, añadiendo el parámetro _**delete_old_session**_ TRUE a la función, aunque esto puede tener consecuencias inesperadas. Una sesión puede ser destruída cuando hay conexiones simultáneas a la aplicación o la red es inestable. En lugar de destruir la sesión antigua inmediatamente, se puede establecer un tiempo corto de expiración en $_SESSION. Si el usuario intenta acceder a la sesión antigua, deniega el acceso a ella.

No se deben usar **session IDs** muy duraderas para **autologin** porque incrementa el riesgo de robo de sesión. Una _**autologin key**_ debe protegerse lo más posible, para ello se pueden utilizar **atributos _/httponly/_**. 

### Prevenir ataque session fixation

Cuando el atacante establece la session ID para la sesión de un usuario se denomina **session fixation**. Una vez que el atacante proporciona una URL al usuario con la session ID establecida y éste accede, el atacante, al conocer la session ID con la que se ha accedido, puede hacerse pasar por el usuario. Para prevenir el **session fixation** se deben tomar las siguientes medidas:

*   Establecer _**session.use_trans_sid = 0**_ en el _php.ini_. esto le dice a PHP que no incluya el session ID en la URL, y no leer la URL en busca de identificadores.
*   Establecer _**session.use_only_cookies = 1**_ en el php.ini. Esto le dice a PHP que nunca use URLs con session IDs.
*   Regenerar la session ID siempre que el estado de la sesión cambie. Ejemplos: autentificación de usuario, guardar información importante en la sesión, cambiar cualquier cosa de la sesión...

### Prevenir ataque session hijacking

_**Session hijacking**_ es cuando el atacante roba una session ID y trata de usarla para entrar en la aplicación como si fuera el verdadero usuario. Como el atacante tiene el session ID, el servidor no puede distinguir cual es el verdadero usuario. Para prevenir el **session hijacking** se deben tomar las siguientes medidas:

*   **Usar un identificador hash muy potente**. Directiva _session.hash_function_ en php.ini. Lo ideal es _session.hash_function = sha256_ o _session.hash_function = sha512_.
*   **Enviar un hash potente**. Directiva _session.hash_bits_per_character_ en php.ini. Configura esta opción a _session.hash_bits_per_character = 5_. Es una traba más para cuando el atacante trate de adivinar el session ID. El ID será más corto pero usará más caracteres.
*   **Establece una entropía adicional** con _session.entropy_file_ y _session.entropy_length_ en php.ini. Por ejemplo _session.entropy_file = /dev/urandom_ y _session.entropy_length = 256_, el número de bytes que serán leídos del archivo entropy.
*   **Cambia el nombre por defecto de la sesión PHPSESSID**. Este valor se establece con la función [session_name()](http://us2.php.net/manual/en/function.session-name.php), con el valor de identificación propio como parámetro, antes de llamar a _**session_start()**_. 
*   **Rotar el nombre de la sesión**, pero ten en cuenta que todas las sesiones serán invalidadas si cambias esto, por ejemplo si se configura de forma que dependa en el tiempo.
*   **Rota el session ID a menudo**. No es recomndable hacerlo en cada request (a no ser que se necesite un nivel de seguridad extremo), pero si en intervalos aleatorios. Si el atacante hace _**session hijacking**_ no se desea que pueda utilizar una sesión durante demasiado tiempo.
*   **Incluir el user agent desde $_SERVER['HTTP_USER_AGENT'] en la sesión**. Cuando la sesión comience, guardala en _**$_SESSION['user-agent']**_. Entonces en cada request posterior comprueba que coincide. Este valor puede ser falso, por lo que no es 100% seguro, pero es mejor que nada.
*   **Incluir la IP de usuario de $_SERVER['REMOTE_ADDR'] en la sesión**. Cuando la sesión comience, guárdala en _**$_SESSION['remote_ip']**_. Esto puede ser problemático para ciertos **ISPs** que usan direcciones IP múltiples para sus usuarios. Pero si se usa puede ser mucho más seguro. La única forma de que un atacante pueda **falsear la IP** es comprometiendo la red en algún punto entre el usuario real y tu servidor. Y si un atacante consigue comprometer una red, puede hacer cosas mucho peores que session hijacking. 
*   **Incluir un token en la sesión y en el navegador que incrementas y comparas frecuentemente**. Para cada _**request**_, haz $_SESSION['counter']++ en el lado del servidor. Crea algo en JS en el lado del navegador haciendo lo mismo (usando almacenamiento local). Entonces cuando se envía un request, coge el **nonce de un token** y verifica que el nonce es el mismo que en el servidor. Haciendo esto, se puede detectar una **session hijacked** ya que el atacante no tendrá el número exacto, y si lo tiene habrá dos sistemas transmitiendo el mismo número. Esto no funcionará en todas las aplicaciones, pero es otra manera de protegerse. 

### Diferencia entre session hijacking y session fixation

La diferencia entre session fixation y session hijacking reside simplemente en cómo la session ID es comprometida. En **session fixation**, el identificador se establece a un valor que el atacante conoce de antemano. En **session hijacking** se adivina o se roba del usuario. Una vez que el session ID es robado, los efectos de ambos dos son los mismos.

## 4\. Encriptación y contraseñas

Desde el principio **PHP** ha sido un **lenguaje de programación para la construcción de sitios web**. Esa idea permanece en el núcleo del lenguaje, y por eso es tan popular para la **construcción de aplicaciones web**. Cuando se creó en los años 90, el término **aplicación web** no existía aún, por lo que la **protección de contraseñas para cuentas de usuarios** no era algo en lo que estuviera centrado. 

Han pasado muchos años desde entonces y actualmente es impensable una aplicación web que no proteja las cuentas de los usuarios con **contraseñas**. Es fundamental para cualquier programador hacer que estas contraseñas tengan una **encriptación segura y eficiente**. **PHP 5.5** añadió una nueva librería llamada [Hash de contraseñas](http://php.net/manual/es/book.password.php) para la **encriptación de contraseñas**, con funciones que facilitan la tarea y utilizan los últimos métodos más eficaces.

### La importancia de los hashes seguros

Siempre hay que guardar las contraseñas encriptadas mediante un **algoritmo de encriptación** como el **algoritmo hashing** para hacer imposible a alguien que acceda a una base de datos conseguir averiguar la contraseña. Esto no es sólo para proteger a los usuarios frente a algún atacante sino también frente a los propios empleados de la aplicación.

Mucha gente utiliza las mismas contraseñas para muchas **aplicaciones web**. Si alguien accede a la dirección de email y contraseña de un usuario, probablemente pueda hacerlo en muchas otras aplicaciones.

Los hashes no se crean iguales, se emplean algoritmos muy distintos para crear un hash. Los dos más usados en el pasado son **MD5** y **SHA-1**. Los ordenadores de hoy en día pueden **crackear** fácilmente estos **algoritmos**. Dependiendo de la **complejidad y longitud de la contraseña**, se puede crackear en menos de una hora con los dos algoritmos nombrados (los ratios son 3650 millones de cálculos por segundo con MD5 y 1360 millones por segundo con SHA-1).

Por eso es importante usar **algoritmos complejos**. Si el hash es más largo reduce el **riesgo de colisiones entre contraseñas** (dos frases generando el mismo hash), pero también conviene que la aplicación se tome el **tiempo necesario para generar el hash**. Esto es porque el usuario apenas notará un segundo o dos más de tiempo de carga al logearse, pero se consigue que crackearlo tome muchisimo más tiempo, en case de que sea posible.

También es necesario protegerse frente a las **Rainbow Tables**. Las Rainbow Tables, como a la **MD5** que puede verse en [este enlace](http://md5cracker.org/), son **tablas de búsqueda inversa para hashes**. El creador de las tablas precalcula los hashes MD5 para palabras comunes, frases, palabras modificadas y strings aleatorios. La **facilidad de crackear un algoritmo MD5** hace posible la existencia de este tipo de tablas.

Generar este tipo de tablas para un **algoritmo complejo** tarda mucho más, pero es posible también. Una medida apropiada es **añadir un salt al hash**. En este contexto, salt es cualquier frase que se añade a la contraseña antes de crear el hash. Usando un salt se gana mucho terreno frente a este tipo de tablas. Se debería **crear una Rainbow Table específica para tu aplicación** y averiguar cual es el salt en tu aplicación.

### Mejora de los antiguos métodos de encriptación

Primero veamos la funciones básicas de hashing para PHP:

* # md5
```
string md5 (string $str [, bool $raw_output = false ])
```

Calcula un hash con el algoritmo [md5](http://www.faqs.org/rfcs/rfc1321.html). Si se establece _$raw_output_ como true se devolverá en raw binario con una longitud de 16\. De normal devuelve un hash de 32 caracteres hexadecimal.

* # sha1
```
string sha1 (string $str [, bool $raw_output = false ])
```

Calcula un hash con el algoritmo [sha1](http://www.faqs.org/rfcs/rfc3174.html). Si se establece _$raw_output_ como true se devolverá en raw binario con una longitud de 20\. De normal devuelve un hash de 40 caracteres hexadecimal.

* # hash
```
string hash ( string $algo, string $data [, bool $raw_output = false ] )
```

La función toma primero el algoritmo que se desea emplear, _$algo_, y después el string que se desea encriptar, _$data_. El algoritmo puede ser **md5**, **sha128**, **sha256**...

Anteriormente, el siguiente código era un **ejemplo de una buena protección de contraseñas**:

```
class Password {
    const SALT = 'EstoEsUnSalt';
    public static function hash($password) {
        return hash('sha512', self::SALT . $password);
    }
    public static function verify($password, $hash) {
        return ($hash == self::hash($password));
    }
}
// Crear la contraseña:
$hash = Password::hash('micontraseña');
// Comprobar la contraseña introducida
if (Password::verify('micontraseña', $hash)) {
    echo 'Contraseña correcta!\n';
} else {
    echo "Contraseña incorrecta!\n";
}
```

Durante mucho tiempo esto ha sido la mejor forma de protegerse, mejor que usar md5\. Se usa un algoritmo mucho más complejo como el sha512, y fuerza a todas las contraseñas a usar un salt, pero tiene algunas carencias:

*   **Se utiliza un salt, pero todas las contraseñas utilizan el mismo**, por lo que si alguien consigue averiguar una contraseña, o el acceso al código fuente donde puede mirar el hash, se puede hacer una Rainbow Table añadiendo el salt descubierto. La solución es crear un salt aleatorio para cada contraseña que se crea, y guardar el salt con la contraseña de forma que después se pueda recuperar.
*   **Se utiliza sha512, un complejo algoritmo que viene con PHP**. Sin embargo también puede ser crackeado a un ratio de 46 millones de cálculos por segundo. Aunque es más lento de crackear que **md5** y **sha1**, todavía no es un nivel de seguridad estable. La solución es utilizar algoritmos que son todavía más complejos y emplearlos varias veces. Por ejemplo emplear un algoritmo **sha512** 10 veces consecutivamente reduciría el intento de hackeo considerablemente.

Las dos soluciones ya vienen por defecto con la librería Hash de contraseñas de PHP.

### La librería Hash de contraseñas de PHP

La extensión Hash de contraseñas crea un password muy complejo, incluyendo la generación de salts aleatorios. En forma más simple se utiliza la función [_password_hash()_](http://php.net/manual/en/function.password-hash.php), con la contraseña que quieres "hashear", y la extensión lo hace directamente. Es necesario facilitar también el algoritmo que se desea emplear. La mejor opción de momento es especificar **PASSWORD_DEFAULT** (se actualiza siempre que se añada un algoritmo nuevo más fuerte), aunque también es posible **PASSWORD_BCRYPT**. 

* # password_hash
```
string password_hash (string $password , integer $algo [, array $options ] )
```

Es compatible con _crypt()_ por lo que los hash de contraseñas creados con _crypt()_ se pueden usar con _password_hash()_. Las opciones que se admiten son _**salt**_ (para proporcionarlo manualmente, pero esta opción ya está obsoleta en PHP 7 por lo que no conviene usarla) y _**cost**_, que denota el **coste del algoritmo** a usar (el valor predeterminado es 10).

```
$hash = password_hash('micontraseña', PASSWORD_DEFAULT, [15]);
```

El coste indica cuánto de complejo debe ser el algoritmo y por lo tanto cuánto tardará en generarse el hash. El número se puede considerar como el número de veces que el algoritmo hashea la contraseña. 

Para poder verificar los passwords, deberíamos saber el salt que se ha creado. Si se usa _password_hash()_ otra vez y se compara con el anterior, se puede ver que son distintos. Cada vez que se llama a la función, se genera un nuevo hash, por lo que la extensión facilita una segunda función: [_password_verify()_](http://php.net/manual/es/function.password-verify.php). Llamando a esta función y pasando la contraseña proporcionada por el usuario, la función devolverá true si coincide con la almacenada:

```
if(password_verify($password, $hash)){
    // Password correcto!
}
```

Ahora la clase que habíamos puesto al principio se puede refactorizar por una mucho más segura:

```
class Password {
    public static function hash($password) {
        return password_hash($password, PASSWORD_DEFAULT, ['cost' => 15]);
    }
    public static function verify($password, $hash) {
        return password_verify($password, $hash);
    }
}
```

### Cambios en los métodos de encriptación

Usando la **extensión de encriptación de PHP**, tu aplicación estará con los **últimos estándares en seguridad**, aunque hace algunos años de decía que **SHA-1** era lo mejor. Esto significa que cada vez se van actualizando los logaritmos y por tanto la extensión se irá adaptando.

¿Y qué ocurre con las **contraseñas antiguas**? Para eso está la función _password_needs_rehash()_, que detecta si una contraseña almacenada no cumple con las necesidades de seguridad de la aplicación. La razón puede ser que hayas **aumentado la complejidad con _cost_**, o que **PHP haya actualizado el algoritmo**. Por esta razón se ha de elegir PASSWORD_DEFAULT, siempre se estará protegido con la opción más segura disponible.

Cuando un usuario se logea, ahora tendríamos una nueva tarea, llamar a _password_needs_rehash()_, que toma parámetros similares a _password_hash()_. Lo que hace la función _password_needs_rehash()_ es decirte si el password necesita un rehash. Depende de ti **cuándo generar un nuevo hash de contraseña y guardarlo**, porque la extensión Hash desconoce cómo deseas hacerlo. 

El siguiente es un ejemplo de una **clase Usuario** simulada para ver el funcionamiento de la **extensión Hash**, y sirve de orientación a cómo podria realizarse:

```
class User {
    // Opciones de contraseña:
    const HASH = PASSWORD_DEFAULT;
    const COST = 14;
    // Almacenamiento de datos del usuario:
    public $data;
    // Constructor simulado:
    public function __construct() {
        //  Leer los datos de la base de datos almacenados en $data, como
        //  $data->passwordHash  o  $data->username
    }
    // Funcionalidad de guardar los datos simulada:
    public function save() {
        // Guardar los datos de $data en la base de datos
    }

```

Ya hemos construido la base de la clase **user**, ahora vamos a ver el cambio de contraseña y el login:

```
// Permite el cambio de contraseña:
    public function setPassword($password) {
        $this->data->passwordHash = password_hash($password, self::HASH, ['cost' => self::COST]);
    }
    // Logear un usuario:
    public function login($password) {
        // Primero comprobamos si se ha empleado una contraseña correcta:
        echo "Login: ", $this->data->passwordHash, "\n";
        if (password_verify($password, $this->data->passwordHash)) {
            // Exito, ahora se comprueba si la contraseña necesita un rehash:
            if (password_needs_rehash($this->data->passwordHash, self::HASH, ['cost' => self::COST])) {
                // Tenemos que hacer rehash en la contraseña y guardarla.  Simplemente se llama a setPassword():
                $this->setPassword($password);
                $this->save();
            }
            return true; // O hacer lo necesario para indicar que el usuario se ha logeado.
        }
        return false;
    }
}
```
