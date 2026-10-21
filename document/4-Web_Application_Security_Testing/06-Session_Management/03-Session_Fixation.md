# Fijación de sesión

|ID          |
|------------|
|WSTG-SESS-03|

## Resumen

La fijación de sesión se produce por la práctica insegura de conservar el mismo valor de las cookies de sesión antes y después de la autenticación. Esto suele ocurrir cuando se usan cookies de sesión para almacenar información de estado incluso antes del inicio de sesión, por ejemplo, para agregar artículos a un carrito de compras antes de autenticarse para pagar.

En el caso genérico de explotación de vulnerabilidades de fijación de sesión, un atacante puede obtener un conjunto de cookies de sesión del sitio objetivo sin autenticarse primero. Luego, el atacante puede forzar estas cookies en el navegador de la víctima mediante distintas técnicas. Si la víctima se autentica posteriormente en el sitio objetivo y las cookies no se renuevan al iniciar sesión, las cookies de sesión elegidas por el atacante identificarán a la víctima. Así, el atacante podrá suplantar a la víctima usando esas cookies conocidas.

Este problema se puede solucionar renovando las cookies de sesión después del proceso de autenticación. Como alternativa, se puede prevenir el ataque garantizando la integridad de las cookies de sesión. Para protegerse de atacantes de red, es decir, atacantes que controlan la red utilizada por la víctima, use [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) completo o agregue el prefijo `__Host-` / `__Secure-` al nombre de la cookie.

La adopción completa de HSTS ocurre cuando un host activa HSTS para sí mismo y para todos sus subdominios. Esto se describe en un artículo titulado *Testing for Integrity Flaws in Web Sessions*, de Stefano Calzavara, Alvise Rabitti, Alessio Ragazzo y Michele Bugliesi.

## Objetivos de la prueba

- Analizar el mecanismo de autenticación y su flujo.
- Forzar cookies y evaluar el impacto.

## Cómo probar

En esta sección se explica la estrategia de prueba que se mostrará en la siguiente sección.

El primer paso es hacer una solicitud al sitio que se va a probar (*p. ej.*, `www.example.com`). Si el evaluador envía la siguiente solicitud:

```http
GET / HTTP/1.1
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

A continuación, si el evaluador se autentica correctamente en la aplicación mediante la siguiente solicitud POST a `https://www.example.com/authentication.php`:

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

El evaluador observa la siguiente respuesta del servidor:

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

Como no se ha emitido ninguna cookie nueva después de una autenticación correcta, el evaluador sabe que es posible secuestrar la sesión, a menos que se garantice la integridad de la cookie de sesión.

El evaluador puede enviar un identificador de sesión válido a un usuario (posiblemente mediante una técnica de ingeniería social), esperar a que se autentique y, después, verificar que se hayan asignado privilegios a esa cookie.

### Prueba con cookies forzadas

Esta estrategia de prueba está dirigida a atacantes de red; por tanto, solo debe aplicarse a sitios sin adopción completa de HSTS (los sitios con adopción completa de HSTS son seguros, ya que todas sus cookies tienen integridad). Supondremos que hay dos cuentas de prueba en el sitio bajo prueba: una para la víctima y otra para el atacante. Simulamos una situación en la que el atacante fuerza en el navegador de la víctima todas las cookies que no se vuelven a emitir después del inicio de sesión y que no tienen integridad. Después de que la víctima inicie sesión, el atacante presenta las cookies forzadas al sitio para acceder a su cuenta. Si son suficientes para actuar en nombre de la víctima, es posible realizar un ataque de fijación de sesión.

Estos son los pasos para realizar la prueba:

1. Acceder a la página de inicio de sesión del sitio.
2. Guardar una instantánea del almacén de cookies antes de iniciar sesión, excluyendo las cookies cuyo nombre contenga el prefijo `__Host-` o `__Secure-`.
3. Iniciar sesión en el sitio como la víctima y acceder a cualquier página que ofrezca una función segura que requiera autenticación.
4. Restaurar el almacén de cookies a la instantánea tomada en el paso 2.
5. Activar la función segura identificada en el paso 3.
6. Observar si la operación del paso 5 se realizó correctamente. Si es así, el ataque tuvo éxito.
7. Vaciar el almacén de cookies, iniciar sesión como el atacante y acceder a la página del paso 3.
8. Escribir en el almacén de cookies, una por una, las cookies guardadas en el paso 2.
9. Volver a activar la función segura identificada en el paso 3.
10. Vaciar el almacén de cookies e iniciar sesión nuevamente como la víctima.
11. Observar si la operación del paso 9 se realizó correctamente en la cuenta de la víctima. Si es así, el ataque tuvo éxito; de lo contrario, el sitio es seguro frente a la fijación de sesión.

Se recomienda usar dos máquinas o navegadores distintos para la víctima y el atacante. Esto permite reducir el número de falsos positivos si la aplicación web realiza fingerprinting para verificar el acceso habilitado desde una cookie determinada. Una variante más breve, pero menos precisa, de la estrategia de prueba solo requiere una cuenta de prueba. Sigue los mismos pasos, pero se detiene en el paso 6.

## Remediación

Implementar la renovación del token de sesión después de que un usuario se autentique correctamente.

La aplicación siempre debe invalidar el ID de sesión existente antes de autenticar a un usuario y, si la autenticación se realiza correctamente, proporcionar otro ID de sesión.

## Herramientas

- [ZAP](https://www.zaproxy.org)

## Referencias

- [Fijación de sesión](https://owasp.org/www-community/attacks/Session_fixation)
- [ACROS Security](https://www.acrossecurity.com/papers/session_fixation.pdf)
- [Chris Shiflett](https://shiflett.org/articles/session-fixation)