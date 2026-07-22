# Pruebas de Secuestro de Sesión

|ID          |
|------------|
|WSTG-SESS-09|

## Resumen

Un atacante que obtiene acceso a las cookies de sesión de un usuario puede suplantarlo presentando dichas cookies. Este ataque se conoce como secuestro de sesión. Cuando se consideran atacantes de red, es decir, atacantes que controlan la red utilizada por la víctima, las cookies de sesión pueden quedar expuestas indebidamente al atacante a través de HTTP. Para prevenir esto, las cookies de sesión deben marcarse con el atributo `Secure` para que solo se comuniquen a través de HTTPS.

Tenga en cuenta que el atributo `Secure` también debe usarse cuando la aplicación web se despliega completamente sobre HTTPS; de lo contrario, es posible el siguiente ataque de robo de cookies. Suponga que `example.com` se despliega completamente sobre HTTPS, pero no marca sus cookies de sesión como `Secure`. Los siguientes pasos de ataque son posibles:

1. La víctima envía una solicitud a `https://another-site.com`.
2. El atacante corrompe la respuesta correspondiente para que active una solicitud a `https://example.com`.
3. El navegador ahora intenta acceder a `https://example.com`.
4. Aunque la solicitud falla, las cookies de sesión se filtran en claro a través de HTTP.

Alternativamente, el secuestro de sesión puede prevenirse prohibiendo el uso de HTTP mediante [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security). Tenga en cuenta que hay una sutileza aquí relacionada con el alcance de las cookies. En particular, se requiere la adopción completa de HSTS cuando las cookies de sesión se emiten con el atributo `Domain` establecido.

La adopción completa de HSTS se describe en un artículo llamado *Testing for Integrity Flaws in Web Sessions* de Stefano Calzavara, Alvise Rabitti, Alessio Ragazzo y Michele Bugliesi. La adopción completa de HSTS ocurre cuando un host activa HSTS para sí mismo y todos sus subdominios. La adopción parcial de HSTS es cuando un host activa HSTS solo para sí mismo.

Con el atributo `Domain` establecido, las cookies de sesión pueden compartirse entre subdominios. El uso de HTTP con subdominios debe evitarse para prevenir la divulgación de cookies no cifradas enviadas a través de HTTP. Para ejemplificar esta vulnerabilidad de seguridad, suponga que el sitio `example.com` activa HSTS sin la opción `includeSubDomains`. El sitio emite cookies de sesión con el atributo `Domain` establecido en `example.com`. El siguiente ataque es posible:

1. La víctima envía una solicitud a `https://another-site.com`.
2. El atacante corrompe la respuesta correspondiente para que active una solicitud a `https://fake.example.com`.
3. El navegador ahora intenta acceder a `https://fake.example.com`, lo cual está permitido por la configuración de HSTS.
4. Dado que la solicitud se envía a un subdominio de `example.com` con el atributo `Domain` establecido, incluye las cookies de sesión, que se filtran en claro a través de HTTP.

Debe activarse HSTS completo en el dominio apex para prevenir este ataque.

## Objetivos de la Prueba

- Identificar cookies de sesión vulnerables.
- Secuestrar cookies vulnerables y evaluar el nivel de riesgo.

## Cómo Probar

La estrategia de prueba se dirige a atacantes de red, por lo tanto, solo necesita aplicarse a sitios sin adopción completa de HSTS (los sitios con adopción completa de HSTS son seguros, ya que sus cookies no se comunican a través de HTTP). Asumimos tener dos cuentas de prueba en el sitio bajo prueba, una para actuar como víctima y otra como atacante. Simulamos un escenario donde el atacante roba todas las cookies que no están protegidas contra divulgación a través de HTTP, y las presenta al sitio para acceder a la cuenta de la víctima. Si estas cookies son suficientes para actuar en nombre de la víctima, el secuestro de sesión es posible.

Estos son los pasos para ejecutar esta prueba:

1. Inicie sesión en el sitio como la víctima y llegue a cualquier página que ofrezca una función segura que requiera autenticación.
2. Elimine del almacén de cookies todas las cookies que satisfagan cualquiera de las siguientes condiciones.
    - En caso de que no haya adopción de HSTS: el atributo `Secure` está establecido.
    - En caso de adopción parcial de HSTS: el atributo `Secure` está establecido o el atributo `Domain` no está establecido.
3. Guarde una instantánea del almacén de cookies.
4. Active la función segura identificada en el paso 1.
5. Observe si la operación en el paso 4 se realizó con éxito. Si es así, el ataque fue exitoso.
6. Limpie el almacén de cookies, inicie sesión como el atacante y llegue a la página en el paso 1.
7. Escriba en el almacén de cookies, una por una, las cookies guardadas en el paso 3.
8. Active nuevamente la función segura identificada en el paso 1.
9. Limpie el almacén de cookies e inicie sesión nuevamente como la víctima.
10. Observe si la operación en el paso 8 se realizó con éxito en la cuenta de la víctima. Si es así, el ataque fue exitoso; de lo contrario, el sitio es seguro contra el secuestro de sesión.

Recomendamos usar dos máquinas o navegadores diferentes para la víctima y el atacante. Esto permite disminuir el número de falsos positivos si la aplicación web realiza huellas dactilares para verificar el acceso habilitado desde una cookie dada. Una variante más corta pero menos precisa de la estrategia de prueba solo requiere una cuenta de prueba. Sigue el mismo patrón, pero se detiene en el paso 5 (tenga en cuenta que esto hace que el paso 3 sea inútil).

## Herramientas

- [ZAP](https://www.zaproxy.org)
- [JHijack - una herramienta de secuestro de sesión numérica](https://sourceforge.net/projects/jhijack/)