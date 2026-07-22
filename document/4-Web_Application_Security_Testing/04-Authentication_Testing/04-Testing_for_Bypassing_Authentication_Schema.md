# Pruebas de omisión del esquema de autenticación

|ID          |
|------------|
|WSTG-ATHN-04|

## Resumen

En seguridad informática, la autenticación es el proceso de intentar verificar la identidad digital del remitente de una comunicación. Un ejemplo común de dicho proceso es el proceso de inicio de sesión. Probar el esquema de autenticación significa comprender cómo funciona el proceso de autenticación y utilizar esa información para eludir el mecanismo de autenticación.

Aunque la mayoría de las aplicaciones requieren autenticación para acceder a información privada o ejecutar tareas, no todos los métodos de autenticación son capaces de proporcionar seguridad adecuada. La negligencia, la ignorancia o la simple subestimación de las amenazas de seguridad a menudo resultan en esquemas de autenticación que pueden omitirse simplemente saltando la página de inicio de sesión y llamando directamente a una página interna que se supone que solo debe accederse después de que se haya realizado la autenticación.

Además, a menudo es posible omitir las medidas de autenticación manipulando solicitudes y engañando a la aplicación para que piense que el usuario ya está autenticado. Esto puede lograrse modificando el parámetro de URL dado, manipulando el formulario o falsificando sesiones.

Los problemas relacionados con el esquema de autenticación pueden encontrarse en diferentes etapas del ciclo de vida del desarrollo de software (SDLC), como las fases de diseño, desarrollo e implementación:

- En la fase de diseño, los errores pueden incluir una definición incorrecta de las secciones de la aplicación que se protegerán, la elección de no aplicar protocolos de cifrado fuertes para asegurar la transmisión de credenciales, y muchos más.
- En la fase de desarrollo, los errores pueden incluir la implementación incorrecta de la funcionalidad de validación de entrada o no seguir las mejores prácticas de seguridad para el lenguaje específico.
- En la fase de implementación de la aplicación, puede haber problemas durante la configuración de la aplicación (actividades de instalación y configuración) debido a la falta de habilidades técnicas requeridas o a la falta de buena documentación.

## Objetivos de la prueba

- Asegurar que la autenticación se aplique en todos los servicios que la requieren.

## Cómo probar

Existen varios métodos para omitir el esquema de autenticación utilizado por una aplicación web:

- Modificación de parámetros
- Predicción de ID de sesión
- Inyección SQL

### Modificación de parámetros

Otro problema relacionado con el diseño de autenticación es cuando la aplicación verifica un inicio de sesión exitoso en base a parámetros de valor fijo. Un usuario podría modificar estos parámetros para acceder a las áreas protegidas sin proporcionar credenciales válidas. En el ejemplo a continuación, el parámetro "authenticated" se cambia a un valor de "yes", lo que permite al usuario acceder. En este ejemplo, el parámetro está en la URL, pero también se podría usar un proxy para modificar el parámetro, especialmente cuando los parámetros se envían como elementos de formulario en una solicitud POST o cuando los parámetros se almacenan en una cookie.

```html
https://www.site.com/page.asp?authenticated=no

raven@blackbox /home $nc www.site.com 80
GET /page.asp?authenticated=yes HTTP/1.0

HTTP/1.1 200 OK
Date: Sat, 11 Nov 2006 10:22:44 GMT
Server: Apache
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<HTML><HEAD>
</HEAD><BODY>
<H1>You Are Authenticated</H1>
</BODY></HTML>
```

![Solicitud con parámetro modificado](images/Basm-parammod.jpg)\
*Figura 4.4.4-1: Solicitud con parámetro modificado*

### Predicción de ID de sesión

Muchas aplicaciones web gestionan la autenticación utilizando identificadores de sesión (ID de sesión). Por lo tanto, si la generación de ID de sesión es predecible, un usuario malicioso podría ser capaz de encontrar un ID de sesión válido y acceder sin autorización a la aplicación, suplantando a un usuario previamente autenticado.

En la siguiente figura, los valores dentro de las cookies aumentan linealmente, por lo que podría ser fácil para un atacante adivinar un ID de sesión válido.

![Valores de cookies a lo largo del tiempo](images/Basm-sessid.jpg)\
*Figura 4.4.4-2: Valores de cookies a lo largo del tiempo*

En la siguiente figura, los valores dentro de las cookies cambian solo parcialmente, por lo que es posible restringir un ataque de fuerza bruta a los campos definidos que se muestran a continuación.

![Valores de cookies parcialmente cambiados](images/Basm-sessid2.jpg)\
*Figura 4.4.4-3: Valores de cookies parcialmente cambiados*

### Inyección SQL (Autenticación de formulario HTML)

La inyección SQL es una técnica de ataque ampliamente conocida. Esta sección no va a describir esta técnica en detalle ya que hay varias secciones en esta guía que explican técnicas de inyección más allá del alcance de esta sección.

![Inyección SQL](images/Basm-sqlinj.jpg)\
*Figura 4.4.4-4: Inyección SQL*

La siguiente figura muestra que con un simple ataque de inyección SQL, a veces es posible omitir el formulario de autenticación.

![Ataque simple de inyección SQL](images/Basm-sqlinj2.gif)\
*Figura 4.4.4-5: Ataque simple de inyección SQL*

### Comparación laxa en PHP

Si un atacante ha podido recuperar el código fuente de la aplicación explotando una vulnerabilidad previamente descubierta (por ejemplo, traversal de directorios), o desde un repositorio web (Aplicaciones de código abierto), podría ser posible realizar ataques refinados contra la implementación del proceso de autenticación.

En el siguiente ejemplo (PHPBB 2.0.12 - Vulnerabilidad de omisión de autenticación), en la línea 2 la función `unserialize()` analiza una cookie proporcionada por el usuario y establece valores dentro del array `$sessiondata`. En la línea 7, el hash MD5 de la contraseña del usuario almacenado en la base de datos backend (`$auto_login_key`) se compara con el proporcionado (`$sessiondata['autologinid']`) por el usuario.

```php
1. if (isset($HTTP_COOKIE_VARS[$cookiename . '_sid'])) {
2.     $sessiondata = isset($HTTP_COOKIE_VARS[$cookiename . '_data']) ? unserialize(stripslashes($HTTP_COOKIE_VARS[$cookiename . '_data'])) : array();
3.     $sessionmethod = SESSION_METHOD_COOKIE;
4. }
5. $auto_login_key = $userdata['user_password'];
6. // We have to login automagically
7. if( $sessiondata['autologinid'] == $auto_login_key )
8. {
9.     // autologinid matches password
10.     $login = 1;
11.     $enable_autologin = 1;
12. }

```

En PHP, una comparación entre un valor de cadena y un valor booleano `true` siempre es `true` (porque la cadena contiene un valor), por lo que al proporcionar la siguiente cadena a la función `unserialize()`, es posible omitir el control de autenticación e iniciar sesión como administrador, cuyo `userid` es 2:

```php
a:2:{s:11:"autologinid";b:1;s:6:"userid";s:1:"2";}  // valor original: a:2:{s:11:"autologinid";s:32:"8b8e9715d12e4ca12c4c3eb4865aaf6a";s:6:"userid";s:4:"1337";}
```

Desglosemos lo que hicimos en esta cadena:

1. `autologinid` ahora es un booleano establecido en `true`: esto se puede ver reemplazando el valor MD5 del hash de la contraseña (`s:32:"8b8e9715d12e4ca12c4c3eb4865aaf6a"`) con `b:1`
2. `userid` ahora se establece en el ID de administrador: esto se puede ver en la última pieza de la cadena, donde reemplazamos nuestro ID de usuario regular (`s:4:"1337"`) con `s:1:"2"`

## Herramientas

- [WebGoat](https://owasp.org/www-project-webgoat/)
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)

## Referencias

- [Niels Teusink: omisión de autenticación phpBB 2.0.12](http://blog.teusink.net/2008/12/classic-bug-phpbb-2012-authentication.html)
- [David Endler: "Explotación y predicción de fuerza bruta de ID de sesión"](https://www.cgisecurity.com/lib/SessionIDs.pdf)