Una **distribución** está diseñada para cumplir las espectativas de un determinado usuario, ya sea para familiarizarse, para crear un **sitio web dinámico con un gestor de contenidos** (CMS) o para probar el framework con su versión completa.

Una distribución Symfony esta hecha a partir de componentes de Symfony, una selección de bundles, una estructura de directorios, una configuración por defecto y un sistema de configuración web opcional.

La **fuerza e innovación de Symfony** reside en el hecho de que cualquier cosa se puede modificar: la distribución es un **pack**, pero diseñado para ser **escalable**, como si lo hubieras creado tú mismo. Puedes **añadir o eliminar bundles**, cambiar la configuración por defecto, modificar la estructura de archivos, etc, según tus necesidades.

#### Distribuciones disponibles

*   [Standard Edition](http://symfony.com/download): es la mejor versión para empezar. Contiene los bundles más usados y tiene una configuración simple compatible con la mayoría de proyectos.
*   [Hello World Edition](https://github.com/symfony/symfony-hello-world): basada en la **Standard**, es la ideal para comparar el **rendimiento del framework**.
*   [Symfony CMF Standard Edition](https://github.com/symfony-cmf/standard-edition): es la distribución para empezar con el [CMF Symfony](http://cmf.symfony.com/).
*   [Symfony REST Edition](https://github.com/gimler/symfony-rest-edition): muestra como construir una aplicación mediante una **RESTful API** usando el [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle) y otros bundles relacionados.

Para instalar la **Standard Edition** lo ideal es utilizar el [Symfony Installer](http://symfony.com/download), una **aplicación PHP**. Simplifica la creación de proyectos en Symfony. 

Una vez se instala sólo hay que escribir el siguiente código cada vez que quieres un nuevo proyecto:

```
symfony new mi_proyecto
```