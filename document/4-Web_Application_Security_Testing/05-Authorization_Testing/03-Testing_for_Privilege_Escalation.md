# Pruebas de Escalada de Privilegios

|ID          |
|------------|
|WSTG-ATHZ-03|

## Resumen

Esta sección describe el problema de escalar privilegios de un nivel a otro. Durante esta fase, el probador debe verificar que no sea posible para un usuario modificar sus privilegios o roles dentro de la aplicación de manera que permita ataques de escalada de privilegios.

La escalada de privilegios ocurre cuando un usuario obtiene acceso a más recursos o funcionalidad de los que normalmente se le permiten, y tal elevación o cambios deberían haber sido prevenidos por la aplicación. Esto generalmente se debe a un defecto en la aplicación. El resultado es que la aplicación realiza acciones con más privilegios de los previstos por el desarrollador o administrador del sistema.

El grado de escalada depende de qué privilegios está autorizado a poseer el atacante, y qué privilegios se pueden obtener en un exploit exitoso. Por ejemplo, un error de programación que permite a un usuario ganar privilegios extra después de una autenticación exitosa limita el grado de escalada, porque el usuario ya está autorizado a tener algunos privilegios. Del mismo modo, un atacante remoto que obtiene privilegios de superusuario sin ninguna autenticación presenta un mayor grado de escalada.

Generalmente, la gente se refiere a *escalada vertical* cuando es posible acceder a recursos concedidos a cuentas más privilegiadas (por ejemplo, adquirir privilegios administrativos para la aplicación), y a *escalada horizontal* cuando es posible acceder a recursos concedidos a una cuenta configurada de manera similar (por ejemplo, en una aplicación de banca en línea, acceder a información relacionada con un usuario diferente).

## Objetivos de las Pruebas

- Identificar puntos de inyección relacionados con la manipulación de privilegios.
- Probar o intentar eludir medidas de seguridad.

## Cómo Probar

### Pruebas de Manipulación de Rol/Privilegio

En cada porción de la aplicación donde un usuario puede crear información en la base de datos (por ejemplo, realizar un pago, agregar un contacto o enviar un mensaje), puede recibir información (estado de cuenta, detalles de pedido, etc.), o eliminar información (eliminar usuarios, mensajes, etc.), es necesario registrar esa funcionalidad. El probador debe intentar acceder a tales funciones como otro usuario para verificar si es posible acceder a una función que no debería estar permitida por el rol/privilegio del usuario (pero podría estar permitida como otro usuario).

#### Manipulación de Grupo de Usuario

Por ejemplo:
La siguiente solicitud HTTP POST permite al usuario que pertenece a `grp001` acceder al pedido #0001:

```http
POST /user/viewOrder.jsp HTTP/1.1
Host: www.example.com
...

groupID=grp001&orderID=0001
```

Verificar si un usuario que no pertenece a `grp001` puede modificar el valor de los parámetros `groupID` y `orderID` para ganar acceso a esos datos privilegiados.

#### Manipulación de Perfil de Usuario

Por ejemplo:
La siguiente respuesta del servidor muestra un campo oculto en el HTML devuelto al usuario después de una autenticación exitosa.

```html
HTTP/1.1 200 OK
Server: Netscape-Enterprise/6.0
Date: Wed, 1 Apr 2006 13:51:20 GMT
Set-Cookie: USER=aW78ryrGrTWs4MnOd32Fs51yDqp; path=/; domain=www.example.com
Set-Cookie: SESSION=k+KmKeHXTgDi1J5fT7Zz; path=/; domain= www.example.com
Cache-Control: no-cache
Pragma: No-cache
Content-length: 247
Content-Type: text/html
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Connection: close

<form  name="autoriz" method="POST" action = "visual.jsp">
<input type="hidden" name="profile" value="SysAdmin">\

<body onload="document.forms.autoriz.submit()">
</td>
</tr>
```

¿Qué pasa si el probador modifica el valor de la variable `profile` a `SysAdmin`? ¿Es posible convertirse en **administrador**?

#### Manipulación de Valor de Condición

Por ejemplo:
En un entorno donde el servidor envía un mensaje de error contenido como un valor en un parámetro específico en un conjunto de códigos de respuesta, como el siguiente:

```text
@0`1`3`3``0`UC`1`Status`OK`SEC`5`1`0`ResultSet`0`PVValid`-1`0`0` Notifications`0`0`3`Command  Manager`0`0`0` StateToolsBar`0`0`0`
StateExecToolBar`0`0`0`FlagsToolBar`0
```

El servidor da una confianza implícita al usuario. Cree que el usuario responderá con el mensaje anterior cerrando la sesión.

En esta condición, verificar que no sea posible escalar privilegios modificando los valores de los parámetros. En este ejemplo particular, modificando el valor `PVValid` de `-1` a `0` (sin condiciones de error), puede ser posible autenticarse como administrador en el servidor.

#### Manipulación de Dirección IP

Algunos sitios limitan el acceso o cuentan el número de intentos de inicio de sesión fallidos basados en la dirección IP.

Por ejemplo:

```text
X-Forwarded-For: 8.1.1.1
```

En este caso, si el sitio usa el valor de `X-forwarded-For` como dirección IP del cliente, el probador puede cambiar el valor IP del encabezado HTTP `X-forwarded-For` para eludir la identificación de fuente IP.

### Pruebas de Elusión Vertical del Esquema de Autorización

Una elusión de autorización vertical es específica al caso en que un atacante obtiene un rol superior al suyo. Las pruebas para esta elusión se centran en verificar cómo se ha implementado el esquema de autorización vertical para cada rol. Para cada función, página, rol específico o solicitud que la aplicación ejecuta, es necesario verificar si es posible:

- Acceder a recursos que deberían ser accesibles solo a un usuario de rol superior.
- Operar funciones en recursos que deberían ser operativas solo por un usuario que tenga una identidad de rol superior o específico.

Para cada rol:

1. Registrar un usuario.
2. Establecer y mantener dos sesiones diferentes basadas en los dos roles diferentes.
3. Para cada solicitud, cambiar el identificador de sesión del original a otro identificador de sesión de rol y evaluar las respuestas para cada uno.
4. Una aplicación se considerará vulnerable si la sesión de privilegios más débiles contiene los mismos datos, o indica operaciones exitosas en funciones de privilegios superiores.

#### Escenario de Roles en Sitio de Banca

La siguiente tabla ilustra los roles del sistema en un sitio de banca. Cada rol se vincula con permisos específicos para la funcionalidad del menú de eventos:

|      ROL      |     PERMISO       | PERMISO ADICIONAL |
|---------------|-------------------|-------------------|
| Administrador | Control Total     | Eliminar          |
| Gerente       | Modificar, Agregar, Leer | Agregar       |
| Personal      | Leer, Modificar   | Modificar         |
| Cliente       | Solo Lectura      |                   |

La aplicación se considerará vulnerable si:

1. El cliente pudiera operar funciones de administrador, gerente o personal;
2. El usuario de personal pudiera operar funciones de gerente o administrador;
3. El gerente pudiera operar funciones de administrador.

Suponga que la función `deleteEvent` es parte del menú de cuenta de administrador de la aplicación, y es posible acceder a ella solicitando la siguiente URL: `https://www.example.com/account/deleteEvent`. Entonces, la siguiente solicitud HTTP se genera al llamar a la función `deleteEvent`:

```http
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[otros encabezados HTTP]
Cookie: SessionID=ADMINISTRATOR_USER_SESSION

EventID=1000001
```

La respuesta válida:

```http
HTTP/1.1 200 OK
[otros encabezados HTTP]

{"message": "Event was deleted"}
```

El atacante puede intentar ejecutar la misma solicitud:

```http
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[otros encabezados HTTP]
Cookie: SessionID=CUSTOMER_USER_SESSION

EventID=1000002
```

Si la respuesta de la solicitud del atacante contiene los mismos datos `{"message": "Event was deleted"}`, la aplicación es vulnerable.

#### Acceso a Página de Administrador

Suponga que el menú de administrador es parte de la cuenta de administrador.

La aplicación se considerará vulnerable si cualquier rol diferente al administrador pudiera acceder al menú de administrador. A veces, los desarrolladores realizan validación de autorización solo a nivel de GUI, y dejan las funciones sin validación de autorización, lo que potencialmente resulta en una vulnerabilidad.

### Recorrido de URL

Intentar recorrer el sitio y verificar si algunas páginas pueden perder la verificación de autorización.

Por ejemplo:

```text
/../.././userInfo.html
```

### Caja Blanca

Si la verificación de autorización de URL se hace solo por coincidencia parcial de URL, entonces es probable que los probadores o hackers puedan eludir la autorización mediante técnicas de codificación de URL.

Por ejemplo:

```text
startswith(), endswith(), contains(), indexOf()
```

## Referencias

### Documentos Técnicos

- [Wikipedia - Privilege Escalation](https://en.wikipedia.org/wiki/Privilege_escalation)

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)