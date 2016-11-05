### Sistema de cache en el lado del cliente

Un** sistema de cache en el lado del cliente** o **forward cache** es una cache fuera de la red del servidor, por ejemplo en el ordenador del cliente, en un ISP o dentro de una red corporativa. Se trata de un tipo de cache _private_, ya que no se comparte con más usuarios. Un cliente como un **navegador web** guarda contenido web para reutilizarlo, por ejemplo cuando pulsas el botón de volver, se muestra una **versión local cacheada** de la página anterior, en lugar de enviar un nuevo request. 

### Sistema de cache en el lado del servidor

Un **sistema de cache en el lado del servidor** o **reverse cache** se sitúa frente a uno o más servidores y aplicaciones web, acelerando los requests y reduciendo la carga del servidor. Se trata de un tipo de cache _public_, ya que se comparte con más usuarios. Un buscador como **Google** por ejemplo cachea **sitios web** y proporciona una forma de mostrar información de los sitios web cacheados rápidamente. 

Construído bajo los estándares del **protocolo HTTP**, el **browser request caching** permite al servidor controlar la frecuencia con la que el navegador solicita nuevas copias de archivos al servidor. Esto se consigue a través de los HTTP headers _Expires_, _If-Modified-Since_, _Last-Modified_, _Cache-Control_ y _ETag_.