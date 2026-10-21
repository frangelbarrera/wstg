# HTTP Strict Transport Security

|ID          |
|------------|
|WSTG-CONF-07|

## Resumen

La función HTTP Strict Transport Security (HSTS) permite a un servidor web informar al navegador del usuario, mediante un encabezado de respuesta especial, que nunca debe establecer una conexión HTTP sin cifrar con los servidores del dominio especificado. En su lugar, debe establecer automáticamente todas las conexiones solicitadas para acceder al sitio a través de HTTPS. Esto también evita que los usuarios omitan los errores de certificado.

Su ausencia o una configuración débil puede provocar problemas de seguridad, como los siguientes:

- Que los atacantes intercepten y accedan a información transferida a través de un canal de red sin cifrar.
- Que los atacantes realicen ataques de intermediario (MITM) aprovechando que los usuarios aceptan certificados que no son de confianza, o degradando o eliminando una redirección a HTTPS (eliminación de SSL).
- Que los usuarios introduzcan por error una dirección en el navegador usando HTTP en lugar de HTTPS, o hagan clic en un enlace de una aplicación web que use incorrectamente el protocolo HTTP.

## Objetivos de la prueba

- Revisar el encabezado HSTS, sus directivas y sus valores para comprobar que se ajustan a las prácticas recomendadas actuales.
- Confirmar que el sitio está protegido contra la brecha de confianza de la primera solicitud, ya sea mediante precarga o mediante una ruta HTTP a HTTPS protegida de otro modo.
- Confirmar que HSTS se aplica de forma coherente en todo el dominio y los subdominios pertinentes, y que no oculta problemas de contenido mixto.

## Cómo probar

- Confirmar la presencia del encabezado HSTS en la respuesta HTTPS examinando la respuesta del servidor mediante un proxy interceptante.
- Usar curl de la siguiente manera:

```bash
$ curl -s -D- https://owasp.org | grep -i strict-transport-security:
Strict-Transport-Security: max-age=31536000
```

- Evaluar los valores de las directivas. Un ejemplo de encabezado apto para la precarga es `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`:
    - `max-age` (número de segundos hasta que el navegador deje de forzar HTTPS para este host) debe ser como mínimo `31536000` (1 año), o `63072000` (2 años) si el dominio se va a enviar a la lista de precarga de HSTS o ya está incluido en ella; señalar cualquier valor considerablemente inferior o `max-age=0`, salvo que se trate de una reversión intencional para desactivar HSTS.
    - Comprobar si está presente `includeSubDomains` (extiende la política a todos los subdominios) y, de ser así, revisar una muestra de subdominios —incluidos los que están fuera de la aplicación inmediata (correo, herramientas internas, entorno de prueba, etc.)— para confirmar que todos sirven HTTPS válido. Los navegadores que hayan almacenado la política en caché no podrán acceder a un subdominio que no pueda servir HTTPS.
    - Comprobar si está presente `preload` y, de ser así, confirmar que el dominio aparece realmente en la [lista de precarga HSTS de Chromium](https://hstspreload.org/), que utilizan Chrome, Firefox, Safari y Edge (la directiva `preload` en sí no tiene efecto en los navegadores; solo importa que el dominio figure realmente en la lista):

```bash
curl -s "https://hstspreload.org/api/v2/status?domain=example.com"
```

  Si está presente `preload` pero el dominio aún no aparece en la lista (o se eliminó), confirmar que se trata de un envío en curso y no de una directiva obsoleta o sin efecto.

- Comprobar la ruta de la primera solicitud y la redirección: HSTS solo protege las solicitudes realizadas después de que el navegador haya recibido el encabezado una vez para ese host (confianza en el primer uso). Por lo tanto, la primera visita de un usuario —o cualquier visita después de que la política almacenada en caché haya caducado— todavía se realiza mediante HTTP sin cifrar, a menos que el host esté precargado. Esa primera solicitud o redirección puede interceptarse o eliminarse antes de que el navegador llegue a conocer HSTS. Probarlo directamente:

```bash
curl -s -D- http://example.com | head -n 5
```

  Confirmar que esto devuelve una redirección (`301`/`308`) al equivalente HTTPS y que la respuesta HTTPS resultante incluye el encabezado HSTS (según la especificación, los navegadores ignoran el encabezado si se envía mediante HTTP sin cifrar, por lo que debe enviarse de nuevo en el lado HTTPS de la redirección). `preload` es el único mecanismo que cierra por completo esta brecha, ya que la política se incluye en el navegador en lugar de aprenderse a partir de una respuesta anterior.

- Comprobar la interacción con el contenido mixto: HSTS actualiza por la fuerza a HTTPS la navegación de nivel superior y las solicitudes de subrecursos al mismo host, pero no modifica las referencias explícitas `http://` a otros orígenes en el marcado. Inspeccionar el código fuente de la página o la pestaña de red en busca de referencias absolutas `http://otherhost/...`: estas siguen sujetas al bloqueo habitual de contenido mixto del navegador, independientemente de HSTS (véase [Intercambio de recursos entre orígenes](../11-Client-side/07-Cross_Origin_Resource_Sharing.md)). Por separado, confirmar que los enlaces `http://` al mismo host en el marcado usan HTTPS por diseño, en lugar de simplemente «funcionar» porque HSTS los actualiza silenciosamente; esa actualización puede ocultar un enlace que debería haberse corregido.

- En el caso de las aplicaciones detrás de una CDN, un balanceador de carga o un proxy inverso, confirmar que el encabezado se agrega en el perímetro que termina TLS para los usuarios finales, o que se conserva hasta llegar a ese punto, y no solo en un origen interno.

### Envío a la lista de precarga

El envío a la lista de precarga de HSTS ([hstspreload.org](https://hstspreload.org/)) es un paso manual y fuera de banda; no se habilita únicamente mediante el encabezado. Antes de recomendar el envío, confirmar que se cumplen realmente los requisitos previos:

- Servir un certificado HTTPS válido en el dominio base y redirigir a HTTPS todas las solicitudes HTTP a ese dominio.
- Servir HTTPS con un certificado válido en el subdominio `www` si existe un registro DNS para este.
- Enviar el encabezado `Strict-Transport-Security` en la respuesta HTTPS del dominio base, con un `max-age` de al menos `31536000` (un año, aunque el sitio recomienda `63072000`), y con `includeSubDomains` y `preload` presentes.
- Confirmar que todos los subdominios, incluidos los que están fuera de la aplicación inmediata (correo, herramientas internas, entorno de prueba, etc.), pueden servir HTTPS, ya que `includeSubDomains` aplica HTTPS forzado por la precarga a todos ellos.

Como los cambios en la lista de precarga se distribuyen con las versiones de los navegadores y son difíciles y lentos de revertir (la eliminación puede tardar meses en propagarse a los usuarios que usan versiones antiguas del navegador almacenadas en caché), el envío debe considerarse una decisión organizativa irreversible. Confirmar que está autorizado antes de enviarlo y no enviar el dominio de un objetivo durante una evaluación.

## Remediación

- Enviar `Strict-Transport-Security` en todas las respuestas HTTPS, con un `max-age` de al menos un año.
- Redirigir todo el tráfico HTTP a HTTPS y asegurarse de que el destino de la redirección devuelva el encabezado HSTS.
- Usar `includeSubDomains` una vez que se haya confirmado que todos los subdominios admiten HTTPS.
- Considerar el uso de `preload` y el envío a la lista de precarga de HSTS para los dominios que necesiten protección desde la primera conexión, tras confirmar los requisitos previos anteriores y obtener la aprobación de la organización, dada la dificultad de revertirlo.
- No confiar en el comportamiento de actualización del contenido mixto en lugar de corregir los enlaces `http://` codificados directamente en el marcado.

## Herramientas

- [hstspreload.org](https://hstspreload.org/) - consulta del estado y envío a la lista de precarga
- [SSL Labs Server Test](https://www.ssllabs.com/ssltest/) - informa sobre la configuración HSTS junto con la configuración TLS
- [ZAP](https://www.zaproxy.org/) - la regla de análisis pasivo señala la falta de HSTS o una configuración débil

## Referencias

- [Hoja de referencia de OWASP sobre HTTP Strict Transport Security](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [Serie de tutoriales Appsec de OWASP - Episodio 4: Strict Transport Security](https://www.youtube.com/watch?v=zEV3HOuM_Vw)
- [RFC 6797 - HTTP Strict Transport Security (HSTS)](https://tools.ietf.org/html/rfc6797)
- [hstspreload.org - Requisitos para el envío a la lista de precarga](https://hstspreload.org/)
- [Código fuente de la lista de precarga HSTS de Chromium](https://chromium.googlesource.com/chromium/src/+/main/net/http/transport_security_state_static.json)
- [MDN: Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security)
- [Habilitar HTTP Strict Transport Security en Nginx](https://blog.nginx.org/blog/http-strict-transport-security-hsts-and-nginx)