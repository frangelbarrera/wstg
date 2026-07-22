# Prueba de inyección IMAP SMTP

|ID          |
|------------|
|WSTG-INPV-10|

## Resumen

Esta amenaza afecta a todas las aplicaciones que se comunican con servidores de correo (IMAP/SMTP), generalmente aplicaciones de webmail. El objetivo de esta prueba es verificar la capacidad de inyectar comandos IMAP/SMTP arbitrarios en los servidores de correo, debido a que los datos de entrada no se sanitizan correctamente.

La técnica de inyección IMAP/SMTP es más efectiva si el servidor de correo no es directamente accesible desde Internet. Cuando es posible la comunicación completa con el servidor de correo backend, se recomienda realizar pruebas directas.

Una inyección IMAP/SMTP permite acceder a un servidor de correo que de otro modo no sería directamente accesible desde Internet. En algunos casos, estos sistemas internos no tienen el mismo nivel de seguridad e infraestructura que se aplica a los servidores web front-end. Por lo tanto, los resultados del servidor de correo pueden ser más vulnerables a ataques por parte de usuarios finales (ver el esquema presentado en la Figura 1).

![IMAP SMTP Injection](images/Imap-smtp-injection.png)\
*Figura 4.7.10-1: Comunicación con los servidores de correo utilizando la técnica de inyección IMAP/SMTP*

La Figura 1 muestra el flujo de tráfico generalmente visto al usar tecnologías de webmail. El paso 1 y 2 es el usuario interactuando con el cliente de webmail, mientras que el paso 2 es el probador omitiendo el cliente de webmail e interactuando con los servidores de correo backend directamente.

Esta técnica permite una amplia variedad de acciones y ataques. Las posibilidades dependen del tipo y alcance de la inyección y la tecnología del servidor de correo que se está probando.

Algunos ejemplos de ataques utilizando la técnica de inyección IMAP/SMTP son:

- Explotación de vulnerabilidades en el protocolo IMAP/SMTP
- Evasión de restricciones de la aplicación
- Evasión de procesos anti-automatización
- Fugas de información
- Relay/SPAM

## Objetivos de la prueba

- Identificar puntos de inyección IMAP/SMTP.
- Entender el flujo de datos y la estructura de despliegue del sistema.
- Evaluar los impactos de la inyección.

## Cómo probar

### Identificación de parámetros vulnerables

Para detectar parámetros vulnerables, el probador debe analizar la capacidad de la aplicación para manejar la entrada. Las pruebas de validación de entrada requieren que el probador envíe solicitudes falsas o maliciosas al servidor y analice la respuesta. En una aplicación segura, la respuesta debería ser un error con alguna acción correspondiente que le diga al cliente que algo ha salido mal. En una aplicación vulnerable, la solicitud maliciosa puede ser procesada por la aplicación backend que responderá con un mensaje de respuesta `HTTP 200 OK`.

Es importante notar que las solicitudes enviadas deben coincidir con la tecnología que se está probando. Enviar cadenas de inyección SQL para Microsoft SQL server cuando se está usando un servidor MySQL resultará en respuestas positivas falsas. En este caso, enviar comandos IMAP maliciosos es el modus operandi ya que IMAP es el protocolo subyacente que se está probando.

Los parámetros especiales de IMAP que se deben usar son:

| En el servidor IMAP     | En el servidor SMTP |
|------------------------|--------------------|
| Autenticación         | Email del emisor     |
| operaciones con buzones (listar, leer, crear, eliminar, renombrar) | Email de destino |
| operaciones con mensajes (leer, copiar, mover, eliminar) | Asunto   |
| Desconexión          | Cuerpo del mensaje       |
|                        | Archivos adjuntos     |

En este ejemplo, el parámetro "mailbox" se está probando manipulando todas las solicitudes con el parámetro en:

`https://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=46106&startMessage=1`

Los siguientes ejemplos se pueden usar.

- Asignar un valor nulo al parámetro:

`https://<webmail>/src/read_body.php?mailbox=&passed_id=46106&startMessage=1`

- Sustituir el valor con un valor aleatorio:

`https://<webmail>/src/read_body.php?mailbox=NOTEXIST&passed_id=46106&startMessage=1`

- Agregar otros valores al parámetro:

`https://<webmail>/src/read_body.php?mailbox=INBOX PARAMETER2&passed_id=46106&startMessage=1`

- Agregar caracteres especiales no estándar (es decir: `\`, `'`, `"`, `@`, `#`, `!`, `|`):

`https://<webmail>/src/read_body.php?mailbox=INBOX"&passed_id=46106&startMessage=1`

- Eliminar el parámetro:

`https://<webmail>/src/read_body.php?passed_id=46106&startMessage=1`

El resultado final de las pruebas anteriores da al probador tres situaciones posibles:
S1 - La aplicación devuelve un código/mensaje de error
S2 - La aplicación no devuelve un código/mensaje de error, pero no realiza la operación solicitada
S3 - La aplicación no devuelve un código/mensaje de error y realiza la operación solicitada normalmente

Las situaciones S1 y S2 representan una inyección IMAP/SMTP exitosa.

El objetivo del atacante es recibir la respuesta S1, ya que es un indicador de que la aplicación es vulnerable a la inyección y a una manipulación adicional.

Supongamos que un usuario recupera los encabezados del email usando la siguiente solicitud HTTP:

`https://<webmail>/src/view_header.php?mailbox=INBOX&passed_id=46105&passed_ent_id=0`

Un atacante podría modificar el valor del parámetro INBOX inyectando el carácter `"` (%22 usando codificación URL):

`https://<webmail>/src/view_header.php?mailbox=INBOX%22&passed_id=46105&passed_ent_id=0`

En este caso, la respuesta de la aplicación puede ser:

```txt
ERROR: Bad or malformed request.
Query: SELECT "INBOX""
Server responded: Unexpected extra arguments to Select
```

La situación S2 es más difícil de probar exitosamente. El probador necesita usar inyección de comandos ciega para determinar si el servidor es vulnerable.

Por otro lado, la última situación (S3) no es relevante en este párrafo.

> Lista de parámetros vulnerables
>
> - Funcionalidad afectada
> - Tipo de inyección posible (IMAP/SMTP)

### Entendiendo el flujo de datos y la estructura de despliegue del cliente

Después de identificar todos los parámetros vulnerables (por ejemplo, `passed_id`), el probador necesita determinar qué nivel de inyección es posible y luego diseñar un plan de pruebas para explotar aún más la aplicación.

En este caso de prueba, hemos detectado que el parámetro `passed_id` de la aplicación es vulnerable y se usa en la siguiente solicitud:

`https://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=46225&startMessage=1`

Usando el siguiente caso de prueba (proporcionando un valor alfabético cuando se requiere un valor numérico):

`https://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=test&startMessage=1`

generará el siguiente mensaje de error:

```txt
ERROR : Bad or malformed request.
Query: FETCH test:test BODY[HEADER]
Server responded: Error in IMAP command received by server.
```

En este ejemplo, el mensaje de error devolvió el nombre del comando ejecutado y los parámetros correspondientes.

En otras situaciones, el mensaje de error (`no controlado` por la aplicación) contiene el nombre del comando ejecutado, pero leer el [RFC](#references) adecuado permite al probador entender qué otros comandos posibles se pueden ejecutar.

Si la aplicación no devuelve mensajes de error descriptivos, el probador necesita analizar la funcionalidad afectada para deducir todos los comandos posibles (y parámetros) asociados con la funcionalidad mencionada anteriormente. Por ejemplo, si se ha detectado un parámetro vulnerable en la funcionalidad de crear buzón, es lógico asumir que el comando IMAP afectado es `CREATE`. Según el RFC, el comando `CREATE` acepta un parámetro que especifica el nombre del buzón a crear.

> Lista de comandos IMAP/SMTP afectados
>
> - Tipo, valor y número de parámetros esperados por los comandos IMAP/SMTP afectados

### Inyección de comandos IMAP/SMTP

Una vez que el probador ha identificado parámetros vulnerables y ha analizado el contexto en el que se ejecutan, la siguiente etapa es explotar la funcionalidad.

Esta etapa tiene dos resultados posibles:

1. La inyección es posible en un estado no autenticado: la funcionalidad afectada no requiere que el usuario esté autenticado. Los comandos (IMAP) inyectados disponibles están limitados a: `CAPABILITY`, `NOOP`, `AUTHENTICATE`, `LOGIN` y `LOGOUT`.
2. La inyección solo es posible en un estado autenticado: la explotación exitosa requiere que el usuario esté completamente autenticado antes de que las pruebas puedan continuar.

En cualquier caso, la estructura típica de una inyección IMAP/SMTP es la siguiente:

- Encabezado: finalización del comando esperado;
- Cuerpo: inyección del nuevo comando;
- Pie: inicio del comando esperado.

Es importante recordar que, para ejecutar un comando IMAP/SMTP, el comando anterior debe terminarse con la secuencia CRLF (`%0d%0a`).

Supongamos que en la etapa [Identificación de parámetros vulnerables](#identifying-vulnerable-parameters), el atacante detecta que el parámetro `message_id` en la siguiente solicitud es vulnerable:

`https://<webmail>/read_email.php?message_id=4791`

Supongamos también que el resultado del análisis realizado en la etapa 2 ("Entendiendo el flujo de datos y la estructura de despliegue del cliente") ha identificado el comando y argumentos asociados con este parámetro como:

`FETCH 4791 BODY[HEADER]`

En este escenario, la estructura de inyección IMAP sería:

`https://<webmail>/read_email.php?message_id=4791 BODY[HEADER]%0d%0aV100 CAPABILITY%0d%0aV101 FETCH 4791`

Que generaría los siguientes comandos:

```sql
???? FETCH 4791 BODY[HEADER]
V100 CAPABILITY
V101 FETCH 4791 BODY[HEADER]
```

donde:

```sql
Header = 4791 BODY[HEADER]
Body   = %0d%0aV100 CAPABILITY%0d%0a
Footer = V101 FETCH 4791
```

> Lista de comandos IMAP/SMTP afectados
>
> - Inyección arbitraria de comandos IMAP/SMTP

## Referencias

### Documentos técnicos

- [RFC 0821 "Simple Mail Transfer Protocol"](https://tools.ietf.org/html/rfc821)
- [RFC 3501 "Internet Message Access Protocol - Version 4rev1"](https://tools.ietf.org/html/rfc3501)