# Evasión de flujos de trabajo

|ID          |
|------------|
|WSTG-BUSL-06|

## Resumen

Las vulnerabilidades de flujo de trabajo incluyen cualquier tipo de vulnerabilidad que permita al atacante hacer un uso indebido de una aplicación o sistema de manera que pueda evadir (no seguir) el flujo de trabajo diseñado o previsto.

[Definición de flujo de trabajo en Wikipedia](https://en.wikipedia.org/wiki/Workflow):

> Un flujo de trabajo consiste en una secuencia de pasos conectados, en la que cada paso sigue sin demora ni interrupción y termina justo antes de que pueda comenzar el paso siguiente. Es una representación de una secuencia de operaciones, definidas como el trabajo de una persona o un grupo, una organización de personal o uno o más mecanismos simples o complejos. El flujo de trabajo puede considerarse cualquier abstracción del trabajo real.

La lógica de negocio de la aplicación debe exigir que el usuario complete pasos específicos en el orden correcto y previsto. Si el flujo de trabajo se interrumpe sin completarse correctamente, todas las acciones y acciones derivadas deben «revertirse» o cancelarse. Las vulnerabilidades relacionadas con la evasión de flujos de trabajo o con eludir el flujo de trabajo correcto de la lógica de negocio son particulares porque dependen mucho de la aplicación o el sistema; para identificarlas, es necesario desarrollar cuidadosamente casos de uso indebido manuales a partir de los requisitos y casos de uso.

El proceso de negocio de la aplicación debe incluir controles para garantizar que las transacciones o acciones del usuario se realicen en el orden correcto y aceptable. Si una transacción desencadena algún tipo de acción, esta debe «revertirse» y eliminarse si la transacción no se completa correctamente.

### Ejemplo 1

Muchos recibimos algún tipo de «puntos de fidelidad» por las compras en supermercados y estaciones de servicio. Supongamos que un usuario puede iniciar una transacción vinculada a su cuenta y, después de que se hayan añadido puntos a su cuenta de fidelidad, cancelar la transacción o quitar artículos de su «carrito» y finalizar el pago. En este caso, el sistema no debería aplicar puntos o créditos a la cuenta hasta que se finalice el pago, o debería «revertirlos» si el incremento de puntos o créditos no coincide con el pago final. En ese caso, un atacante podría iniciar transacciones y cancelarlas para acumular puntos sin comprar nada.

### Ejemplo 2

Un sistema electrónico de tablón de anuncios podría estar diseñado para garantizar que las publicaciones iniciales no contengan palabras obscenas, comparando cada publicación con una lista. Si el texto introducido por el usuario contiene una palabra incluida en una lista de bloqueo, la publicación no se publica. Sin embargo, una vez publicada, el autor puede acceder a ella, editarla y modificar su contenido para incluir palabras de la lista de obscenidades o de bloqueo, ya que la publicación no se vuelve a comparar durante la edición. Teniendo esto en cuenta, los atacantes podrían crear una discusión inicial en blanco o con contenido mínimo y luego añadir lo que quieran en una actualización.

## Objetivos de prueba

- Revisar la documentación del proyecto para encontrar métodos que permitan omitir pasos del proceso de la aplicación o realizarlos en un orden distinto al previsto por la lógica de negocio.
- Desarrollar un caso de uso indebido e intentar evadir cada flujo lógico identificado.

## Cómo probar

### Método 1

- Iniciar una transacción y avanzar en la aplicación hasta superar los puntos que activan la asignación de créditos o puntos a la cuenta del usuario.
- Cancelar la transacción o reducir el pago final, de modo que deban disminuir los valores de puntos, y revisar el sistema de puntos o créditos para confirmar que se hayan registrado correctamente.

### Método 2

- En un sistema de gestión de contenido o tablón de anuncios, introducir y guardar texto o valores iniciales válidos.
- Luego, intentar añadir, editar y eliminar datos que dejen los datos existentes en un estado no válido o con valores no válidos, para comprobar que no se permita guardar la información incorrecta. Algunos datos o información «no válidos» pueden consistir en palabras específicas (obscenidades) o temas específicos (como cuestiones políticas).

### Método 3: Abuso de flujos de trabajo de varios pasos y transiciones de estado

Las aplicaciones modernas suelen implementar flujos de trabajo que abarcan varias solicitudes, API o servicios backend.
En esos casos, el estado del flujo de trabajo puede inferirse a partir de parámetros controlados por el cliente o del estado de un sistema distribuido, en lugar de aplicarse estrictamente en el servidor.

Los atacantes pueden intentar abusar de estos flujos de trabajo omitiendo pasos obligatorios, reordenando solicitudes o manipulando transiciones de estado para alcanzar estados de la aplicación no autorizados o no previstos.

Entre los patrones comunes de abuso de flujos de trabajo y transiciones de estado se incluyen:

- Omitir pasos obligatorios del flujo de trabajo invocando directamente funciones de etapas posteriores
- Reordenar solicitudes para eludir los controles de secuencia
- Reproducir o reutilizar identificadores de estado en distintos pasos del flujo de trabajo
- Manipular parámetros de estado, fase o paso proporcionados por el cliente
- Ejecutar acciones dependientes en paralelo para eludir la validación de secuencia

Pasos de prueba:

- Identificar los flujos de trabajo de varios pasos y las funciones implicadas en cada etapa.
- Capturar las solicitudes asociadas a cada paso del flujo de trabajo y anotar los parámetros de estado.
- Intentar invocar funciones de etapas posteriores sin completar los pasos previos obligatorios.
- Reproducir o reordenar solicitudes para determinar si se exige una secuencia.
- Modificar los parámetros de estado para asignarles valores no válidos, futuros o no autorizados.
- Verificar si la aplicación valida el estado del flujo de trabajo en el servidor.

Algunos ejemplos son:

- Activar acciones de envío o preparación de pedidos sin haber confirmado el pago.
- Acceder a funciones restringidas o prémium invocando directamente los endpoints de activación o habilitación.

## Casos de prueba relacionados

- [Pruebas de directory traversal e inclusión de archivos](../05-Authorization/01-Directory_Traversal_File_Include.md)
- [Evasión del esquema de autorización](../05-Authorization/02-Bypassing_Authorization_Schema.md)
- [Evasión del esquema de gestión de sesiones](../06-Session_Management/01-Session_Management_Schema.md)
- [Probar la validación de datos de la lógica de negocio](01-Business_Logic_Data_Validation.md)
- [Probar la capacidad de falsificar solicitudes](02-Ability_to_Forge_Requests.md)
- [Probar los controles de integridad](03-Integrity_Checks.md)
- [Probar el tiempo de procesamiento](04-Process_Timing.md)
- [Probar los límites del número de veces que se puede usar una función](05-Number_of_Times_a_Function_Can_Be_Used_Limits.md)
- [Probar las defensas contra el uso indebido de la aplicación](07-Defenses_Against_Application_Misuse.md)
- [Probar la carga de tipos de archivo inesperados](08-Upload_of_Unexpected_File_Types.md)
- [Probar la carga de archivos maliciosos](09-Upload_of_Malicious_Files.md)

## Remediación

La aplicación debe conocer su propio estado e incluir controles que garanticen que los usuarios completen cada paso del flujo de trabajo en el orden correcto, y que impidan a los atacantes evadir, omitir o repetir cualquier paso o proceso del flujo de trabajo. Para probar las vulnerabilidades de flujo de trabajo, se deben desarrollar casos de abuso o uso indebido de la lógica de negocio con el objetivo de completar correctamente el proceso de negocio sin realizar los pasos adecuados en el orden correcto.

## Referencias

- [OWASP Abuse Case Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Abuse_Case_Cheat_Sheet.html)
- [CWE-840: Business Logic Errors](https://cwe.mitre.org/data/definitions/840.html)