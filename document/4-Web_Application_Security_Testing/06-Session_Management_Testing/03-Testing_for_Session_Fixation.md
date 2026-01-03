# Prueba de fijación de sesión

|ID          |
|------------|
|WSTG-SESS-03|

## Resumen

La fijación de sesión se habilita mediante la práctica insegura de preservar el mismo valor de las cookies de sesión antes y después de la autenticación. Esto generalmente ocurre cuando las cookies de sesión se utilizan para almacenar información de estado incluso antes del inicio de sesión, por ejemplo, para agregar artículos a un carrito de compras antes de autenticarse para el pago.

En el exploit genérico de vulnerabilidades de fijación de sesión, un atacante puede obtener un conjunto de cookies de sesión del sitio objetivo sin autenticarse primero. El atacante puede entonces forzar estas cookies en el navegador de la víctima utilizando diferentes técnicas. Si la víctima posteriormente se autentica en el sitio objetivo y las cookies no se refrescan tras el inicio de sesión, la víctima será identificada por las cookies de sesión elegidas por el atacante. El atacante entonces puede suplantar a la víctima con estas cookies conocidas.

Este problema se puede solucionar refrescando las cookies de sesión después del proceso de autenticación. Alternativamente, el ataque se puede prevenir asegurando la integridad de las cookies de sesión. Al considerar atacantes de red, es decir, atacantes que controlan la red utilizada por la víctima, utilice HSTS completo o agregue el prefijo `__Host-` / `__Secure-` al nombre de la cookie.

La adopción completa de HSTS ocurre cuando un host activa HSTS para sí mismo y todos sus subdominios. Esto se describe en un artículo llamado *Testing for Integrity Flaws in Web Sessions* de Stefano Calzavara, Alvise Rabitti, Alessio Ragazzo y Michele Bugliesi.

## Objetivos de la prueba

- Analizar el mecanismo de autenticación y su flujo.
- Forzar cookies y evaluar el impacto.

## Cómo probar

En esta sección se da una explicación de la estrategia de prueba que se mostrará en la siguiente sección.

El primer paso es hacer una solicitud al sitio a probar (*ej.* `www.example.com`). Si el probador solicita lo siguiente:

```http
GET / HTTP/1.1
Host: www.example.com
```

Obtendrá la siguiente respuesta:

```html
HTTP/1.1 200 OK
Date: Wed, 14 Aug 2008 08:45:11 GMT
Server: IBM_HTTP_Server
Set-Cookie: JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1; Path=/; secure
Cache-Control: no-cache="set-cookie,set-cookie2"
Expires: Thu, 01 Dec 1994 16:00:00 GMT
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html;charset=Cp1254
Content-Language: en-US
```

La aplicación establece un nuevo identificador de sesión, `JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1`, para el cliente.

A continuación, si el probador se autentica exitosamente en la aplicación con el siguiente POST a `https://www.example.com/authentication.php`:

```http
POST /authentication.php HTTP/1.1
Host: www.example.com
[...]
Referer: https://www.example.com
Cookie: JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1
Content-Type: application/x-www-form-urlencoded
Content-length: 57

Name=Meucci&wpPassword=secret!&wpLoginattempt=Log+in
```

El probador observa la siguiente respuesta del servidor:

```http
HTTP/1.1 200 OK
Date: Thu, 14 Aug 2008 14:52:58 GMT
Server: Apache/2.2.2 (Fedora)
X-Powered-By: PHP/5.1.6
Content-language: en
Cache-Control: private, must-revalidate, max-age=0
X-Content-Encoding: gzip
Content-length: 4090
Connection: close
Content-Type: text/html; charset=UTF-8
...
HTML data
...
```

Como no se ha emitido una nueva cookie tras una autenticación exitosa, el probador sabe que es posible realizar secuestro de sesión a menos que se asegure la integridad de la cookie de sesión.

El probador puede enviar un identificador de sesión válido a un usuario (posiblemente utilizando un truco de ingeniería social), esperar a que se autentique, y posteriormente verificar que se han asignado privilegios a esta cookie.

### Prueba con cookies forzadas

Esta estrategia de prueba está dirigida a atacantes de red, por lo tanto, solo necesita aplicarse a sitios sin adopción completa de HSTS (los sitios con adopción completa de HSTS son seguros, ya que todas sus cookies tienen integridad). Asumimos tener dos cuentas de prueba en el sitio bajo prueba, una para actuar como víctima y una para actuar como atacante. Simulamos un escenario donde el atacante fuerza en el navegador de la víctima todas las cookies que no son emitidas recién después del inicio de sesión y no tienen integridad. Después del inicio de sesión de la víctima, el atacante presenta las cookies forzadas al sitio para acceder a la cuenta de la víctima: si son suficientes para actuar en nombre de la víctima, la fijación de sesión es posible.

Aquí están los pasos para ejecutar esta prueba:

1. Llegar a la página de inicio de sesión del sitio.
2. Guardar una instantánea del jar de cookies antes de iniciar sesión, excluyendo cookies que contengan el prefijo `__Host-` o `__Secure-` en su nombre.
3. Iniciar sesión en el sitio como la víctima y llegar a cualquier página que ofrezca una función segura que requiera autenticación.
4. Establecer el jar de cookies a la instantánea tomada en el paso 2.
5. Activar la función segura identificada en el paso 3.
6. Observar si la operación en el paso 5 se ha realizado exitosamente. Si es así, el ataque fue exitoso.
7. Limpiar el jar de cookies, iniciar sesión como el atacante y llegar a la página en el paso 3.
8. Escribir en el jar de cookies, una por una, las cookies guardadas en el paso 2.
9. Activar nuevamente la función segura identificada en el paso 3.
10. Limpiar el jar de cookies e iniciar sesión nuevamente como la víctima.
11. Observar si la operación en el paso 9 se ha realizado exitosamente en la cuenta de la víctima. Si es así, el ataque fue exitoso; de lo contrario, el sitio es seguro contra la fijación de sesión.

Recomendamos usar dos máquinas o navegadores diferentes para la víctima y el atacante. Esto permite disminuir el número de falsos positivos si la aplicación web hace fingerprinting para verificar el acceso habilitado desde una cookie dada. Una variante más corta pero menos precisa de la estrategia de prueba solo requiere una cuenta de prueba. Sigue los mismos pasos, pero se detiene en el paso 6.

## Remediation

Implementar una renovación del token de sesión después de que un usuario se autentique exitosamente.

La aplicación siempre debe invalidar primero el ID de sesión existente antes de autenticar a un usuario, y si la autenticación es exitosa, proporcionar otro ID de sesión.

## Herramientas

- [ZAP](https://www.zaproxy.org)

## Referencias

- [Session Fixation](https://owasp.org/www-community/attacks/Session_fixation)
- [ACROS Security](https://www.acrossecurity.com/papers/session_fixation.pdf)
- [Chris Shiflett](https://shiflett.org/articles/session-fixation)