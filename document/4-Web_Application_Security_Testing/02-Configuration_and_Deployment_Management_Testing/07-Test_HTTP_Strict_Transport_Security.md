# Prueba de HTTP Strict Transport Security

|ID          |
|------------|
|WSTG-CONF-07|

## Resumen

La función HTTP Strict Transport Security (HSTS) permite a un servidor web informar al navegador del usuario, a través de un encabezado de respuesta especial, que nunca debe establecer una conexión HTTP no encriptada a los servidores de dominio especificados. En su lugar, debe establecer automáticamente todas las solicitudes de conexión para acceder al sitio a través de HTTPS. Esto también evita que los usuarios anulen los errores de certificado.

Dada la importancia de esta medida de seguridad, es prudente verificar que el sitio esté utilizando este encabezado HTTP para asegurar que todos los datos viajen encriptados entre el navegador web y el servidor.

El encabezado de seguridad de transporte estricto HTTP utiliza tres directivas específicas:

- `max-age`: para indicar el número de segundos que el navegador debe convertir automáticamente todas las solicitudes HTTP a HTTPS.
- `includeSubDomains`: para indicar que todos los subdominios relacionados deben usar HTTPS.
- `preload` No oficial: para indicar que el dominio(s) están en la lista(s) de precarga y que los navegadores nunca deben conectarse sin HTTPS.
    - Aunque esto es compatible con todos los principales navegadores, no es parte oficial de la especificación. (Ver [hstspreload.org](https://hstspreload.org/) para más información.)

Aquí hay un ejemplo de la implementación del encabezado HSTS:

`Strict-Transport-Security: max-age=31536000; includeSubDomains`

La presencia de este encabezado debe verificarse, ya que su ausencia podría llevar a problemas de seguridad tales como:

- Atacantes interceptando y accediendo a la información transferida a través de un canal de red no encriptado.
- Atacantes realizando ataques de hombre en el medio (MITM) aprovechando usuarios que aceptan certificados no confiables.
- Usuarios que por error ingresan una dirección en el navegador usando HTTP en lugar de HTTPS, o usuarios que hacen clic en un enlace en una aplicación web que incorrectamente usa el protocolo HTTP.

## Objetivos de la Prueba

- Revisar el encabezado HSTS y su validez.

## Cómo Probar

- Confirmar la presencia del encabezado HSTS examinando la respuesta del servidor a través de un proxy interceptante.
- Usar curl de la siguiente manera:

```bash
$ curl -s -D- https://owasp.org | grep -i strict-transport-security:
Strict-Transport-Security: max-age=31536000
```

## Referencias

- [OWASP HTTP Strict Transport Security](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [OWASP Appsec Tutorial Series - Episode 4: Strict Transport Security](https://www.youtube.com/watch?v=zEV3HOuM_Vw)
- [HSTS Specification](https://tools.ietf.org/html/rfc6797)
- [Enable HTTP Strict Transport Security In Apache](https://https.cio.gov/hsts/)
- [Enable HTTP Strict Transport Security In Nginx](https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/)