Los bundles pueden ser específicos de una aplicación o reusables. Para que sean reusables es necesario que sigan una serie de convenciones de nombrado.

Un bundle es también un **namespace PHP**. El namespace debe seguir los estándares **PSR-0** o **PSR-4**: comienza con un **vendor**, seguido de cero o más **segmentos categóricos**, y termina con el **nombre corto** del namespace, que debe terminar con el sufijo Bundle.

Un namespace se convierte en bundle cuando se le añade una clase bundle. El nombre de la clase bundle debe seguir unas simples reglas:

*   Utiliza sólo caracteres **alfanuméricos y barras bajas**.
*   Utiliza un nombre **camelCase**.
*   Utiliza un nombre **corto y descriptivo** (no más de dos palabras).
*   Añade un prefijo al nombre con la **concatenación del vendor** (y opcionalmente los namespaces categóricos).
*   Utiliza el sufijo **Bundle**.

Ejemplos de bundle namespaces y nombres de clases:

| | |
| -------- | -------- |
| **Namespace** | **Bundle Class Name** |
| Acme\Bundle\BlogBundle | AcmeBlogBundle |
| Acme\BlogBundle | AcmeBlogBundle |

Por convención, el método _getName()_ de la clase bundle debe devolver del nombre de la clase.

Si compartes el bundle públicamente, debes usar el nombre de clase del bundle como el nombre del repositorio (**AcmeBlogBundle** y no **BlogBundle** por ejemplo).

Los bundles del core de Symfony no tienen como prefijo el nombre de la clase y siempre añaden el sufijo Bundle. Por ejemplo: **FrameworkBundle**.

Cada bundle tiene un alias, que es la versión corta en minúsculas del nombre del bundle utilizando barras bajas (acme_blog para AcmeBlogBundle). Este alias se utiliza para aplicar unicidad dentro de un proyecto y para definir las opciones de configuración del bundle.