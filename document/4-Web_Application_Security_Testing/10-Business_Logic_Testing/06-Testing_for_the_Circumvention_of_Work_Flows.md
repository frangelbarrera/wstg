# Pruebas de Evasión de Flujos de Trabajo

|ID          |
|------------|
|WSTG-BUSL-06|

## Resumen

Las vulnerabilidades de flujo de trabajo involucran cualquier tipo de vulnerabilidad que permita al atacante usar indebida una aplicación/sistema de una manera que les permita evadir (no seguir) el flujo de trabajo diseñado/previsto.

[Definición de workflow en Wikipedia](https://en.wikipedia.org/wiki/Workflow):

> Un flujo de trabajo consiste en una secuencia de pasos conectados donde cada paso sigue sin demora o brecha y termina justo antes de que el paso subsiguiente pueda comenzar. Es una representación de una secuencia de operaciones, declaradas como trabajo de una persona o grupo, una organización de personal, o uno o más mecanismos simples o complejos. El flujo de trabajo puede verse como cualquier abstracción de trabajo real.

La lógica de negocio de la aplicación debe requerir que el usuario complete pasos específicos en el orden correcto/específico y si el flujo de trabajo se termina sin completarse correctamente, todas las acciones y acciones generadas se "revierten" o cancelan. Las vulnerabilidades relacionadas con la evasión de flujos de trabajo o evasión del flujo de trabajo correcto de lógica de negocio son únicas en que son muy específicas de la aplicación/sistema y se deben desarrollar casos de uso indebido manuales cuidadosos usando requisitos y casos de uso.

El proceso de negocio de las aplicaciones debe tener verificaciones para asegurar que las transacciones/acciones del usuario procedan en el orden correcto/aceptable y si una transacción dispara algún tipo de acción, esa acción será "revertida" y eliminada si la transacción no se completa exitosamente.

### Ejemplo 1

Muchos de nosotros recibimos algún tipo de "puntos de club/lealtad" por compras de tiendas de comestibles y estaciones de gasolina. Suponer que un usuario fue capaz de iniciar una transacción vinculada a su cuenta y luego después de que los puntos se han añadido a su cuenta de club/lealtad cancelar la transacción o remover ítems de su "canasta" y pagar. En este caso el sistema o no debería aplicar puntos/créditos a la cuenta hasta que se pague o los puntos/créditos deberían ser "revertidos" si el incremento de punto/crédito no coincide con el pago final. Con esto en mente, un atacante podría iniciar transacciones y cancelarlas para construir sus niveles de punto sin realmente comprar nada.

### Ejemplo 2

Un sistema de tablero de anuncios electrónico podría estar diseñado para asegurar que las publicaciones iniciales no contengan profanidad basada en una lista contra la cual se compara la publicación. Si se encuentra una palabra en una lista de bloqueo en el texto ingresado por el usuario, el envío no se publica. Pero, una vez que un envío se publica el remitente puede acceder, editar, y cambiar el contenido del envío para incluir palabras incluidas en la lista de profanidad/bloqueo ya que en la edición la publicación nunca se compara de nuevo. Teniendo esto en mente, los atacantes podrían abrir una discusión inicial en blanco o mínima y luego añadir lo que quieran como actualización.

## Objetivos de Prueba

- Revisar la documentación del proyecto para métodos de saltar o ir a través de pasos en el proceso de la aplicación en un orden diferente del flujo de lógica de negocio previsto.
- Desarrollar un caso de uso indebido e intentar evadir cada flujo lógico identificado.

## Cómo Probar

### Método de Prueba 1

- Iniciar una transacción yendo a través de la aplicación más allá de los puntos que disparan créditos/puntos a la cuenta del usuario.
- Cancelar la transacción o reducir el pago final para que los valores de punto deban disminuirse y verificar el sistema de puntos/crédito para asegurar que los puntos/créditos apropiados se registraron.

### Método de Prueba 2

- En un sistema de gestión de contenido o tablero de anuncios ingresar y guardar texto o valores iniciales válidos.
- Luego intentar añadir, editar y remover datos que dejarían los datos existentes en un estado inválido o con valores inválidos para asegurar que el usuario no tenga permitido guardar la información incorrecta. Algunos datos o información "inválida" podrían ser palabras específicas (profanidad) o temas específicos (tales como temas políticos).

### Método de Prueba 3: Abuso de Flujo de Trabajo Multi-Paso y Transición de Estado

Las aplicaciones modernas a menudo implementan flujos de trabajo que abarcan múltiples solicitudes, APIs o servicios backend.
En tales casos, el estado del flujo de trabajo podría inferirse de parámetros controlados por el cliente o estado de sistema distribuido en lugar de ser estrictamente impuesto del lado del servidor.

Los atacantes podrían intentar abusar de estos flujos de trabajo saltando pasos requeridos, reordenando solicitudes, o manipulando transiciones de estado para alcanzar estados de aplicación no autorizados o no previstos.

Patrones comunes de abuso de flujo de trabajo y transición de estado incluyen:

- Saltar pasos requeridos del flujo de trabajo invocando directamente funcionalidad de etapa posterior
- Reordenar solicitudes para evadir controles de secuencia impuestos
- Reproducir o reusar identificadores de estado a través de pasos del flujo de trabajo
- Manipular parámetros de estado, fase o paso proporcionados por el cliente
- Ejecutar acciones dependientes en paralelo para evadir validación de secuencia

Pasos de Prueba:

- Identificar flujos de trabajo multi-paso y la funcionalidad involucrada en cada etapa.
- Capturar solicitudes asociadas con cada paso del flujo de trabajo y notar cualquier parámetro de estado o estado.
- Intentar invocar funcionalidad de etapa posterior sin completar pasos prerrequisitos.
- Reproducir o reordenar solicitudes para determinar si existe imposición de secuencia.
- Modificar parámetros de estado o estado a valores inválidos, futuros, o no autorizados.
- Verificar si la aplicación impone validación de estado de flujo de trabajo del lado del servidor.

Ejemplos incluyen:

- Disparar acciones de envío o cumplimiento de orden sin un paso de pago confirmado.
- Acceder a características restringidas o premium invocando directamente endpoints de activación o habilitación.

## Casos de Prueba Relacionados

- [Pruebas de Directory Traversal/File Include](../05-Authorization_Testing/01-Testing_Directory_Traversal_File_Include.md)
- [Pruebas de Evasión de Esquema de Autorización](../05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema.md)
- [Pruebas de Evasión de Esquema de Gestión de Sesión](../06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md)
- [Probar la Validación de Datos de Lógica de Negocio](01-Test_Business_Logic_Data_Validation.md)
- [Probar la Capacidad de Falsificar Solicitudes](02-Test_Ability_to_Forge_Requests.md)
- [Probar Verificaciones de Integridad](03-Test_Integrity_Checks.md)
- [Probar el Tiempo del Proceso](04-Test_for_Process_Timing.md)
- [Probar Límites en el Número de Veces que una Función Puede Ser Usada](05-Test_Number_of_Times_a_Function_Can_Be_Used_Limits.md)
- [Probar Defensas contra el Uso Indebido de la Aplicación](07-Test_Defenses_Against_Application_Misuse.md)
- [Probar la Subida de Tipos de Archivo Inesperados](08-Test_Upload_of_Unexpected_File_Types.md)
- [Probar la Subida de Archivos Maliciosos](09-Test_Upload_of_Malicious_Files.md)

## Remediación

La aplicación debe ser auto-consciente y tener verificaciones en su lugar asegurando que los usuarios completen cada paso en el proceso de flujo de trabajo en el orden correcto y prevenir a los atacantes de evadir/saltar/o repetir cualquier paso/proceso en el flujo de trabajo. Las pruebas de vulnerabilidades de flujo de trabajo involucran desarrollar casos de abuso/uso indebido de lógica de negocio con el objetivo de completar exitosamente el proceso de negocio mientras no se completan los pasos correctos en el orden correcto.

## Referencias

- [OWASP Abuse Case Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Abuse_Case_Cheat_Sheet.html)
- [CWE-840: Business Logic Errors](https://cwe.mitre.org/data/definitions/840.html)
