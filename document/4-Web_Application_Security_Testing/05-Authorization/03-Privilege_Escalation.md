# Escalada de privilegios

|ID          |
|------------|
|WSTG-ATHZ-03|

## Resumen

Esta sección describe el problema de escalar privilegios de una etapa a otra. Durante esta fase, el probador debe verificar que no sea posible que un usuario modifique sus privilegios o roles dentro de la aplicación de formas que puedan permitir ataques de escalada de privilegios.

La escalada de privilegios ocurre cuando un usuario obtiene acceso a más recursos o funcionalidades de los que normalmente tiene permitidos, y la aplicación debería haber impedido esa elevación o esos cambios. Esto suele deberse a un defecto en la aplicación. Como resultado, la aplicación realiza acciones con más privilegios de los previstos por el desarrollador o el administrador del sistema.

El grado de escalada depende de los privilegios que el atacante está autorizado a tener y de los privilegios que puede obtener mediante un exploit exitoso. Por ejemplo, un error de programación que permite a un usuario obtener privilegios adicionales después de autenticarse correctamente limita el grado de escalada, porque el usuario ya está autorizado a tener algunos privilegios. Del mismo modo, un atacante remoto que obtiene privilegios de superusuario sin autenticación presenta un mayor grado de escalada.

Por lo general, se habla de *escalada vertical* cuando es posible acceder a recursos concedidos a cuentas con más privilegios (por ejemplo, obtener privilegios administrativos en la aplicación), y de *escalada horizontal* cuando es posible acceder a recursos concedidos a una cuenta configurada de forma similar (por ejemplo, en una aplicación de banca en línea, acceder a información relacionada con otro usuario).

## Objetivos de las pruebas

- Identificar puntos de inyección relacionados con la manipulación de privilegios.
- Fuzzear o intentar de otro modo eludir las medidas de seguridad.

## Cómo probar

### Manipulación de roles y privilegios

En cada parte de la aplicación donde un usuario pueda crear información en la base de datos (por ejemplo, realizar un pago, agregar un contacto o enviar un mensaje), recibir información (estado de cuenta, detalles de pedidos, etc.) o eliminar información (eliminar usuarios, mensajes, etc.), es necesario registrar esa funcionalidad. El probador debe intentar acceder a esas funciones como otro usuario para verificar si es posible acceder a una función que no debería estar permitida para el rol o los privilegios del usuario (pero que podría estarlo para otro usuario).

#### Manipulación de grupos de usuarios

Por ejemplo:
La siguiente solicitud HTTP POST permite que el usuario que pertenece a `grp001` acceda al pedido #0001:

```http
POST /user/viewOrder.jsp HTTP/1.1
Host: www.example.com
...

groupID=grp001&orderID=0001
```

Verifique si un usuario que no pertenece a `grp001` puede modificar los valores de los parámetros `groupID` y `orderID` para obtener acceso a esos datos privilegiados.

#### Manipulación de perfiles de usuario

Por ejemplo:
La siguiente respuesta del servidor muestra un campo oculto en el HTML devuelto al usuario después de una autenticación correcta.

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
<input type="hidden" name="profile" value="SysGuest">\

<body onload="document.forms.autoriz.submit()">
</td>
</tr>
```

¿Qué ocurre si el probador modifica el valor de la variable `profile` a `SysAdmin`? ¿Es posible convertirse en **administrador**?

#### Manipulación de valores de condiciones

Por ejemplo:
En un entorno donde el servidor envía un mensaje de error como valor de un parámetro específico dentro de un conjunto de códigos de respuesta, como el siguiente:

```text
@0`1`3`3``0`UC`1`Status`OK`SEC`5`1`0`ResultSet`0`PVValid`-1`0`0` Notifications`0`0`3`Command  Manager`0`0`0` StateToolsBar`0`0`0`
StateExecToolBar`0`0`0`FlagsToolBar`0
```

El servidor confía implícitamente en el usuario. Cree que el usuario responderá con el mensaje anterior para cerrar la sesión.

En este caso, verifique que no sea posible escalar privilegios modificando los valores de los parámetros. En este ejemplo concreto, al modificar el valor de `PVValid` de `-1` a `0` (sin condiciones de error), podría ser posible autenticarse en el servidor como administrador.

#### Manipulación de direcciones IP

Algunos sitios limitan el acceso o cuentan el número de intentos fallidos de inicio de sesión en función de la dirección IP.

Por ejemplo:

```text
X-Forwarded-For: 8.1.1.1
```

En este caso, si el sitio usa el valor de `X-forwarded-For` como dirección IP del cliente, el probador puede cambiar el valor de IP del encabezado HTTP `X-forwarded-For` para eludir la identificación de la dirección IP de origen.

### Elusión vertical

Una elusión vertical de la autorización se da cuando un atacante obtiene un rol superior al suyo. Las pruebas para detectar esta elusión se centran en verificar cómo se ha implementado el esquema de autorización vertical para cada rol. Para cada función, página, rol específico o solicitud que ejecuta la aplicación, es necesario verificar si es posible:

- Acceder a recursos que deberían estar disponibles solo para un usuario con un rol superior.
- Ejecutar funciones en recursos que deberían estar disponibles solo para un usuario que tenga una identidad de rol superior o específica.

Para cada rol:

1. Registrar un usuario.
2. Establecer y mantener dos sesiones diferentes basadas en los dos roles distintos.
3. Para cada solicitud, cambiar el identificador de sesión original por el identificador de sesión de otro rol y evaluar las respuestas de cada una.
4. Se considerará que una aplicación es vulnerable si la sesión con menos privilegios contiene los mismos datos o indica que se realizaron operaciones correctamente en funciones de mayor privilegio.

#### Escenario de roles en un sitio bancario

La siguiente tabla ilustra los roles del sistema en un sitio bancario. Cada rol se asocia con permisos específicos para la funcionalidad del menú de eventos:

|      ROL      |     PERMISO    | PERMISO ADICIONAL |
|---------------|----------------|-------------------|
| Administrador | Control total  | Eliminar          |
| Gerente       | Modificar, Agregar, Leer | Agregar   |
| Personal      | Leer, Modificar | Modificar        |
| Cliente       | Solo lectura   |                   |

La aplicación se considerará vulnerable si:

1. El cliente puede ejecutar funciones de administrador, gerente o personal;
2. El usuario de personal puede ejecutar funciones de gerente o administrador;
3. El gerente puede ejecutar funciones de administrador.

Supongamos que la función `deleteEvent` forma parte del menú de la cuenta de administrador de la aplicación y que es posible acceder a ella solicitando la siguiente URL: `https://www.example.com/account/deleteEvent`. Entonces, al llamar a la función `deleteEvent`, se genera la siguiente solicitud HTTP:

```http
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=ADMINISTRATOR_USER_SESSION

EventID=1000001
```

La respuesta válida:

```http
HTTP/1.1 200 OK
[other HTTP headers]

{"message": "Event was deleted"}
```

El atacante puede intentar ejecutar la misma solicitud:

```http
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[other HTTP headers]
Cookie: SessionID=CUSTOMER_USER_SESSION

EventID=1000002
```

Si la respuesta a la solicitud del atacante contiene los mismos datos `{"message": "Event was deleted"}`, la aplicación es vulnerable.

#### Acceso a páginas de administrador

Supongamos que el menú de administrador forma parte de la cuenta de administrador.

La aplicación se considerará vulnerable si cualquier rol que no sea el de administrador puede acceder al menú de administrador. A veces, los desarrolladores realizan la validación de autorización solo en la interfaz gráfica de usuario y dejan las funciones sin validación de autorización, lo que puede dar lugar a una vulnerabilidad.

### Recorrido de URL

Intente recorrer el sitio y compruebe si hay páginas a las que les falte la verificación de autorización.

Por ejemplo:

```text
/../.././userInfo.html
```

### Caja blanca

Si la verificación de autorización de URL se realiza solo mediante una coincidencia parcial de URL, es probable que los probadores o los atacantes puedan eludir la autorización mediante técnicas de codificación de URL.

Por ejemplo:

```text
startswith(), endswith(), contains(), indexOf()
```

## Referencias

### Documentos técnicos

- [Wikipedia - Escalada de privilegios](https://en.wikipedia.org/wiki/Privilege_escalation)

## Herramientas

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- RAGE (Relational Attack Graph Exchange): formato NDJSON portátil y taxonomía para modelar rutas de ataque a identidades y accesos, así como cadenas de escalada de privilegios; resulta especialmente útil cuando la aplicación se ejecuta en entornos de IAM en la nube o con varias cuentas. Consulte [RAGE](https://rage.fail/).