**Índice de contenido**

1.  ¿Qué son las extensiones PHP?
2.  Incorporar extensiones PHP
3.  Activar y desactivar extensiones PHP
4.  Extensiones populares en PHP

### 1. ¿Qué son las extensiones PHP?

Las **extensiones** son una parte muy importante y extensa en **PHP**. Una **extensión en PHP** es un módulo que proporciona alguna funcionalidad concreta al motor de PHP. La mayoría de **funciones y clases predefinidas** vienen de extensiones que vienen ligadas a la **distribución PHP**, algunas compiladas de forma que no pueden eliminarse.

Como el propio **lenguaje PHP**, la gran mayoría de extensiones están escritas en **C** (aunque hay alguna en C++) y se compilan y cargan en PHP como un objeto compartido o DLL, o de forma estática. Esto difiere totalmente de las **librerías**, **componentes** y **frameworks** que están escritos y distribuídos como código PHP.

Existen extensiones que son consideradas como tal por utilizar la Extension _API_, y algunas son difícilmente separables de **PHP**, como las extensiones **SPL**.

La mayoría de extensiones proporcionan una **API** con nuevas clases, funciones y constantes. Hay otras que no lo hacen de forma directa, pero si añaden funcionalidades a otras ya existentes, como por ejemplo ocurre con las extensiones **PDO_MySQL** o **PDO_PGSQL**, que mejoran las capacidades de la **extensión PDO**.

### 2. Incorporar extensiones PHP

Muchas extensiones vienen directamente con PHP, y si no están habilitadas, puede hacerse a través del **php.ini**. También hay otras que no vienen directamente pero están disponibles en el [PHP Extension Community Library PECL](https://pecl.php.net/). Las **extensiones PECL** se pueden descargar, compilar, y añadir a una instalación PHP. Esto se puede hacer directamente a través de la **línea de comandos** con '_pecl_' o manualmente con herramientas estándar.

Lo normal es confiar en que las extensiones PECL funcionen correctamente, porque tienen que adherirse a ciertos estándares establecidos por el equipo de PHP y suelen considerarse estables.

Existen otras extensiones que por diferentes motivos (legales, por ejemplo) no se distribuyen a través de PECL. Con estas hay que ser mas cauto ya que no están adheridas a los estándares del equipo de PHP.

### 3. Activar y desactivar extensiones PHP

Una vez que la **extensión** está instalada (lo que significa que tienes el archivo DLL o .so en el directorio de extensiones PHP), necesita ser activada.

Primero, se puede comprobar la salida de **_phpinfo()_** o [_get_loaded_extensions()_](http://php.net/manual/es/function.get-loaded-extensions.php) para ver si ya está habilitada. Si no está habilitada, es necesario asegurarse que lo siguiente está configurado en el _php.ini_:

```
extension_dir= <directorio donde están instaladas las extensiones>
extension= myext.so
```

_"myext.so"_ es el nombre del archivo, sin el directorio. En este caso es un .so pero puede ser un DLL. Hay una línea _extension = file_ por cada extensión habilitada. Puedes comentar las líneas donde están definidas extensiones que no quieres utilizar, precediendo un punto y coma ;.

Después de hacer cambios en el **php.ini** es necesario reiniciar el **servidor web** o el **manejador de procesos FPM / FastCGI**.

Se comprueba de nuevo la salida de **_phpinfo()_** o _**get_loaded_extensions()**_ para ver si ya está habilitada. También se puede mirar el error log de PHP o del servidor, por si ha habido algún error al reiniciar que podría indicar algún problema con la carga de la extensión.

Para proyectos en **estado de producción**, se recomienda deshabilitar cualquier extensión que no se utiliza, para ahorrar CPU y reducir las posibilidades de cualquier **bug** o **problema de seguridad** con las extensiones que afectan al servidor. 

Algunas extensiones están construidas estáticamente y no aparecen en la lista en _**p**__**hpinfo()**_ o _**get_loaded_extensions**_ y no tienen archivo en el directorio de extensiones, por lo que no pueden deshabilitarse. **Eliminar extensiones compiladas de forma estática** requiere **recompilar PHP entero**, y en la mayoría de los casos no es necesario, ya que incluyen alguna funcionalidad básica de PHP que muy pocas veces es necesario remover.

### 4. Extensiones populares en PHP

Existen más de 150 **extensiones de PHP estandarizadas** listadas de distintas formas: [categóricamente](http://php.net/manual/es/funcref.php), [por orden alfabético](http://php.net/manual/es/extensions.alphabetical.php), [por incorporación](http://php.net/manual/es/extensions.membership.php) o [por estado](http://php.net/manual/es/extensions.state.php).

A continuación se nombran y describen algunas de las **extensiones más importantes de PHP** ordenadas categóricamente (un total de 48 extensiones):

| | |
| -------- | -------- |
| **Extensión / Incorporación** | **Descripción / funciones y clases e interfaces** |
| **Extensiones básicas** | |
| [Manejo de variables](http://php.net/manual/es/book.var.php) / _núcleo_ | Funciones para el manejo de **variables**\: boolval, empty, floatval, get_defined_vars, gettype, intval, is_array, is_callable, is_null, isset, print_r, serialize, unset, unserialize, var_dump...|
| [Manejo de funciones](http://php.net/manual/es/book.funchand.php) / _núcleo_ | Funciones para el manejo de **funciones**\: call_user_func_array, call_user_func, func_get_args, func_num_args, function_exists, get_defined_functions...|
| [Strings](http://php.net/manual/es/book.strings.php) / _núcleo_ | Funciones para la manipulación de **strings**\: chr, echo, explode, htmlentities, htmlspecialchars, implode, join, md5, nl2br, print, printf, str_repeat, str_replace, str_shuffle, stripos, strlen, strtoupper, substr, ucfirst, trim... |
| [Arrays](http://php.net/manual/es/book.array.php) / _núcleo_ | Funciones para interactuar con **arrays** y manipularlos\: array_chunk, array_diff, array_keys, array_map, array_push, array_rand, array_replace, array_search, array_shift, array_pop, array_merge, array_walk, count, each, extract, list, reset, sort... |
| [Información de clases y objetos](http://php.net/manual/es/book.classobj.php) / _núcleo_ | Permiten obtener información sobre **clases** y **objetos**\: __autoload, call_user_method, class_alias, class_exists, get_called_class, get_class, get_object_vars, is_a, get_declared_classes, method_exists...|
| [Directorios](http://php.net/manual/es/book.dir.php) / _núcleo_ | Funciones para trabajar con **directorios**. **Clases**: Directory. **Functions**\: chdir, chroot, closedir, dir, getcwd, opendir, scandir...|
| [Sistema de ficheros](http://php.net/manual/es/book.filesystem.php) / _núcleo_ | Funciones para trabajar con **archivos**\: basename, chmod, chown, dirname, fclose, feof, file_exists, file_get_contents, file_put_contents, file, filesize, fopen, fwrite, is_dir, is_readable, is_uploaded_file, mkdir, pathinfo, readfile, rmdir, tmpfile...|
| [Fecha/Hora](http://php.net/manual/es/book.datetime.php) / _núcleo_ | Funciones para obtener **fecha y hora** que **dependen del servidor**. **Clases**: DateTime, DateTimeImmutable, DateTimeZone, DateInterval, DatePeriod. **Functions**\: checkdate, date_add, date_create, date_format, date_time_set, date, localtime, microtime, strtotime, time... (muchas funciones son alias de las clases)|
| [Sesiones](http://php.net/manual/es/book.session.php) / _núcleo_ | Funciones de **sesiones**, formas de preservar información para accesos subsiguientes. **Clases**: SessionHandler. **Interfaces**: SessionHandlerInterface. **Functions**\: session_abort, session_encode, session_id, session_name, session_start, session_status, session_unset...|
| [Mail](http://php.net/manual/es/book.mail.php) / _núcleo_ | Función que permite **enviar correos**\: mail |
| [PDO](http://php.net/manual/es/book.pdo.php) / _bundled_ | La extensión **PDO** (_PHP Data Objects_) define una interfaz ligera para acceder a bases de datos en PHP. Proporciona una capa de abstracción de acceso a datos que emplea las mismas funciones y consultas independientemente de la base de datos que se utilice. **Clases**\: PDO, PDOStatement, PDOException. **Controladores**\: MS SQL Server, Mysql, Oracle, PostgreSQL, SQLite...|
| **Afectan al comportamiento de PHP**  | |
| [APC](http://php.net/manual/es/book.apc.php) / _Pecl_ | Alternative PHP Cache, código de operación de **caché libre** y abierto para PHP. Clases: APCIterator. Functions: apc_add, aps_bin_dump, apc_cache_info, apc_clear_cache, apc_exists, apc_store...|
| [Manejo de errores](http://php.net/manual/es/book.errorfunc.php) / _núcleo_ | Funciones para el manejo y registro de **errores**\: debug_backtrace, error_clear_last, error_get_last, error_log, set_error_handler, trigger_error...|
| [OPCache](http://php.net/manual/es/book.opcache.php) / _bundled_ | **Mejora el rendimiento de PHP** almacenando el código de bytes en un script precompilado en la memoria compartida, eliminando la necesidad de que PHP cargue y analice los script en cada petición. Functions\: opcache_get_configuration, opcache_get_status, opcache_is_script_cached...|
| [Control de la salida](http://php.net/manual/es/book.outcontrol.php) / _núcleo_ | Funciones que permiten controlar **cuándo se envía la salida**. Functions\: flush, ob_clean, ob_flush, ob_get_contents, ob_get_status, ob_start...|
| [Opciones e información de PHP](http://php.net/manual/es/book.info.php) / _núcleo_ | Funciones que proporcionan **información sobre PHP**. Functions\: extension_loaded, gc_enable, get_defined_constants, get_extension_funcs, get_include_path, get_loaded_extensions, getenv, memory_get_usage, phpinfo, phpversion, set_time_limit, version_compare...|
| **Extensiones de compresión y archivos** | |
| [Bzip2](http://php.net/manual/es/book.bzip2.php) / _externa_ | Módulo para leer y escribir archivos comprimidos con **bzip2** (.bz2). Functions\: bzclose, bzcompress, bzdecompress, bzerror, bzflush, bzopen... |
| [Zlib](http://php.net/manual/es/book.zlib.php) / _bundled_ | Módulo para leer y escribir ficheros comprimidos con **gzip** (.gz). Functions\: gzclose, gzcompress, gzencode, gzfile, gzopen, gzread, gztell, gzwrite...|
| [LZF](http://php.net/manual/es/book.lzf.php) / _pecl_ | **LZF** es un algoritmo de compresión muy rápido, ideal para ahorrar espacio con un ligero costo en velocidad. Utiliza la [**librería liblzf**](http://oldhome.schmorp.de/marc/liblzf.html). Functions\: lzf_compress, lzf_decompress, lzf_optimized_for|
| [Zip](http://php.net/manual/es/book.zip.php) / _externa_ | Extensión que permite leer o escribir **archivos Zip** y los archivos de dentro. Clases: ZipArchive. Functions\: zip_close, zip_entry_filesize, zip_entry_name, zip_entry_name, zip_entry_read, zip_open, zip_read...|
| **Extensiones criptográficas** | |
| [Hash](http://php.net/manual/es/book.hash.php) / _núcleo_ | Motor para cifrar mensajes (algoritmos hash). Functions\: hash_algos, hash_copy, hash_file, hash_init, hash_update, hash...|
| [Mcrypt](http://php.net/manual/es/book.mcrypt.php) / _externas_ | Interfaz para la biblioteca mcrypt, admite una gran variedad de algoritmos de bloques como DES, TRIpleDES, Blowfish, 3-WAY, SAFER-SK64, SAFER-SK128, TWOFISH, TEA, RC2 y GOST en los modos de cifrado CBC, OFB, CBF y ECB. Functions\: mcrypt_cbc, mcrypt_cfb, mcrypt_decrypt, mcrypt_encrypt, mcrypt_generic, mcrypt_list_algorithms, mcrypt_list_modes, mcrypt_module_open, mcrypt_ofb...|
| [OpenSSL](http://php.net/manual/es/book.openssl.php) / _externas_ | Módulo que emplea las funciones de [OpenSSL](http://www.openssl.org/) (sólo algunas) para la generación y verificación de firmas y para encriptar y desencriptar datos. Functions:\ openssl_csr_export, openssl_csr_new, openssl_csr_sign, openssl_decrypt, openssl_digest, openssl_encrypt, openssl_open, openssl_pkey_free, openssl_seal, openssl_sign, spenssl_spki_new, openssl_verify...|
| [Hash de contraseñas](http://php.net/manual/es/book.password.php) / _núcleo_ | Proporciona una envoltura fácil de usar sobre [_**crypt()**_](http://php.net/manual/es/function.crypt.php) para hacerla sencilla para crear y administrar contraseñas de forma segura. Functions\: password_get_info, password_hash, password_needs_rehash, password_verify|
| **Soporte para el lenguaje humano y codificación de caracteres** | |
| [iconv](http://php.net/manual/es/book.iconv.php) / _bundled_ | Este módulo tiene una interfaz para la conversión de **conjuntos de caracteres iconv**. Se puede convertir un conjunto de caracteres local por otro, el cual puede ser **Unicode**. Es recomendable instalar la librería [GNU libiconv](http://www.gnu.org/software/libiconv/). Functions\: iconv_get_encoding, iconv_mime_encode, iconv_strlen, iconv_strpos, iconv_substr, iconv...|
| [intl](http://php.net/manual/es/book.intl.php) / _bundled_ | Extensión para la internacionalización _**intl**_ es una envoltura para la librería [ICU](http://site.icu-project.org/), permitiendo realizar un cotejo que cumple con el [UCA](http://www.unicode.org/reports/tr10/) y formatear fecha/hora/número/moneda en los scripts. **Clases**: Collator, NumberFormatter, Locale, Normalizer, MessageFormatter, IntlCalendar, IntlTimeZone, IntlDateFormatter, ResourceBundle, Spoofchecker, Transliterator, IntlChar, IntlException...|
| [mbstring](http://php.net/manual/es/book.mbstring.php) / _bundled_ | Los **esquemas de codificación multibyte** se desarrollaron para expresar más de 256 caracteres en el sistema de codificación regular a nivel de bits. Cuando se manipulan strings en una codificación multibyte, es necesario utilizar funciones especiales. _**mbstring**_ proporciona funciones específicas para cadenas de texto multibyte que ayudan a tratar codificaciones multibyte en PHP. Functions\: mb_convert_case, mb_detect_encoding, mb_encoding_aliases, mb_ereg_replace, mb_ereg_search_init, mb_ereg, mb_parse_str, mb_send_mail, mb_split, mb_strlen, mb_strrpos, mb_substr...|
| **Procesamiento y generación de imágenes** | |
| [Exif](http://php.net/manual/es/book.exif.php) / _bundled_ | Con la extensión Exif, _Exchangeable image information_, se puede trabajar con metadatos de imágenes (ej: funciones para leer metadatos de fotografías tomadas con cámaras digitales con la información almacenada en las cabeceras de imágenes JPEG y TIFF). Functions\: exif_imagetype, exif_read_data, exif_tagname, exif_thumbnail...|
| [GD](http://php.net/manual/es/book.image.php) / bundled | PHP también se puede usar para crear y manipular ficheros de imágenes en diferentes formatos: GIF, PNG, JPEG, WBMP, XPM. PHP puede transferir flujos de imagen directamente al navegador. Functions\: gd_info, getimagesize, imageaffine, imagearc, imagecolorat, imagecolormatch, imagecolorset, imagecopy, imagecreate, imagecreatefromgd, imagecrop, imagedestroy, imagefill, imageflip, imagegif, imagejpeg, imageline, imagescale...|
| [ImageMagick](http://php.net/manual/es/book.imagick.php) / _pecl_ | Extensión nativa de PHP para crear y modificar imágenes con la API **ImageMagick** (software para crear, editar y componer imágenes de mapa de bits). Puede leer, convertir y escribir imágenes en más de 100 formatos. **Clases**: Imagick, ImagickDraw, ImagicPixel, ImagickPixelIterator, ImagickKernel...|
| **Servicios web** | |
| [OAuth](http://php.net/manual/es/book.oauth.php) / _pecl_ | Extensión que provee un cliente y un servicio **OAuth** (protocolo de autorización basado en **HTTP** que permite a las aplicaciones garantizar el acceso a datos sin tener alojado un nombre de usuario y contraseña). **Clases**: OAuth, OAuthProvider |
| [SOAP](http://php.net/manual/es/book.soap.php) / _externas_ | La extensión **SOAP** se utiliza para escribir servidores y clientes SOAP. **Clases**: SOAPClient, SoapServer, SoapFault, SoapHeader, SoapParam, SoapVar...|
| **Manipulación XML** | |
| [DOM](http://php.net/manual/es/book.dom.php) / _externas_ | La extensión **DOM**, _Document Object Model_, permite manipular documentos XML mediante la API DOM. **Clases**: DOMAttr, DOMCdataSection, DOMCharacterData, DOMComment, DOMDocument, DOMElement, DOMEntity, DOMNode...|
| [SimpleXML](http://php.net/manual/es/book.simplexml.php) / _externas_ | Proporciona un conjunto de herramientas para **convertir XML a un objeto** que pueda ser procesado con selectores de propiedades e iteradores de arrays. **Clases**: SimpleXMLElement, SimpleXMLIterator. Functions\: simplexml_import_dom, simplexml_load_file, simplexml_load_string...|
| [Analizador XML](http://php.net/manual/es/book.xml.php) / _externas_ | Extensión que implementa el soporte para expat con herramientas para **analizar** (no validar) **documentos XML**. Functions\: utf8_encode, xml_error_string, xml_get_current_line_number, xml_get_error_code, xml_parse, xml_set_default_handler, xml_set_object|
| [XMLReader](http://php.net/manual/es/book.xmlreader.php) / _externas_ | **XMLReader** es un **analizador de XML**. El lector actúa como un cursor yendo hacia delante en el flujo del documento y deteniéndose en cada nodo. **Clases**: XMLReader|
| [XMLWriter](http://php.net/manual/es/book.xmlwriter.php) / _externas_ | Envuelve la API XMLWriter de libxml. Representa un escritor que provee medios no almacenados en caché de sólo avance para la generación de flujos o ficheros con datos XML. **Clases**: XMLWriter|
| **Otras extensiones básicas** |
| [Curl](http://php.net/manual/es/book.curl.php) / _externas_ | Librería _**libcurl**_ que permite **conectarse y comunicarse con diferentes tipos de servidores y protocolos**: http, https, _ftp_, _gopher_, _telnet_, _dict_, _file_ y _ldap_. También admite certificados HTTPS, HTTP, POST, HTTP PUT, subidas mediante FTP (también es posible hacerlo con la extensión FTP de PHP), subidas con formularios HTTP, proxies, cookies y autentificación. **Clases**: CURLFile. Functions\: curl_close, curl_errno, curl_error, curl_exec, curl_init, curl_multi_exec, curl_multi_init, curl_pause, curl_reset, curl_setopt, curl_strerror...|
| [FTP](http://php.net/manual/es/book.ftp.php) / _bundled_ | Las funciones de esta extensión implementan el acceso por parte del cliente a los servidores mediante **FTP**, el [Protocolo de Transferencia de Ficheros](http://www.faqs.org/rfcs/rfc959.html). Functions\: ftp_alloc, ftp_chdir, ftp_chmod, ftp_close, ftp_connect, ftp_exec, ftp_get, ftp_login, ftp_mkdir, ftp_put, ftp_quit, ftp_raw, ftp_rename, ftp_rmdir...|
| [LDAP](http://php.net/manual/es/book.ldap.php) / _externas_ | El **LDAP**, _Lightweight Directory Acess Protocol_, se utiliza para acceder a servidores de directorio. Un **directorio es un tipo especial de base de datos** que mantiene información en una estructura de árbol. Functions\: ldap_add, ldap, bind, ldap_close, ldap_compare, ldap_connect, ldap_delete, ldap_error, ldap_free_result, ldap_get_values, ldap_list, ldap_read...|
| [GeoIP](http://php.net/manual/es/book.geoip.php) / _pecl_ | La extensión GeoIP permite buscar la **localicación de una dirección IP** (ciudad, país, estado, longitud, latitud, ISP, tipo de conexión...). Functions\: geoip_continent_code_by_name, geoip_country_code_by_name, geoip_database_info, geoip_id_by_name, geoip_netspeedcell_by_name, geoip_region_by_name, geoip_time_zone_by_country_and_region...|
| [JSON](http://php.net/manual/es/book.json.php) / _bundled_ | Extensión que implementa el formato de intercambio de datos **JSON** [Javascript Object Notation JSON](http://www.json.org/). **Interface**: JsonSerializable. Functions:\ json_decode, json_encode, json_last_error_msg, json_last_error...|
| [Reflection](http://php.net/manual/es/book.reflection.php) / _núcleo_ | **API de reflexión** con capacidad para realizar ingeniería inversa de clases, interfaces, funciones, métodos y extensiones. Ofrece también formas de obtener comentarios de documentación de funciones, clases y métodos. **Clases**: Reflection, ReflectionClass, ReflectionExtension, ReflectionFunction, ReflectionMethod, ReflectionProperty...|
| [SPL](http://php.net/manual/es/book.spl.php) / _núcleo_ | **Biblioteca Estándar de PHP**, _Standard PHP Library_, colección de interfaces y clases pensadas para solucionar problemas comunes. Estructuras de datos, Iteradores, Interfaces, Excepciones, Manejo de ficheros. Functions\: class_implements, class_uses, iterator_count, spl_autoload_register, spl_classes... |
| [Tokenizer](http://php.net/manual/es/book.tokenizer.php) / _núcleo_ | Funciones que proporcionan una interfaz para la **creación de tokens de PHP**. Con esta extensión, integrada en el Motor Zend, se pueden escribir herramientas propias de análisis o de modificación de código fuente de PHP sin tener que hacer frente a las especificaciones del lenguaje a nivel léxico. Functions\: token_get_all, token_name|
| [URLs](http://php.net/manual/es/book.url.php) / _núcleo_ | Funciones para tratar con **URLs**: codificación, decodificación y conversión. Functions\: get_headers, get_meta_tags, parse_url, urldecode, urlencode...|
| [Memcached](http://php.net/manual/es/book.memcached.php) / _pecl_ | Sistema de alto rendimiento para el almacenamiento de objetos en caché de memoria distribuida, pensado para acelerar aplicaciones web dinámicas aliviando la carga de bases de datos. **Clases**: Memcached, MemcachedException|
| [SSH2](http://php.net/manual/es/book.ssh2.php) / _pecl_ | Enlaza a la biblioteca [libssh2](http://libssh2.org/) que provee **acceso a recursos sobre una máquina remota** utilizando una **vía de transporte criptográfica segura**. Functions\: ssh2_auth_agent, ssh2_auth_none, ssh2_connect, ssh2_exec, ssh2_fingerprint, ssh2_scp_recv, ssh2_sftp_mkdir, ssh2_sftp_rename, ssh2_sftp...|