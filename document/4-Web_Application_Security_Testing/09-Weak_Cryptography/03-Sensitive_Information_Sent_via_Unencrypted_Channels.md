# Información sensible enviada a través de canales no cifrados

|ID          |
|------------|
|WSTG-CRYP-03|

## Resumen

Los datos sensibles deben protegerse cuando se transmiten a través de la red. Si los datos se transmiten mediante HTTPS o se cifran de otra manera, el mecanismo de protección no debe tener limitaciones ni vulnerabilidades, como se explica en el artículo más amplio [Seguridad débil de la capa de transporte](01-Weak_Transport_Layer_Security.md) y en otra documentación de OWASP:

- [A04:2025 Fallos criptográficos](https://owasp.org/Top10/2025/A04_2025-Cryptographic_Failures/).
- [OWASP ASVS - Verification V9](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x17-V9-Communications.md).
- [Hoja de referencia sobre protección de la capa de transporte](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html).

Como regla general, si los datos deben protegerse cuando se almacenan, también deben protegerse durante la transmisión. Algunos ejemplos de datos sensibles son:

- Información utilizada en la autenticación (p. ej., credenciales, PIN, identificadores de sesión, tokens, cookies, etc.)
- Información protegida por leyes, regulaciones o políticas organizacionales específicas (p. ej., tarjetas de crédito, registros médicos, datos personales, etc.)

Si la aplicación transmite información sensible a través de canales no cifrados, como HTTP, esto se considera un riesgo de seguridad. Los atacantes pueden tomar el control de cuentas [esnifando el tráfico de red](https://owasp.org/www-community/attacks/Manipulator-in-the-middle_attack). Algunos ejemplos son la autenticación básica, que envía credenciales de autenticación en texto sin formato a través de HTTP; las credenciales de autenticación basada en formularios enviadas mediante HTTP; o la transmisión en texto sin formato de cualquier otra información considerada sensible debido a regulaciones, leyes, políticas organizacionales o la lógica de negocio de la aplicación.

Algunos ejemplos de información de identificación personal (PII, por sus siglas en inglés) son:

- Números de seguro social
- Números de cuentas bancarias
- Información del pasaporte
- Información relacionada con la atención médica
- Información del seguro médico
- Información estudiantil
- Números de tarjetas de crédito y débito
- Información de licencias de conducir e identificaciones estatales

## Objetivos de prueba

- Identificar la información sensible transmitida a través de los distintos canales.
- Evaluar la privacidad y la seguridad de los canales utilizados.

## Cómo probar

La aplicación podría transmitir en texto sin formato distintos tipos de información que deben protegerse. Para comprobar si esta información se transmite mediante HTTP en lugar de HTTPS, capture el tráfico entre un cliente y el servidor de la aplicación web que requiere credenciales. Para cualquier mensaje que contenga datos sensibles, verifique que el intercambio se haya realizado mediante HTTPS. Consulte más información sobre la transmisión insegura de credenciales en [A04:2025 Fallos criptográficos](https://owasp.org/Top10/2025/A04_2025-Cryptographic_Failures/) o en la [Hoja de referencia sobre protección de la capa de transporte](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html).

### Autenticación básica mediante HTTP

Un ejemplo típico es el uso de la autenticación básica mediante HTTP. Al usar la autenticación básica, las credenciales del usuario se codifican en lugar de cifrarse y se envían como encabezados HTTP. En el ejemplo siguiente, el evaluador utiliza [curl](https://curl.se/) para comprobar este problema. Observe que la aplicación utiliza autenticación básica y HTTP en lugar de HTTPS.

```bash
$ curl -kis http://example.com/restricted/
HTTP/1.1 401 Authorization Required
Date: Fri, 01 Aug 2013 00:00:00 GMT
WWW-Authenticate: Basic realm="Restricted Area"
Accept-Ranges: bytes Vary:
Accept-Encoding Content-Length: 162
Content-Type: text/html

<html><head><title>401 Authorization Required</title></head>
<body bgcolor=white> <h1>401 Authorization Required</h1>  Invalid login credentials!  </body></html>
```

### Autenticación basada en formularios realizada mediante HTTP

Otro ejemplo típico son los formularios de autenticación que transmiten las credenciales de autenticación del usuario mediante HTTP. En el ejemplo siguiente se puede ver que se utiliza HTTP en el atributo `action` del formulario. También es posible detectar este problema examinando el tráfico HTTP con un proxy de interceptación.

```html
<form action="http://example.com/login">
    <label for="username">User:</label> <input type="text" id="username" name="username" value=""/><br />
    <label for="password">Password:</label> <input type="password" id="password" name="password" value=""/>
    <input type="submit" value="Login"/>
</form>
```

Si no es posible acceder directamente a un endpoint HTTP desde un navegador (por ejemplo, debido a HSTS o al comportamiento de la caché), solicite el mismo endpoint con [curl](https://curl.se/) e inspeccione directamente la respuesta. Un endpoint protegido correctamente debería devolver de inmediato una respuesta 3XX que redirija a una URL HTTPS.

```bash
$ curl -kis -o /dev/null -D - http://example.com/login
HTTP/1.1 301 Moved Permanently
Location: https://example.com/login
```

Si el servidor responde con 200, sirve contenido mediante HTTP o redirige a otro endpoint HTTP, es posible que la información sensible siga expuesta a través de un transporte no cifrado.

### Cookie que contiene el ID de sesión enviada mediante HTTP

La cookie con el ID de sesión debe transmitirse a través de canales protegidos. Si la cookie no tiene establecido el [atributo Secure](../06-Session_Management/02-Cookies_Attributes.md), la aplicación puede transmitirla sin cifrar. Observe que, en el ejemplo siguiente, la cookie se establece sin el atributo Secure y todo el proceso de inicio de sesión se realiza mediante HTTP, no HTTPS.

```http
https://secure.example.com/login

POST /login HTTP/1.1
Host: secure.example.com
[...]
Referer: https://secure.example.com/
Content-Type: application/x-www-form-urlencoded
Content-Length: 188

HTTP/1.1 302 Found
Server: Apache
Set-Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA; expires=Fri, 01-Jan-2014 00:00:00 GMT; path=/; domain=example.com; httponly
[...]
```

```http
http://example.com/private

GET /private HTTP/1.1
Host: example.com
[...]
Referer: https://secure.example.com/login
Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA;

HTTP/1.1 200 OK
[...]
```

### Restablecimiento de contraseña, cambio de contraseña u otra manipulación de la cuenta mediante HTTP

Si la aplicación web tiene funciones que permiten a un usuario cambiar una cuenta o llamar a otro servicio con credenciales, verifique que todas esas interacciones utilicen HTTPS. Entre las interacciones que se deben probar se incluyen los formularios que:

- Permiten a los usuarios gestionar una contraseña olvidada u otras credenciales.
- Permiten a los usuarios editar credenciales.
- Requieren que el usuario se autentique con otro proveedor (por ejemplo, para procesar pagos).

### Prueba de información sensible sobre contraseñas en el código fuente o los registros

Utilice una de las siguientes técnicas para buscar información sensible.

Compruebe si hay contraseñas o claves de cifrado codificadas directamente en el código fuente o en los archivos de configuración.

`grep -r -E "Pass | password | pwd |user | guest| admin | encry | key | decrypt | sharekey " ./PathToSearch/`

Compruebe si los registros o el código fuente pueden contener números de teléfono, direcciones de correo electrónico, identificadores u otros datos PII. Cambie la expresión regular según el formato de los datos PII.

`grep -r " {2\}[0-9]\{6\} "  ./PathToSearch/`

## Mitigación

Utilice HTTPS en todo el sitio web y redirija todas las solicitudes HTTP a HTTPS.

## Herramientas

- [curl](https://curl.se/)
- [grep](https://man7.org/linux/man-pages/man1/egrep.1.html)
- [Wireshark](https://www.wireshark.org/)
- [TCPDUMP](https://www.tcpdump.org/)

## Referencias

- [Transporte inseguro de OWASP](https://owasp.org/www-community/vulnerabilities/Insecure_Transport)
- [Hoja de referencia de OWASP sobre HTTP Strict Transport Security](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [Let's Encrypt](https://letsencrypt.org)