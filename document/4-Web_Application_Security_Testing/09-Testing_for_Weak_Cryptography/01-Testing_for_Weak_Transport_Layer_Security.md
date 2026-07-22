# Pruebas de Seguridad de Capa de Transporte Débil

|ID          |
|------------|
|WSTG-CRYP-01|

## Resumen

Cuando se envía información entre el cliente y el servidor, debe estar cifrada y protegida para prevenir que un atacante pueda leerla o modificarla. Esto se hace más comúnmente usando HTTPS, que usa el protocolo [Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), un reemplazo para el antiguo protocolo Secure Socket Layer (SSL). TLS también proporciona una manera para que el servidor demuestre al cliente que se han conectado al servidor correcto, presentando un certificado digital confiable.

A lo largo de los años ha habido una gran cantidad de debilidades criptográficas identificadas en los protocolos SSL y TLS, así como en los cifrados que usan. Adicionalmente, muchas de las implementaciones de estos protocolos también han tenido vulnerabilidades serias. Como tal, es importante probar que los sitios no solo están implementando TLS, sino que lo están haciendo de manera segura.

## Objetivos de Prueba

- Validar la configuración del servicio.
- Revisar la fuerza criptográfica y validez del certificado digital.
- Asegurar que la seguridad TLS no sea evadible y esté propiamente implementada a través de la aplicación.

## Cómo Probar

Los problemas relacionados con la seguridad de capa de transporte pueden dividirse ampliamente en las siguientes áreas:

### Configuración del Servidor

Hay una gran cantidad de versiones de protocolo, cifrados y extensiones soportadas por TLS. Muchas de estas se consideran heredadas, y tienen debilidades criptográficas, tales como las listadas a continuación. Es probable que se identifiquen nuevas debilidades con el tiempo, por lo que esta lista podría estar incompleta.

- [SSLv2 (DROWN)](https://drownattack.com/)
- [SSLv3 (POODLE)](https://en.wikipedia.org/wiki/POODLE)
- [TLSv1.0 (BEAST)](https://www.acunetix.com/blog/web-security-zone/what-is-beast-attack/)
- [TLSv1.1 (Deprecado por RFC 8996)](https://tools.ietf.org/html/rfc8996)
- [Suites de cifrado EXPORT (FREAK)](https://en.wikipedia.org/wiki/FREAK)
- Cifrados NULL ([solo proporcionan autenticación](https://tools.ietf.org/html/rfc4785)).
- Cifrados anónimos (estos podrían estar soportados en servidores SMTP, como se discute en [RFC 7672](https://tools.ietf.org/html/rfc7672#section-8.2))
- [Cifrados RC4 (NOMORE)](https://www.rc4nomore.com/)
- Cifrados en modo CBC (BEAST, [Lucky 13](https://en.wikipedia.org/wiki/Lucky_Thirteen_attack))
- [Compresión TLS (CRIME)](https://en.wikipedia.org/wiki/CRIME)
- [Claves DHE Débiles (LOGJAM)](https://weakdh.org/)

El proyecto [TLSRef](https://tlsref.org/) (anteriormente operado por Mozilla) proporciona recomendaciones para los protocolos y cifrados que deberían usarse.

Debido a su compatibilidad con cifrados post-cuánticos, se debería preferir TLS 1.3 sobre TLS 1.2.

#### Explotabilidad

Debería enfatizarse que mientras muchos de estos ataques se han demostrado en un entorno de laboratorio, generalmente no se consideran prácticos de explotar en el mundo real, ya que requieren un ataque MitM (usualmente activo), y recursos significativos. Como tal, es poco probable que sean explotados por alguien que no sea estados nacionales.

### Certificados Digitales

#### Debilidades Criptográficas

Desde una perspectiva criptográfica, hay dos áreas principales que necesitan revisarse en un certificado digital: La firma del certificado mismo y el algoritmo de intercambio de claves usado durante el handshake TLS. Los siguientes puntos deberían considerarse al usar criptografía clásica:

- La fuerza del cifrado usado debería ser de al menos 128 bits de seguridad o superior.
- Para firmas clásicas deberías al menos usar RSA-3072 o ECDSA con P-256 en combinación con SHA-256 o más fuerte.
- Para un intercambio de claves clásico usa ECDHE con curvas elípticas P-256 o X25519 como mínimo.

Adicionalmente, tanto la firma como el algoritmo de intercambio de claves deberían soportar criptografía post-cuántica. Los siguientes puntos deberían considerarse:

- Para firmas post-cuánticas deberías usar al menos ML-DSA-44 (de acuerdo con [FIPS-204](https://csrc.nist.gov/pubs/fips/204/final)) o superior.
- Para intercambios de claves post-cuánticos deberías usar al menos ML-KEM-512 (de acuerdo con [FIPS-203](https://csrc.nist.gov/pubs/fips/203/final)) o superior.
- Puedes usar cifrados híbridos tales como X25519+ML-KEM-768 durante la transición a algoritmos post-cuánticos.

#### Validez

Así como ser criptográficamente seguro, el certificado también debe considerarse válido (o confiable). Esto significa que debe:

- Estar dentro del período de validez definido.
    - Cualquier certificado emitido después del 1 de septiembre de 2020 no debe tener una vida útil máxima de más de [398 días](https://blog.mozilla.org/security/2020/07/09/reducing-tls-certificate-lifespans-to-398-days/).
    - El período de validez de los certificados gradualmente se reducirá a una vida útil máxima de [47 días en marzo de 2029](https://github.com/cabforum/servercert/blob/main/docs/BR.md#11-overview).
- Estar firmado por una autoridad certificadora (CA) confiable.
    - Esto debería ser o bien una CA pública confiable para aplicaciones externas, o una CA interna para aplicaciones internas.
    - No marcar aplicaciones internas como teniendo certificados no confiables solo porque *tu* sistema no confía en la CA.
- Tener un Subject Alternate Name (SAN) que coincida con el hostname del sistema.
    - El campo Common Name (CN) es ignorado por los navegadores modernos, los cuales solo miran el SAN.
    - Asegurarse de que se está accediendo al sistema con el nombre correcto (por ejemplo, si se accede al host por IP entonces cualquier certificado parecerá no confiable).

Algunos certificados podrían emitirse para dominios wildcard (tales como `*.example.org`), lo que significa que pueden ser válidos para múltiples subdominios. Aunque conveniente, hay varias preocupaciones de seguridad alrededor de esto que deberían considerarse. Estas se discuten en la [Hoja de Referencia de Protección de Capa de Transporte de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html#carefully-consider-the-use-of-wildcard-certificates).

Los certificados también pueden filtrar información sobre sistemas internos o nombres de dominio en los campos Issuer y SAN, lo cual puede ser útil al intentar construir una imagen de la red interna o conducir actividades de ingeniería social.

### Vulnerabilidades de Implementación

A lo largo de los años ha habido vulnerabilidades en las varias implementaciones de TLS. Hay demasiadas para listar aquí, pero algunos ejemplos clave son:

- [Generador de Números Aleatorios Predecible de Debian OpenSSL](https://www.debian.org/security/2008/dsa-1571) (CVE-2008-0166)
- [Renegociación Insegura de OpenSSL](https://www.openssl.org/news/secadv/20091111.txt) (CVE-2009-3555)
- [OpenSSL Heartbleed](https://heartbleed.com) (CVE-2014-0160)
- [F5 TLS POODLE](https://support.f5.com/csp/article/K15882) (CVE-2014-8730)
- [Microsoft Schannel Denial of Service](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-066) (MS14-066 / CVE-2014-6321)

### Vulnerabilidades de Aplicación

Así como la configuración TLS subyacente está configurada de manera segura, la aplicación también necesita usarla de manera segura. Algunos de estos puntos se abordan en otra parte de esta guía:

- [No enviar datos sensibles sobre canales no cifrados (WSTG-CRYP-03)](03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md)
- [Establecer el encabezado HTTP Strict-Transport-Security (WSTG-CONF-07)](../02-Configuration_and_Deployment_Management_Testing/07-Test_HTTP_Strict_Transport_Security.md)
- [Establecer el flag Secure en cookies (WSTG-SESS-02)](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)

#### Contenido Activo Mixto

El contenido activo mixto es cuando recursos activos (tales como scripts a CSS) se cargan sobre HTTP no cifrado y se incluyen en una página segura (HTTPS). Esto es peligroso porque permitiría a un atacante modificar estos archivos (ya que se envían sin cifrar), lo cual podría permitirles ejecutar código arbitrario (JavaScript o CSS) en la página. El contenido pasivo (tales como imágenes) cargado sobre una conexión insegura también puede filtrar información o permitir a un atacante desfigurar la página, aunque es menos probable que lleve a un compromiso completo.

> Nota: los navegadores modernos bloquearán el contenido activo siendo cargado desde fuentes inseguras en páginas seguras.

#### Redirigiendo de HTTP a HTTPS

Muchos sitios aceptarán conexiones sobre HTTP no cifrado, y luego redirigirán inmediatamente al usuario a la versión segura (HTTPS) del sitio con una redirección `301 Moved Permanently`. La versión HTTPS del sitio entonces establece el encabezado `Strict-Transport-Security` para instruir al navegador a usar siempre HTTPS en el futuro.

Sin embargo, si un atacante es capaz de interceptar esta solicitud inicial, podría redirigir al usuario a un sitio malicioso, o usar una herramienta tal como [sslstrip](https://github.com/moxie0/sslstrip) para interceptar solicitudes subsiguientes.

Para defenderse contra este tipo de ataque, el sitio debe añadirse a la [lista de preload](https://hstspreload.org).

## Pruebas Automatizadas

Hay una gran cantidad de herramientas de escaneo que pueden usarse para identificar debilidades en la configuración SSL/TLS de un servicio, incluyendo tanto herramientas dedicadas como escáneres de vulnerabilidades de propósito general. Algunas de las más populares son:

- [Nmap](https://nmap.org) (varios scripts)
- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslscan](https://github.com/rbsec/sslscan)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [SSL Labs](https://www.ssllabs.com/ssltest/)
- [testssl.sh](https://github.com/drwetter/testssl.sh)

### Pruebas Manuales

También es posible llevar a cabo la mayoría de las verificaciones manualmente, usando herramientas de línea de comandos tales como `openssl s_client` o `gnutls-cli` para conectar con protocolos, cifrados u opciones específicos.

Al probar de esta manera, ser consciente de que la versión de OpenSSL o GnuTLS enviada con la mayoría de los sistemas modernos podría no soportar algunos protocolos desactualizados e inseguros tales como SSLv2 o cifrados EXPORT. Asegurarse de que la versión soporta las versiones desactualizadas antes de usarla para pruebas, o terminarás con falsos negativos.

También puede ser posible realizar pruebas limitadas usando un navegador web, ya que los navegadores modernos proporcionarán detalles de los protocolos y cifrados que se están usando en sus herramientas de desarrollador. También proporcionan una manera fácil de probar si un certificado se considera confiable, navegando al servicio y viendo si se presenta una advertencia de certificado.

## Referencias

- [Hoja de Referencia de Protección de Capa de Transporte de OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
- [TLSRef](https://docs.tlsref.org/)
- [CWE-1428: Reliance on HTTP instead of HTTPS](https://cwe.mitre.org/data/definitions/1428.html)
