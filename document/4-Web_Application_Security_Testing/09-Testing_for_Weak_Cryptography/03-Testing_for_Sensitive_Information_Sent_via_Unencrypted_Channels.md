# Pruebas de Información Sensible Enviada vía Canales No Cifrados

|ID          |
|------------|
|WSTG-CRYP-03|

## Resumen

Los datos sensibles deben protegerse cuando se transmiten a través de la red. Si los datos se transmiten sobre HTTPS o se cifran de otra manera, el mecanismo de protección no debe tener limitaciones o vulnerabilidades, como se explica en el artículo más amplio [Pruebas de Seguridad de Capa de Transporte Débil](01-Testing_for_Weak_Transport_Layer_Security.md) y en otra documentación de OWASP:

- [OWASP Top 10 2017 A3-Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure).
- [OWASP ASVS - Verification V9](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x17-V9-Communications.md).
- [Hoja de Referencia de Protección de Capa de Transporte](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html).

Como regla general, si los datos deben protegerse cuando se almacenan, estos datos también deben protegerse durante la transmisión. Algunos ejemplos de datos sensibles son:

- Información usada en autenticación (por ejemplo credenciales, PINs, identificadores de sesión, tokens, cookies, etc.)
- Información protegida por leyes, regulaciones o política organizacional específica (por ejemplo tarjetas de crédito, registros médicos, datos personales, etc.)

Si la aplicación transmite información sensible vía canales no cifrados tales como HTTP se considera un riesgo de seguridad. Los atacantes pueden tomar el control de cuentas [espiando el tráfico de red](https://owasp.org/www-community/attacks/Manipulator-in-the-middle_attack). Algunos ejemplos son autenticación Basic que envía credenciales de autenticación en texto plano sobre HTTP, credenciales de autenticación basada en formulario enviadas vía HTTP, o transmisión en texto plano de cualquier otra información considerada sensible debido a regulaciones, leyes, política organizacional o lógica de negocio de la aplicación.

Ejemplos de Información de Identificación Personal (PII, por sus siglas en inglés) son:

- Números de seguro social
- Números de cuenta bancaria
- Información de pasaporte
- Información relacionada con salud
- Información de seguro médico
- Información de estudiante
- Números de tarjetas de crédito y débito
- Información de licencia de conducir e identificación estatal

## Objetivos de Prueba

- Identificar información sensible transmitida a través de los varios canales.
- Evaluar la privacidad y seguridad de los canales usados.

## Cómo Probar

Varios tipos de información que deben protegerse podrían ser transmitidos por la aplicación en texto claro. Para verificar si esta información se transmite sobre HTTP en lugar de HTTPS, capturar tráfico entre un cliente y un servidor de aplicación web que necesita credenciales. Para cualquier mensaje que contenga datos sensibles, verificar que el intercambio ocurra usando HTTPS. Ver más información sobre transmisión insegura de credenciales en [OWASP Top 10 2017 A3-Sensitive Data Exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure) o [Hoja de Referencia de Protección de Capa de Transporte](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html).

### Ejemplo 1: Autenticación Basic sobre HTTP

Un ejemplo típico es el uso de Autenticación Basic sobre HTTP. Al usar Autenticación Basic, las credenciales del usuario se codifican en lugar de cifrarse, y se envían como encabezados HTTP. En el ejemplo a continuación el tester usa [curl](https://curl.haxx.se/) para probar este problema. Notar cómo la aplicación usa autenticación Basic, y HTTP en lugar de HTTPS.

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

### Ejemplo 2: Autenticación Basada en Formulario Realizada sobre HTTP

Otro ejemplo típico es formularios de autenticación que transmiten credenciales de autenticación de usuario sobre HTTP. En el ejemplo a continuación se puede ver HTTP siendo usado en el atributo `action` del formulario. También es posible ver este problema examinando el tráfico HTTP con un proxy de intercepción.

```html
<form action="http://example.com/login">
    <label for="username">User:</label> <input type="text" id="username" name="username" value=""/><br />
    <label for="password">Password:</label> <input type="password" id="password" name="password" value=""/>
    <input type="submit" value="Login"/>
</form>
```

### Ejemplo 3: Cookie Conteniendo ID de Sesión Enviada sobre HTTP

La Cookie de ID de Sesión debe transmitirse sobre canales protegidos. Si la cookie no tiene el [flag Secure](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md) establecido, se permite a la aplicación transmitirla sin cifrar. Notar a continuación que el establecimiento de la cookie se hace sin el flag Secure, y el proceso de login completo se realiza en HTTP y no HTTPS.

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

### Ejemplo 4: Reseteo de Contraseña, Cambio de Contraseña u Otra Manipulación de Cuenta sobre HTTP

Si la aplicación web tiene características que permiten al usuario cambiar una cuenta o llamar a un servicio diferente con credenciales, verificar que todas esas interacciones usen HTTPS. Las interacciones a probar incluyen formularios que:

- Permiten a los usuarios manejar una contraseña olvidada u otras credenciales.
- Permiten a los usuarios editar credenciales.
- Requieren que el usuario se autentique con otro proveedor (por ejemplo, procesamiento de pagos).

### Ejemplo 5: Probar Información Sensible de Contraseña en Código Fuente o Logs

Usar una de las siguientes técnicas para buscar información sensible.

Verificar si contraseña o clave de cifrado está hardcoded en el código fuente o archivos de configuración.

`grep -r –E "Pass | password | pwd |user | guest| admin | encry | key | decrypt | sharekey " ./PathToSearch/`

Verificar si logs o código fuente podrían contener número de teléfono, dirección de correo electrónico, ID u otra PII. Cambiar la expresión regular basada en el formato de la PII.

`grep -r " {2\}[0-9]\{6\} "  ./PathToSearch/`

## Remediación

Usar HTTPS para todo el sitio web y redirigir cualquier solicitud HTTP a HTTPS.

## Herramientas

- [curl](https://curl.haxx.se/)
- [grep](https://man7.org/linux/man-pages/man1/egrep.1.html)
- [Wireshark](https://www.wireshark.org/)
- [TCPDUMP](https://www.tcpdump.org/)

## Referencias

- [OWASP Insecure Transport](https://owasp.org/www-community/vulnerabilities/Insecure_Transport)
- [Hoja de Referencia de HTTP Strict Transport Security de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [Let's Encrypt](https://letsencrypt.org)
