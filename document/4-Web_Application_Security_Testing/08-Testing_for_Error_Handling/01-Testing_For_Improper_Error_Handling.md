# Pruebas de Manejo Improper de Errores

|ID          |
|------------|
|WSTG-ERRH-01|

## Resumen

Todos los tipos de aplicaciones (apps web, servidores web, bases de datos, etc.) generarán errores por varias razones. Los desarrolladores a menudo ignoran el manejo de estos errores, o descartan la idea de que un usuario intentará disparar un error intencionalmente (*por ejemplo* enviar una cadena donde se espera un entero). Cuando el desarrollador solo considera el camino feliz, se olvidan de todas las demás posibles entradas del usuario que el código puede recibir pero no puede manejar.

Los errores a veces surgen como:

- Trazas de pila
- Timeouts de red
- Desajuste de entrada
- Volcados de memoria

El manejo improper de errores puede permitir a los atacantes:

- Entender las APIs siendo usadas internamente.
- Mapear los varios servicios integrándose entre sí ganando entendimiento sobre sistemas internos y frameworks usados, lo cual abre puertas al encadenamiento de ataques.
- Recolectar las versiones y tipos de aplicaciones siendo usadas.
- Identificar rutas del sistema de archivos.
- Causar una denegación de servicio forzando al sistema a un deadlock o una excepción no manejada que envía una señal de pánico al motor que lo ejecuta.
- Evasión de controles donde una cierta excepción no está restringida por la lógica establecida alrededor del camino feliz.

## Objetivos de Prueba

- Identificar salida de error existente.
- Analizar las diferentes salidas devueltas.

## Cómo Probar

Los errores usualmente se ven como benignos ya que proporcionan datos de diagnóstico y mensajes que podrían ayudar al usuario a entender el problema en mano, o para que el desarrollador depure ese error.

Al intentar enviar datos inesperados, o forzar al sistema a ciertos casos límite y escenarios, el sistema o aplicación, la mayoría del tiempo, dará un poco sobre lo que está sucediendo internamente, a menos que los desarrolladores hayan apagado todos los posibles errores y devuelvan un cierto mensaje personalizado.

### Servidores Web

Todas las apps web se ejecutan en un servidor web, ya sea integrado o uno completo. Las apps web deben manejar y analizar solicitudes HTTP, y para eso un servidor web siempre es parte del stack. Algunos de los servidores web más comunes son Nginx, Apache e IIS.

Los servidores web tienen mensajes y formatos de error conocidos. Si no se está familiarizado con cómo se ven, buscar en línea proporcionaría ejemplos. Otra manera sería buscar en su documentación, o simplemente configurar un servidor localmente y descubrir los errores yendo a través de las páginas que el servidor web usa.

Para disparar mensajes de error, un tester debe:

- Buscar archivos y carpetas aleatorios que no se encontrarán (404s).
- Intentar solicitar carpetas que existen y ver el comportamiento del servidor (403s, página en blanco, o listado de directorio).
- Intentar enviar una solicitud que rompa el [RFC HTTP](https://tools.ietf.org/html/rfc7231). Un ejemplo sería enviar una ruta muy larga, romper el formato de encabezados, o cambiar la versión HTTP.
    - Incluso si los errores se manejan a nivel de aplicación, romper el RFC HTTP podría hacer que el servidor web integrado se muestre ya que tiene que manejar la solicitud, y los desarrolladores olvidan sobrescribir estos errores.

### Aplicaciones

Las aplicaciones son las más susceptibles de dejar salir una amplia variedad de mensajes de error, los cuales incluyen: trazas de pila, volcados de memoria, excepciones mal manejadas, y errores genéricos. Esto sucede debido al hecho de que las aplicaciones son construidas a medida la mayoría del tiempo y los desarrolladores necesitan observar y manejar todos los posibles casos de error (o tener un mecanismo global de captura de errores), y estos errores pueden aparecer de integraciones con otros servicios.

Para hacer que una aplicación lance estos errores, un tester debe:

1. Identificar posibles puntos de entrada donde la aplicación está esperando datos.
2. Analizar el tipo de entrada esperado (cadenas, enteros, JSON, XML, etc.).
3. Fuzzear cada punto de entrada basado en los pasos previos para tener un escenario de prueba más enfocado.
   - Fuzzear cada entrada con todas las posibles inyecciones no es la mejor solución a menos que se tenga tiempo de prueba ilimitado y la aplicación pueda manejar esa cantidad de entrada.
   - Si el fuzzing no es una opción, seleccionar manualmente entradas viables que tengan la mayor probabilidad de romper un cierto parser (*por ejemplo* un corchete de cierre para un cuerpo JSON, un texto grande donde solo se esperan un par de caracteres, inyección CRLF con parámetros que podrían ser analizados por servidores y controles de validación de entrada, caracteres especiales que no son aplicables para nombres de archivo, etc.).
   - El fuzzing con datos jerga debería ejecutarse para cada tipo ya que a veces los intérpretes se romperán fuera del manejo de excepciones del desarrollador.
4. Entender el servicio respondiendo con el mensaje de error e intentar hacer una lista de fuzz más refinada para sacar más información o detalles de error de ese servicio (podría ser una base de datos, un servicio standalone, etc.).

Los mensajes de error a veces son la debilidad principal en el mapeo de sistemas, especialmente bajo una arquitectura de microservicios. Si los servicios no están propiamente establecidos para manejar errores de manera genérica y uniforme, los mensajes de error permitirían a un tester identificar qué servicio maneja qué solicitudes, y permite un ataque más enfocado por servicio.

> El tester necesita mantener un ojo vigilante sobre el tipo de respuesta. A veces los errores se devuelven como éxito con un cuerpo de error, ocultan el error en un 302, o simplemente teniendo una manera personalizada de representar ese error.

## Remediación

Para remediación, revisar los [Proactive Controls C10](https://owasp.org/www-project-proactive-controls/v3/en/c10-errors-exceptions) y la [Hoja de Referencia de Manejo de Errores](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html).

## Playgrounds

- [Juice Shop - Error Handling](https://pwning.owasp-juice.shop/companion-guide/latest/part2/security-misconfiguration.html#provoke-an-error-that-is-neither-very-gracefully-nor-consistently-handled)

## Referencias

- [WSTG: Apéndice C - Fuzzing](../../6-Appendix/C-Fuzzing.md)
- [Proactive Controls C10: Handle All Errors and Exceptions](https://owasp.org/www-project-proactive-controls/v3/en/c10-errors-exceptions)
- [ASVS v4.1 v7.4: Error handling](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x15-V7-Error-Logging.md#v74-error-handling)
- [CWE 728 - Improper Error Handling](https://cwe.mitre.org/data/definitions/728.html)
- [Cheat Sheet Series: Error Handling](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)
