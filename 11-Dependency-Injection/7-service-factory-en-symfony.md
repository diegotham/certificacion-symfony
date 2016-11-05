El **Service Container de Symfony** proporciona una potente forma de controlar la creación de objetos, permitiéndote especificar los argumentos que se pasan al constructor así como llamar a métodos y establecer parámetros. A veces esto puede no ser suficiente para **construir los objetos**, en cuyo caso puedes **emplear una factory para crear el objeto** y decirle al service container que llame a un método de la factory en lugar de instanciar directamente la clase.

Por ejemplo tenemos una factory que configura y devuelve un objeto **NewsletterManager**:

```
class NewsletterManagerFactory
{
    public static function createNewsletterManager()
    {
        $newsletterManager = new NewsletterManager();

        // ...

        return $newsletterManager;
    }
}
```

Para poner el objeto **NewsletterManager** disponible como service, puedes configurar el service container para usar el método factory _NewsletterManagerFactory::createNewsletterManager():_

```
services:
    newsletter_manager:
        class:   NewsletterManager
        factory: [NewsletterManagerFactory, createNewsletterManager]
```

Cuando se usa una **factory** para crear services, el valor elegido para la opción _class_ no tiene ningún efecto sobre el service resultante. El **nombre de clase generado** sólo depende del objeto devuelto por la factory. Sin embargo, el **nombre de clase configurado** puede ser utilizado por **compiler passes** y por tanto debería establecerse.

El método se llamará estáticamente. Si queremos que la clase factory sea instanciada y se llame al método del objeto resultante, podemos configurar la propia factory como un service. En este caso el método se ha de cambiar a **non-static**:

```
services:
    newsletter_manager.factory:
        class: NewsletterManagerFactory
    newsletter_manager:
        class:   NewsletterManager
        factory: ["@newsletter_manager.factory", createNewsletterManager]
```

### Pasar argumentos al método Factory

Si necesitas pasar argumentos al **método factory**, puedes emplear la opción _arguments_ dentro del **service container**. Por ejemplo, si el método **createNewsletterManager** del ejemplo anterior adapta el service _templating_ como _argument_:

```
services:
    newsletter_manager.factory:
        class: NewsletterManagerFactory

    newsletter_manager:
        class:   NewsletterManager
        factory: ["@newsletter_manager.factory", createNewsletterManager]
        arguments:
            - '@templating'
```