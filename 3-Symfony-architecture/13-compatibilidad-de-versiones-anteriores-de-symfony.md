Asegurarse de que las actualizaciones se pueden realizar bien es una de las prioridades de SensioLabs. Prometen backwards compatibility (BC) para todos los lanzamientos de versiones minor. Este estrategia es del tipo [Semantic Versioning](http://semver.org/), y significa que sólo las **versiones mayor** (2.0, 3.0, etc) pueden romper con la **compatibilidad de versiones**. Las **versiones minor** (como 2.5, 2.6, etc) pueden introducir nuevas características, pero deben hacerlo sin romper el API existente.

Sin embargo, la compatibilidad entre versiones es aplicable de formas muy diferentes. De hecho casi cada cambio que se hace en el framework puede potencialmente romper una aplicación. Si añaden un método a una clase, esto romperá una aplicación que extendía esta clase y añadía el mismo método, pero con diferente nombre.

Además, no todos los BC breaks tienen el mismo impacto en el código de una aplicación. Mientras algunos BC breaks requieren que hagas cambios significativos a tus clases o arquitectura, otros se solucionan de forma tan fácil como cambiando el nombre del método.

Por ello se ha creado la guía de compatibilidad de versiones. La sección "Usando el código de Symfony" muestra cómo puedes asegurar que tu aplicación no se romperá por completo cuando se actualice a una nueva versión de una misma versión mayor.

La segunda sección, "Modificando el código de Symfony" va dirigida a los contribuyentes de Symfony. Esta sección detalla reglas que cada contribuyente tiene que seguir para asegurar actualizaciones cómodas a los usuarios.

### Usando el código de Symfony

Las siguientes pautas ayudan a asegurar actualizaciones seguras en las futuras versiones minor de Symfony.

#### Interfaces

Todas las interfaces de Symfony pueden usarse en [type hints](http://diego.com.es/type-hinting-en-php). Puedes también llamar a cualquiera de los métodos que declaran. Desde Symfony garantizan que no romperán código que tiene estas reglas. 

La excepción a esta norma es que las interfaces etiquetadas con @Interval no deberían usarse o implementarse.

La siguiente tabla explica los casos que respetan la compatibilidad de versiones:

| | |
| -------- | -------- |
| **Uso** | **Compatibilidad de versiones** |
| **Si tu...** | **Desde Symfony garantizan BC** |
| Haces type hint en la interface | Sí |
| Llamas a un método | Sí |
| **Si implementas una interface y...** | **Desde Symfony garantizan BC** |
| Implementas un método | Sí |
| Añades un argumento a un método implementado | Sí |
| Añades un valor por defecto a un argumento | Sí |

#### Clases

Todas las clases de **Symfony** pueden instanciarse y pueden ser accedidas a través de sus métodos y propiedades públicos.

Las clases, propiedades y métodos con la etiqueta **@internal** así como las clases localizadas en los namespaces ***\\Tests\\** son una excepción a esta norma. Están construidas para uso interno sólamente y no deben ser accedidas desde tu propio código.

La siguiente tabla explica los casos que respetan la compatibilidad de versiones:

| | |
| -------- | -------- |
| **Uso** | **Compatibilidad de versiones** |
| **Si tu...** | **Desde Symfony garantizan...** |
| Haces type hint en la clase | Sí |
| Creas una nueva instancia | Sí |
| Extiendes la clase | Sí |
| Accedes a una propiedad pública | Sí |
| Llamas a un método público | Sí |
| **Si extiendes la clase y...** | **Desde Symfony garantizan...** |
| Accedes a una propiedad protected | Sí |
| Llamas a un método protected | Sí |
| Sobreescribes una propiedad public | Sí |
| Sobreescribes una propiedad protected | Sí |
| Sobreescribes un método public | Sí |
| Sobreescribes un método protected | Sí |
| Añades una nueva propiedad | No |
| Añades un nuevo método | No |
| Añades un argumento a un método sobreescrito | Sí |
| Añades un valor por defecto a un argumento | Sí |
| Llamas a un método private con Reflection | No |
| Accedes a una propiedad private con Reflection | No |

### Modificando el código de Symfony

Si quieres ayudas a mejorar **Symfony**, es necesario respetar las normas de [este enlace](http://symfony.com/doc/current/contributing/code/bc.html#working-on-symfony-code) para garantizar las actualizaciones seguras.