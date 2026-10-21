# Comprobaciones de integridad

|ID          |
|------------|
|WSTG-BUSL-03|

## Resumen

Muchas aplicaciones están diseñadas para mostrar distintos campos según el usuario o la situación, por lo que dejan algunas entradas ocultas. Sin embargo, en muchos casos es posible enviar valores de campos ocultos al servidor mediante un proxy. En estos casos, los controles del lado del servidor deben ser lo suficientemente inteligentes para realizar modificaciones relacionales o del lado del servidor que garanticen que solo se permitan enviar al servidor los datos adecuados, de acuerdo con la lógica de negocio específica del usuario y de la aplicación.

Además, la aplicación no debe depender de controles no editables, menús desplegables ni campos ocultos para procesar la lógica de negocio, porque estos campos solo son no editables en el contexto de los navegadores. Los usuarios podrían editar sus valores con herramientas de edición de proxy e intentar manipular la lógica de negocio. Si la aplicación expone valores relacionados con reglas de negocio, como la cantidad, en campos no editables, debe mantener una copia en el lado del servidor y usarla para procesar la lógica de negocio. Por último, además de los datos de la aplicación o del sistema, los sistemas de registro deben protegerse para impedir su lectura, escritura y actualización.

Las vulnerabilidades de comprobación de integridad de la lógica de negocio son particulares porque estos casos de uso indebido dependen de cada aplicación y, si los usuarios pueden hacer cambios, solo deberían poder escribir o actualizar/editar determinados artefactos en momentos específicos, según la lógica del proceso de negocio.

La aplicación debe ser lo suficientemente inteligente para comprobar las modificaciones relacionales y no permitir que los usuarios envíen directamente al servidor información que no sea válida, que no sea confiable por provenir de controles no editables o que el usuario no esté autorizado a enviar a través del frontend. Además, los artefactos del sistema, como los registros, deben estar «protegidos» contra la lectura, escritura y eliminación no autorizadas.

### Ejemplo 1

Imaginemos una aplicación GUI ASP.NET que solo permite al usuario administrador cambiar las contraseñas de otros usuarios del sistema. El administrador verá los campos de nombre de usuario y contraseña para introducir un nombre de usuario y una contraseña, mientras que los demás usuarios no verán ninguno de esos campos. Sin embargo, si un usuario que no es administrador envía información en los campos de nombre de usuario y contraseña a través de un proxy, podría «engañar» al servidor para que crea que la solicitud proviene de un administrador y cambiar las contraseñas de otros usuarios.

### Ejemplo 2

La mayoría de las aplicaciones web tienen listas desplegables que facilitan al usuario seleccionar rápidamente su estado, mes de nacimiento, etc. Supongamos que una aplicación de gestión de proyectos permite a los usuarios iniciar sesión y, según sus privilegios, les presenta una lista desplegable de los proyectos a los que tienen acceso. ¿Qué ocurre si un atacante descubre el nombre de otro proyecto al que no debería tener acceso y envía esa información mediante un proxy? ¿Le dará la aplicación acceso al proyecto? No debería tener acceso, aunque haya omitido una comprobación de autorización de la lógica de negocio.

### Ejemplo 3

Supongamos que el sistema de administración de vehículos motorizados requiere que un empleado verifique inicialmente la documentación y la información de cada ciudadano cuando se emite una identificación o una licencia de conducir. En este punto, el proceso de negocio ha creado datos con un alto nivel de integridad, ya que los empleados comprueban la integridad de los datos enviados. Supongamos ahora que la aplicación se traslada a Internet para que los empleados puedan iniciar sesión y acceder a todos los servicios, o para que los ciudadanos puedan iniciar sesión en una aplicación de autoservicio con funciones limitadas y actualizar cierta información. En este punto, un atacante podría usar un proxy de interceptación para añadir o actualizar datos a los que no debería tener acceso, y destruir la integridad de los datos al indicar que el ciudadano no estaba casado, pero proporcionar datos para el nombre de su cónyuge. Este tipo de inserción o actualización de datos no verificados destruye la integridad de los datos y podría haberse evitado si se hubiera seguido la lógica del proceso de negocio.

### Ejemplo 4

Muchos sistemas incluyen registros con fines de auditoría y resolución de problemas. Pero ¿qué tan buena o válida es la información de estos registros? ¿Pueden los atacantes manipularla, intencional o accidentalmente, y destruir su integridad?

## Objetivos de prueba

- Revisar la documentación del proyecto para identificar los componentes del sistema que trasladan, almacenan o gestionan datos.
- Determinar qué tipos de datos son lógicamente aceptables para el componente y contra cuáles debería protegerse el sistema.
- Determinar quién debería tener permiso para modificar o leer esos datos en cada componente.
- Intentar insertar, actualizar o eliminar valores de datos utilizados por cada componente que no deberían permitirse según el flujo de trabajo de la lógica de negocio.

## Cómo probar

### Método de prueba específico 1

- Usar un proxy para capturar el tráfico HTTP y buscar campos ocultos.
- Si se encuentra un campo oculto, comparar estos campos con la aplicación GUI y empezar a examinar su valor a través del proxy, enviando distintos valores de datos e intentando eludir el proceso de negocio y manipular valores a los que no se tenía previsto tener acceso.

### Método de prueba específico 2

- Usar un proxy para capturar el tráfico HTTP y buscar un lugar donde insertar información en áreas no editables de la aplicación.
- Si se encuentra uno, comparar estos campos con la aplicación GUI y empezar a examinar su valor a través del proxy, enviando distintos valores de datos e intentando eludir el proceso de negocio y manipular valores a los que no se tenía previsto tener acceso.

### Método de prueba específico 3

- Enumerar los componentes de la aplicación o del sistema que podrían verse afectados, como los registros o las bases de datos.
- Para cada componente identificado, intentar leer, editar o eliminar su información. Por ejemplo, se deben identificar los archivos de registro y los evaluadores deben intentar manipular los datos o la información que se recopila.

## Casos de prueba relacionados

Todos los casos de prueba de [pruebas de inyección](../07-Injection/README.md).

## Remediación

La aplicación debe aplicar controles de acceso estrictos que rijan cómo pueden modificarse y leerse los datos y artefactos, y utilizar canales confiables que garanticen la integridad de los datos. Se debe implementar un registro adecuado para revisar y garantizar que no se produzcan accesos ni modificaciones no autorizados.

## Herramientas

- Diversas herramientas de sistema o aplicación, como editores y herramientas de manipulación de archivos.
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## Referencias

- [Implementación de la integridad referencial y de la lógica de negocio compartida en una RDB](https://agiledata.org/essays/referentialIntegrity.html)
- [Reglas y restricciones de integridad en sistemas de bases de datos](https://www.comp.nus.edu.sg/~lingtw/papers/IST92.teopk.pdf)
- [Uso de la integridad referencial para aplicar reglas de negocio básicas en Oracle](https://web.archive.org/web/20160423081903/http://www.techrepublic.com/article/use-referential-integrity-to-enforce-basic-business-rules-in-oracle/)
- [Maximización de la reutilización de la lógica de negocio con lógica reactiva](https://web.archive.org/web/20221116203934/https://dzone.com/articles/maximizing-business-logic)