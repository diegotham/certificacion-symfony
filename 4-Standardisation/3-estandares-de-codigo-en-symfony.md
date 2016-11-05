Para contribuir en el código de Symfony se deben seguir unos estándares. Aunque no contribuyas, es mejor acostumbrarse a construir un código más universal siguiendo ciertas reglas. La **regla principal** es **imitar el código Symfony ya existente**. La mayoría de **bundles open-source** y **librerías** usadas por **Symfony** también siguen las mismas líneas.

La **principal ventaja de los estándares** es que cada pieza de código es más familiar.

El **código de ejemplo** que muestra la **documentación** oficial es el siguiente:

```
/*
 * This file is part of the Symfony package.
 *
 * (c) Fabien Potencier <fabien@symfony.com>
 *
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 */

namespace Acme;

/**
 * Coding standards demonstration.
 */
class FooBar
{
    const SOME_CONST = 42;

    private $fooBar;

    /**
     * @param string $dummy Some argument description
     */
    public function __construct($dummy)
    {
        $this->fooBar = $this->transformText($dummy);
    }

    /**
     * @param string $dummy Some argument description
     * @param array  $options
     *
     * @return string|null Transformed input
     *
     * @throws \RuntimeException
     */
    private function transformText($dummy, array $options = array())
    {
        $mergedOptions = array_merge(
            array(
                'some_default' => 'values',
                'another_default' => 'more values',
            ),
            $options
        );

        if (true === $dummy) {
            return;
        }

        if ('string' === $dummy) {
            if ('values' === $mergedOptions['some_default']) {
                return substr($dummy, 0, 5);
            }

            return ucwords($dummy);
        }

        throw new \RuntimeException(sprintf('Unrecognized dummy option "%s"', $dummy));
    }

    private function reverseBoolean($value = null, $theSwitch = false)
    {
        if (!$theSwitch) {
            return;
        }

        return !$value;
    }
}
```

### Estructura

*   Añade un espacio después de cada coma.
*   Añade un espacio después de cada operador binario (==, &&, ...) con la exceptión del operador de concatenación (.).
*   Añade operadores unarios (!, --, ...) adyacentes a la variable afectada.
*   Añade una coma después de cada array en un array multi-línea, incluso después del último.
*   Añade una línea en blanco antes de la sentencia **return**, a no ser que **return** esté sólo dentro de un grupo de sentencias (como en un **if**).
*   Usa **llaves {}** para indicar la estructura de control, independientemente del número de declaraciones que contenga.
*   Define **una clase por archivo** (esto no se aplica a **clases helper** que no se van a instanciar desde fuera).
*   Declara propiedades antes que métodos.
*   Declara métodos **public** primero, luego **protected** y finalmente los **private**. Las excepciones a esta norma son los constructores y los métodos setUp y tearDown de los tests PHPUnit, que deben ser siempre los primeros para mejorar la legibilidad.
*   Usa **paréntesis** para instanciar clases independientemente del número de argumentos que tiene el **constructor**.
*   Los **mensajes de excepción** _**string**_ deben ser concatenados con _**sprintf**._

### Documentación

*   Añade bloques **PHPDoc** para todas las clases, métodos y funciones.
*   Agrupa las anotaciones de forma que las anotaciones del mismo tipo siguen una a otra, y las anotaciones de diferente tipo están separadas por una línea vacía.
*   Omite la etiqueta **@return** si el método no devuelve nada.
*   Las anotaciones **@package** y **@subpackage** no se usan.

### Licencia

Symfony se publica bajo licencia MIT, y el bloque de licencia tiene que estar presente en la parte de arriba de cada archivo PHP, antes del namespace.