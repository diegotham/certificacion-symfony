Una **aplicación moderna en PHP** está llena de **objetos**. Un objeto se puede encargar del envío de mensajes de email mientras otro se encarga de registrar datos en la base de datos. En tu aplicación, puedes crear un objeto que administra un inventorio de productos, y otro objeto que procesa datos de un API de terceros. La cuestión es que en las aplicaciones modernas todo está lleno de objetos y conviene tener una forma de organizarlos.

El **Service Container** es un objeto especial que ayuda a organizar e instanciar los objetos de una aplicación, ayudando a estandarizar y centralizar la forma en que los objetos están estructurados en la aplicación. El container hace las cosas más fáciles, es muy rápido y enfatiza una arquitectura que promueve código reusable. Todas las **clases core de Symfony** utilizan el container, por lo que es importante saber cómo extender, configurar y usar cualquier objeto de Symfony. **En gran parte, container es el mayor contribuyente a la rapidez y extensibilidad de Symfony**.

### ¿Qué es un service?

Un _service_ es un **objeto** que efectúa una **tarea global**. Cada _service_ se usa en la aplicación cuando se necesita la funcionalidad que desenvuelve. La ventaja que tienen es que te permite pensar en la aplicación en forma de diferentes tareas separadas por services, pudiendo acceder a cualquier tarea en cualquier momento. Esto hace que también sea más fácil **testear** estos objetos.

Por ejemplo un **Mailer** service se utiliza globalmente para enviar mensajes de email mientras que los objetos **Message** que envía no son services. De la misma forma, un objeto **Product** no es un service, pero un objeto que registra objetos **Product** en una base de datos es un service.

### ¿Qué es un Service Container?

Un **Service Container** (o _dependency injection container_) es un objeto PHP que administra la instancia de services (objetos).

Si por ejemplo tenemos una clase PHP que envía mensajes de email, sin un service container tenemos que crear manualmente el objeto cuando lo necesitemos:

```
use AppBundle\Mailer;

$mailer = new Mailer('sendmail');
$mailer->send('ryan@example.com', ...);
```

La clase **Mailer** permite configurar el método usado para enviar mensajes de email (_sendmail_, _stmp_, etc). Pero si por ejemplo queremos usar el mailer service en algún otro lado, vendría bien no tener que repetir la configuración mailer cada vez que queremos usar el objeto Mailer. Además, si quisiéramos cambiar el _transport_ de _sendmail_ a _smtp_ en todas las partes de la aplicación, tendríamos que modificar cada archivo individualmente.