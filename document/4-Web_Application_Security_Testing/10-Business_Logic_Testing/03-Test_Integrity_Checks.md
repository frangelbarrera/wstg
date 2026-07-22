# Probar Verificaciones de Integridad

|ID          |
|------------|
|WSTG-BUSL-03|

## Resumen

Muchas aplicaciones están diseñadas para mostrar diferentes campos dependiendo del usuario o situación dejando algunas entradas ocultas. Sin embargo, en muchos casos es posible enviar valores de campos ocultos al servidor usando un proxy. En estos casos los controles del lado del servidor deben ser lo suficientemente inteligentes para realizar ediciones relacionales o del lado del servidor para asegurar que los datos apropiados sean permitidos al servidor basados en la lógica de negocio específica del usuario y la aplicación.

Adicionalmente, la aplicación no debe depender de controles no editables, menús desplegables o campos ocultos para el procesamiento de la lógica de negocio porque estos campos permanecen no editables solo en el contexto de los navegadores. Los usuarios podrían ser capaces de editar sus valores usando herramientas de edición de proxy e intentar manipular la lógica de negocio. Si la aplicación expone valores relacionados con reglas de negocio como cantidad, etc. como campos no editables, debe mantener una copia en el lado del servidor y usar la misma para el procesamiento de la lógica de negocio. Finalmente, además de los datos de la aplicación/sistema, los sistemas de log deben estar asegurados para prevenir lectura, escritura y actualización.

Las vulnerabilidades de verificación de integridad de lógica de negocio son únicas en que estos casos de uso indebido son específicos de la aplicación y si los usuarios son capaces de hacer cambios, uno solo debería ser capaz de escribir o actualizar/editar artefactos específicos en momentos específicos según la lógica del proceso de negocio.

La aplicación debe ser lo suficientemente inteligente para verificar ediciones relacionales y no permitir a los usuarios enviar información directamente al servidor que no sea válida, confiable porque provino de controles no editables o el usuario no está autorizado para enviar a través del frontend. Adicionalmente, los artefactos del sistema tales como logs deben estar "protegidos" de lectura, escritura y eliminación no autorizada.

### Ejemplo 1

Imaginar una aplicación GUI ASP.NET que solo permite al usuario admin cambiar la contraseña para otros usuarios en el sistema. El usuario admin verá los campos de nombre de usuario y contraseña para ingresar un nombre de usuario y contraseña mientras que otros usuarios no verán ninguno de los campos. Sin embargo, si un usuario no admin envía información en los campos de nombre de usuario y contraseña a través de un proxy podrían ser capaces de "engañar" al servidor para que crea que la solicitud ha venido de un usuario admin y cambiar la contraseña de otros usuarios.

### Ejemplo 2

La mayoría de las aplicaciones web tienen listas desplegables haciendo fácil para el usuario seleccionar rápidamente su estado, mes de nacimiento, etc. Suponer que una aplicación de Gestión de Proyectos permite a los usuarios iniciar sesión y dependiendo de sus privilegios les presenta una lista desplegable de proyectos a los que tienen acceso. ¿Qué pasa si un atacante encuentra el nombre de otro proyecto al que no debería tener acceso y envía la información vía un proxy? ¿La aplicación dará acceso al proyecto? No deberían tener acceso aunque hayan saltado una verificación de lógica de negocio de autorización.

### Ejemplo 3

Suponer que el sistema de administración de vehículos motorizados requiere que un empleado inicialmente verifique la documentación e información de cada ciudadano cuando emite una identificación o licencia de conducir. En este punto el proceso de negocio ha creado datos con un alto nivel de integridad ya que la integridad de los datos enviados es verificada por los empleados. Ahora suponer que la aplicación se mueve a internet para que los empleados puedan iniciar sesión para servicio completo o los ciudadanos puedan iniciar sesión para una aplicación de autoservicio reducida para actualizar cierta información. En este punto un atacante podría ser capaz de usar un proxy de intercepción para añadir o actualizar datos a los que no debería tener acceso y podría destruir la integridad de los datos diciendo que el ciudadano no estaba casado pero proporcionando datos para el nombre del cónyuge. Este tipo de inserción o actualización de datos no verificados destruye la integridad de los datos y podría haberse prevenido si se hubiera seguido la lógica del proceso de negocio.

### Ejemplo 4

Muchos sistemas incluyen logging para propósitos de auditoría y solución de problemas. Pero, ¿qué tan buenos/válidos son los datos en estos logs? ¿Pueden ser manipulados por atacantes ya sea intencional o accidentalmente teniendo su integridad destruida?

## Objetivos de Prueba

- Revisar la documentación del proyecto para componentes del sistema que muevan, almacenen, o manejen datos.
- Determinar qué tipo de datos es lógicamente aceptable por el componente y qué tipos debería proteger contra.
- Determinar quién debería estar autorizado a modificar o leer esos datos en cada componente.
- Intentar insertar, actualizar o eliminar valores de datos usados por cada componente que no deberían estar permitidos según el flujo de trabajo de lógica de negocio.

## Cómo Probar

### Método de Prueba Específico 1

- Usando un proxy capturar tráfico HTTP buscando campos ocultos.
- Si se encuentra un campo oculto, ver cómo estos campos se comparan con la aplicación GUI y empezar a interrogar este valor a través del proxy enviando diferentes valores de datos intentando evadir el proceso de negocio y manipular valores a los que no se pretendía tener acceso.

### Método de Prueba Específico 2

- Usando un proxy capturar tráfico HTTP buscando un lugar para insertar información en áreas de la aplicación que no son editables.
- Si se encuentra, ver cómo estos campos se comparan con la aplicación GUI y empezar a interrogar este valor a través del proxy enviando diferentes valores de datos intentando evadir el proceso de negocio y manipular valores a los que no se pretendía tener acceso.

### Método de Prueba Específico 3

- Listar componentes de la aplicación o sistema que podrían ser impactados, por ejemplo logs o bases de datos.
- Para cada componente identificado, intentar leer, editar o eliminar su información. Por ejemplo, los archivos de log deberían ser identificados y los testers deberían intentar manipular los datos/información siendo recolectados.

## Casos de Prueba Relacionados

Todos los casos de prueba de [Validación de Entrada](../07-Input_Validation_Testing/README.md).

## Remediación

La aplicación debería seguir controles de acceso estrictos sobre cómo los datos y artefactos pueden ser modificados y leídos, y a través de canales confiables que aseguren la integridad de los datos. Se debería establecer logging apropiado para revisar y asegurar que no está sucediendo acceso o modificación no autorizada.

## Herramientas

- Varias herramientas de sistema/aplicación tales como editores y herramientas de manipulación de archivos.
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Referencias

- [Implementing Referential Integrity and Shared Business Logic in a RDB](https://agiledata.org/essays/referentialIntegrity.html)
- [On Rules and Integrity Constraints in Database Systems](https://www.comp.nus.edu.sg/~lingtw/papers/IST92.teopk.pdf)
- [Use referential integrity to enforce basic business rules in Oracle](https://www.techrepublic.com/article/use-referential-integrity-to-enforce-basic-business-rules-in-oracle/)
- [Maximizing Business Logic Reuse with Reactive Logic](https://dzone.com/articles/maximizing-business-logic)
