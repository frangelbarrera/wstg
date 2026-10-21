# Manejo inadecuado de errores

|ID          |
|------------|
|WSTG-ERRH-01|

## Resumen

Todos los tipos de aplicaciones (aplicaciones web, servidores web, bases de datos, etc.) generan errores por diversos motivos. A menudo, los desarrolladores ignoran el manejo de estos errores o descartan la posibilidad de que un usuario intente provocar uno intencionalmente (*por ejemplo*, enviar una cadena donde se espera un entero). Cuando el desarrollador solo considera el caso de éxito, olvida todas las demás entradas posibles que el usuario puede proporcionar y que el código no puede manejar.

Los errores a veces se presentan como:

- Trazas de pila
- Tiempos de espera de red
- Incompatibilidad de entradas
- Volcados de memoria

El manejo inadecuado de errores puede permitir a los atacantes:

- Entender las API que se utilizan internamente.
- Mapear los distintos servicios que se integran entre sí al obtener información sobre los sistemas internos y los frameworks utilizados, lo que abre la puerta al encadenamiento de ataques.
- Recopilar las versiones y los tipos de aplicaciones que se utilizan.
- Identificar rutas del sistema de archivos.
- Provocar una denegación de servicio al forzar al sistema a entrar en un interbloqueo o a generar una excepción no controlada que envía una señal de pánico al motor que lo ejecuta.
- Evadir controles cuando una excepción determinada no está restringida por la lógica establecida para el caso de éxito.

## Objetivos de la prueba

- Identificar las salidas de error existentes.
- Analizar las distintas salidas devueltas.

## Cómo probar

Los errores suelen considerarse inocuos, ya que proporcionan datos de diagnóstico y mensajes que pueden ayudar al usuario a comprender el problema en cuestión o al desarrollador a depurar el error.

Al intentar enviar datos inesperados o forzar al sistema a enfrentarse a determinados casos límite y escenarios, el sistema o la aplicación suele revelar algo de lo que ocurre internamente, a menos que los desarrolladores hayan desactivado todos los errores posibles y devuelvan un mensaje personalizado.

### Servidores web

Todas las aplicaciones web se ejecutan en un servidor web, ya sea integrado o independiente. Las aplicaciones web deben manejar y analizar solicitudes HTTP, por lo que un servidor web siempre forma parte de la pila. Algunos de los servidores web más comunes son Nginx, Apache e IIS.

Los servidores web tienen mensajes y formatos de error conocidos. Si no se sabe cómo son, una búsqueda en línea permite encontrar ejemplos. Otra opción es consultar su documentación o simplemente configurar un servidor local y descubrir los errores recorriendo las páginas que utiliza el servidor web.

Para provocar mensajes de error, un evaluador debe:

- Buscar archivos y carpetas aleatorios que no existan (404).
- Intentar solicitar carpetas que existan y observar el comportamiento del servidor (403, página en blanco o listado de directorios).
- Intentar enviar una solicitud que infrinja el [RFC HTTP](https://tools.ietf.org/html/rfc7231). Por ejemplo, enviar una ruta muy larga, alterar el formato de los encabezados o cambiar la versión HTTP.
    - Aunque los errores se manejen a nivel de aplicación, infringir el RFC HTTP puede hacer que el servidor web integrado se revele, ya que debe manejar la solicitud y los desarrolladores olvidan sobrescribir estos errores.

### Aplicaciones

Las aplicaciones son las más propensas a revelar una amplia variedad de mensajes de error, entre ellos trazas de pila, volcados de memoria, excepciones mal manejadas y errores genéricos. Esto se debe a que, la mayoría de las veces, las aplicaciones se desarrollan a medida y los desarrolladores deben observar y manejar todos los posibles casos de error (o contar con un mecanismo global para capturarlos); además, estos errores pueden surgir de las integraciones con otros servicios.

Para hacer que una aplicación genere estos errores, un evaluador debe:

1. Identificar los posibles puntos de entrada en los que la aplicación espera datos.
2. Analizar el tipo de entrada esperado (cadenas, enteros, JSON, XML, etc.).
3. Hacer fuzzing en cada punto de entrada según los pasos anteriores para contar con un escenario de prueba más enfocado.
   - Hacer fuzzing en cada entrada con todas las inyecciones posibles no es la mejor solución, a menos que se disponga de tiempo ilimitado para las pruebas y la aplicación pueda manejar tal cantidad de entradas.
   - Si el fuzzing no es una opción, seleccionar manualmente entradas viables que tengan mayores probabilidades de romper un analizador (*por ejemplo*, un corchete de cierre para un cuerpo JSON, un texto largo cuando solo se esperan un par de caracteres, inyección CLRF con parámetros que podrían analizar los servidores y los controles de validación de entrada, caracteres especiales que no sean aplicables a los nombres de archivo, etc.).
   - Se debe hacer fuzzing con datos sin sentido para cada tipo, ya que a veces los intérpretes fallan fuera del manejo de excepciones del desarrollador.
4. Comprender el servicio que responde con el mensaje de error e intentar elaborar una lista de fuzzing más precisa para obtener más información o detalles del error de ese servicio (podría ser una base de datos, un servicio independiente, etc.).

Los mensajes de error a veces son la principal debilidad al mapear sistemas, especialmente en una arquitectura de microservicios. Si los servicios no están configurados adecuadamente para manejar errores de forma genérica y uniforme, los mensajes de error permitirán al evaluador identificar qué servicio gestiona cada solicitud y realizar un ataque más enfocado contra cada servicio.

> El evaluador debe prestar mucha atención al tipo de respuesta. A veces los errores se devuelven como una respuesta satisfactoria con un cuerpo de error, se ocultan en una redirección 302 o simplemente se representan de una manera personalizada.

## Remediación

Para la remediación, consulte los [Controles proactivos OWASP Top 10 C3: Validar entradas y manejar excepciones](https://top10proactive.owasp.org/archive/2024/the-top-10/c3-validate-input-and-handle-exceptions/) y la [Hoja de referencia sobre el manejo de errores](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html).

## Entornos de práctica

- [Juice Shop - Manejo de errores](https://pwning.owasp-juice.shop/companion-guide/latest/part2/security-misconfiguration.html#provoke-an-error-that-is-neither-very-gracefully-nor-consistently-handled)

## Referencias

- [WSTG: Apéndice D - Fuzzing](../../6-Appendix/D-Fuzzing.md)
- [Controles proactivos OWASP Top 10 (2024) C3: Validar entradas y manejar excepciones](https://top10proactive.owasp.org/archive/2024/the-top-10/c3-validate-input-and-handle-exceptions/)
- [ASVS v4.1 v7.4: Manejo de errores](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x15-V7-Error-Logging.md#v74-error-handling)
- [CWE 728 - Manejo inadecuado de errores](https://cwe.mitre.org/data/definitions/728.html)
- [Serie de hojas de referencia: Manejo de errores](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html)