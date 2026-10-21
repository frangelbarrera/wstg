# Seguridad débil de la capa de transporte

|ID          |
|------------|
|WSTG-CRYP-01|

## Resumen

Cuando se envía información entre el cliente y el servidor, debe cifrarse y protegerse para evitar que un atacante pueda leerla o modificarla. Esto suele hacerse mediante HTTPS, que utiliza el protocolo [Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), sustituto del antiguo protocolo Secure Socket Layer (SSL). TLS también permite que el servidor demuestre al cliente que se ha conectado al servidor correcto, presentando un certificado digital de confianza.

A lo largo de los años, se han identificado numerosas debilidades criptográficas en los protocolos SSL y TLS, así como en los cifrados que utilizan. Además, muchas implementaciones de estos protocolos también han presentado vulnerabilidades graves. Por ello, es importante comprobar que los sitios no solo implementen TLS, sino que lo hagan de forma segura.

## Objetivos de la prueba

- Validar la configuración del servicio.
- Revisar la solidez criptográfica y la validez del certificado digital.
- Asegurarse de que no sea posible eludir la seguridad de TLS y de que esté implementada correctamente en toda la aplicación.

## Cómo realizar la prueba

Los problemas relacionados con la seguridad de la capa de transporte pueden dividirse, a grandes rasgos, en las siguientes áreas:

### Configuración del servidor

TLS admite numerosas versiones de protocolos, cifrados y extensiones. Muchos de ellos se consideran heredados y presentan debilidades criptográficas, como las que se indican a continuación. Es probable que con el tiempo se identifiquen nuevas debilidades, por lo que esta lista podría estar incompleta.

- [SSLv2 (DROWN)](https://drownattack.com/)
- [SSLv3 (POODLE)](https://en.wikipedia.org/wiki/POODLE)
- [TLSv1.0 (BEAST)](https://www.acunetix.com/blog/web-security-zone/what-is-beast-attack/)
- [TLSv1.1 (obsoleto según RFC 8996)](https://tools.ietf.org/html/rfc8996)
- [Suites de cifrado EXPORT (FREAK)](https://en.wikipedia.org/wiki/FREAK)
- Cifrados NULL ([solo proporcionan autenticación](https://tools.ietf.org/html/rfc4785)).
- Cifrados anónimos (pueden admitirse en servidores SMTP, como se explica en [RFC 7672](https://tools.ietf.org/html/rfc7672#section-8.2))
- [Cifrados RC4 (NOMORE)](https://www.rc4nomore.com/)
- Cifrados en modo CBC (BEAST, [Lucky 13](https://en.wikipedia.org/wiki/Lucky_Thirteen_attack))
- [Compresión TLS (CRIME)](https://en.wikipedia.org/wiki/CRIME)
- [Claves DHE débiles (LOGJAM)](https://weakdh.org/)

El proyecto [TLSRef](https://tlsref.org/) (anteriormente gestionado por Mozilla) ofrece recomendaciones sobre los protocolos y cifrados que deberían utilizarse.

Debido a su compatibilidad con los cifrados poscuánticos, se debería preferir TLS 1.3 a TLS 1.2.

#### Explotabilidad

Cabe destacar que, aunque muchos de estos ataques se han demostrado en un entorno de laboratorio, por lo general no se consideran prácticos de explotar en el mundo real, ya que requieren un ataque MitM (normalmente activo) y recursos considerables. Por ello, es poco probable que los explote alguien que no sean Estados nación.

### Certificados digitales

#### Debilidades criptográficas

Desde el punto de vista criptográfico, hay dos áreas principales que deben revisarse en un certificado digital: la firma del propio certificado y el algoritmo de intercambio de claves utilizado durante el protocolo de enlace TLS. Al utilizar criptografía clásica, deberían tenerse en cuenta los siguientes puntos:

- La solidez del cifrado utilizado debería ser de al menos 128 bits de seguridad.
- Para las firmas clásicas, se debería utilizar como mínimo RSA-3072 o ECDSA con P-256, en combinación con SHA-256 o un algoritmo más robusto.
- Para el intercambio clásico de claves, se debería utilizar como mínimo ECDHE con las curvas elípticas P-256 o X25519.

Además, tanto la firma como el algoritmo de intercambio de claves deberían admitir criptografía poscuántica. Deberían tenerse en cuenta los siguientes puntos:

- Para las firmas poscuánticas, se debería utilizar como mínimo ML-DSA-44 (según [FIPS-204](https://csrc.nist.gov/pubs/fips/204/final)) o una variante superior.
- Para el intercambio de claves poscuántico, se debería utilizar como mínimo ML-KEM-512 (según [FIPS-203](https://csrc.nist.gov/pubs/fips/203/final)) o una variante superior.
- Se pueden utilizar cifrados híbridos como X25519+ML-KEM-768 durante la transición a los algoritmos poscuánticos.

#### Validez

Además de ser criptográficamente seguro, el certificado también debe considerarse válido (o de confianza). Esto significa que debe:

- Estar dentro del período de validez definido.
    - Los certificados emitidos después del 1 de septiembre de 2020 no deben tener una vida útil máxima superior a [398 días](https://blog.mozilla.org/security/2020/07/09/reducing-tls-certificate-lifespans-to-398-days/).
    - El período de validez de los certificados se reducirá gradualmente hasta alcanzar una vida útil máxima de [47 días en marzo de 2029](https://github.com/cabforum/servercert/blob/main/docs/BR.md#11-overview).
- Estar firmado por una autoridad certificadora (CA) de confianza.
    - En el caso de aplicaciones accesibles desde el exterior, debería tratarse de una CA pública de confianza; para las aplicaciones internas, de una CA interna.
    - No se deben señalar las aplicaciones internas por tener certificados no confiables solo porque *tu* sistema no confíe en la CA.
- Tener un Subject Alternate Name (SAN) que coincida con el nombre de host del sistema.
    - Los navegadores modernos ignoran el campo Common Name (CN) y solo consultan el SAN.
    - Asegúrate de acceder al sistema con el nombre correcto (por ejemplo, si accedes al host mediante su dirección IP, cualquier certificado aparecerá como no confiable).

Algunos certificados pueden emitirse para dominios comodín (como `*.example.org`), lo que significa que pueden ser válidos para varios subdominios. Aunque son prácticos, hay varias cuestiones de seguridad que deberían tenerse en cuenta. Se explican en la [Hoja de referencia de seguridad de la capa de transporte de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html#carefully-consider-the-use-of-wildcard-certificates).

Los certificados también pueden filtrar información sobre sistemas internos o nombres de dominio en los campos Issuer y SAN, lo que puede resultar útil para obtener una imagen de la red interna o llevar a cabo actividades de ingeniería social.

### Vulnerabilidades de implementación

A lo largo de los años, se han descubierto vulnerabilidades en diversas implementaciones de TLS. Hay demasiadas para enumerarlas aquí, pero algunos ejemplos importantes son:

- [Generador de números aleatorios predecibles de Debian OpenSSL](https://www.debian.org/security/2008/dsa-1571) (CVE-2008-0166)
- [Renegociación insegura de OpenSSL](https://www.openssl.org/news/secadv/20091111.txt) (CVE-2009-3555)
- [OpenSSL Heartbleed](https://heartbleed.com) (CVE-2014-0160)
- [F5 TLS POODLE](https://my.f5.com/manage/s/article/K15882) (CVE-2014-8730)
- [Denegación de servicio de Microsoft Schannel](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-066) (MS14-066 / CVE-2014-6321)

### Vulnerabilidades de la aplicación

Además de configurar de forma segura la configuración TLS subyacente, la aplicación también debe utilizarla de forma segura. Algunos de estos puntos se abordan en otras partes de esta guía:

- [No enviar datos sensibles por canales no cifrados (WSTG-CRYP-03)](03-Sensitive_Information_Sent_via_Unencrypted_Channels.md)
- [Configurar el encabezado HTTP Strict-Transport-Security (WSTG-CONF-07)](../02-Configuration_and_Deployment_Management/07-HTTP_Strict_Transport_Security.md)
- [Configurar el atributo Secure en las cookies (WSTG-SESS-02)](../06-Session_Management/02-Cookies_Attributes.md)

#### Contenido activo mixto

El contenido activo mixto se produce cuando se cargan recursos activos (como scripts o CSS) mediante HTTP sin cifrar y se incluyen en una página segura (HTTPS). Esto es peligroso porque permitiría a un atacante modificar esos archivos (ya que se envían sin cifrar), lo que podría permitirle ejecutar código arbitrario (JavaScript o CSS) en la página. El contenido pasivo (como las imágenes) cargado mediante una conexión insegura también puede filtrar información o permitir que un atacante altere la página, aunque es menos probable que provoque una vulneración completa.

> Nota: los navegadores modernos bloquearán la carga de contenido activo desde fuentes inseguras en páginas seguras.

#### Redirección de HTTP a HTTPS

Muchos sitios aceptan conexiones mediante HTTP sin cifrar y, a continuación, redirigen inmediatamente al usuario a la versión segura (HTTPS) del sitio con una redirección `301 Moved Permanently`. La versión HTTPS del sitio establece entonces el encabezado `Strict-Transport-Security` para indicar al navegador que utilice siempre HTTPS en adelante.

Sin embargo, si un atacante puede interceptar esta solicitud inicial, podría redirigir al usuario a un sitio malicioso o utilizar una herramienta como [sslstrip](https://github.com/moxie0/sslstrip) para interceptar solicitudes posteriores.

Para defenderse de este tipo de ataque, el sitio debe añadirse a la [lista de precarga](https://hstspreload.org).

## Pruebas automatizadas

Hay numerosas herramientas de análisis que pueden utilizarse para identificar debilidades en la configuración SSL/TLS de un servicio, tanto herramientas específicas como escáneres de vulnerabilidades de propósito general. Algunas de las más conocidas son:

- [Nmap](https://nmap.org) (varios scripts)
- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslscan](https://github.com/rbsec/sslscan)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [SSL Labs](https://www.ssllabs.com/ssltest/)
- [testssl.sh](https://github.com/drwetter/testssl.sh)

### Pruebas manuales

También es posible realizar manualmente la mayoría de las comprobaciones con herramientas de línea de comandos, como `openssl s_client` o `gnutls-cli`, para conectarse con protocolos, cifrados u opciones específicos.

Al realizar pruebas de este modo, ten en cuenta que la versión de OpenSSL o GnuTLS incluida en la mayoría de los sistemas modernos podría no admitir algunos protocolos obsoletos e inseguros, como SSLv2 o los cifrados EXPORT. Asegúrate de que tu versión admite las versiones obsoletas antes de utilizarla para las pruebas; de lo contrario, obtendrás falsos negativos.

También es posible realizar pruebas limitadas con un navegador web, ya que los navegadores modernos muestran en sus herramientas para desarrolladores los protocolos y cifrados que se están utilizando. También ofrecen una forma sencilla de comprobar si un certificado se considera de confianza: basta con acceder al servicio y ver si aparece una advertencia sobre el certificado.

## Referencias

- [Hoja de referencia de seguridad de la capa de transporte de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
- [TLSRef](https://docs.tlsref.org/)
- [CWE-1428: Reliance on HTTP instead of HTTPS](https://cwe.mitre.org/data/definitions/1428.html)