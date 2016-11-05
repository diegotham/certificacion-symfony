Los **estándares** para nombrar los diferentes elementos en **Symfony** son los siguientes:

*   Usa **camelCase**, no barras bajas, para variables, funciones, métodos y argumentos.
*   Usa **barras bajas** para nombres de opciones y parámetros.
*   Usa **namespaces** para todas las clases.
*   Las **clases abstract** deben ir prefijadas con _Abstract_. 
*   Las **interfaces** deben llevar el sufijo _Interface_.
*   Los **traits** deben llevar el sufijo _Trait_.
*   Las **excepciones** deben llevar el sufijo _Exception_. 
*   Usa caracteres alfanuméricos y barras bajas para nombres de archivos.
*   Para el **type hinting** en **PHPDocs** y **casting**, usa _**bool**_ en lugar de boolean, _**int**_ en lugar de integer, y _**float**_ en lugar de double o real. 
*   Existen otras convenciones más para el core del framework que pueden verse [aquí](http://symfony.com/doc/2.3/contributing/code/conventions.html). 

También existen **convenciones de nombres para los services**:

*   El **nombre de un service** contiene grupos, separados por puntos.
*   El **DI alias** del bundle es el **primer grupo** (ej: fos_user).
*   Usa letras en minúscula para nombres en **parámetros** y **servicios**.
*   Un **nombre de grupo** utiliza la notación subrayado.