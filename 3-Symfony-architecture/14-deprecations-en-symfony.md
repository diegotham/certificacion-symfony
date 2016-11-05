De vez en cuando algunas clases o métodos quedan **obsoletos** en el framework. Esto ocurre cuando la implementación de una funcionalidad no puede cambiarse por problemas de compatibilidad, pero la mejor alternativa es implementar funcionalidades como **deprecated**.

Una característica se marca como deprecated añadiendo **@deprecated** en **phpdoc** para clases relevantes, métodos, propiedades, etc:

```
/**
* @deprecated Deprecated since version 2.8, to be removed in 3.0\. Use XXX instead.
*/
```

El **mensaje deprecation** debe indicar la versión desde que la clase o método han sido marcados como deprecated, la versión en la que será removida por completo, y siempre que se pueda, cómo se reemplazará.

Un error PHP **E_USER_DEPRECATED** debe también ser lanzado para ayudar a los desarrolladores con las migraciones, comenzando desde una o dos versiones minor antes de que la funcionalidad sea removida (dependiendo de la importancia del cambio):

```
@trigger_error('XXX() is deprecated since version 2.8 and will be removed in 3.0\. Use XXX instead.', E_USER_DEPRECATED);
```

Sin el operador de silencio **@**, los desarrolladores necesitarían quitar los deprecation notices. El silenciarlos cambia este comportamiento y permite a los desarrolladores incluirlos una vez ya están preparados para ello (añadiendo un **error handler** personalizado como el usado en la **Web Debug Toolbar** o en el **bridge PHPUnit**).

En Symfony existe una herramienta muy útil llamada [Deprecation Detector](https://github.com/sensiolabs-de/deprecation-detector). Esta aplicación de consola inicia un análisis en el código fuente de tu proyecto para encontrar usos de métodos, clases, interfaces y services deprecated. Específicamente, identifica el uso de código deprecated gracias a la anotación **@deprecated**.