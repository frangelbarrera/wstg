# Enumerar Aplicaciones en el Servidor Web

|ID          |
|------------|
|WSTG-INFO-04|

## Resumen

Un paso primordial en las pruebas de vulnerabilidades de aplicaciones web es determinar qué aplicaciones particulares están alojadas en un servidor web. Muchas aplicaciones tienen vulnerabilidades conocidas y estrategias de ataque conocidas que pueden ser explotadas para obtener control remoto o explotar datos. Además, muchas aplicaciones a menudo están mal configuradas o no actualizadas, debido a la percepción de que solo se usan "internamente" y por lo tanto no existe amenaza.

Con la proliferación de servidores web virtuales, la relación tradicional 1:1 entre una dirección IP y un servidor web está perdiendo gran parte de su significado original. No es inusual tener múltiples sitios o aplicaciones cuyos nombres simbólicos se resuelven a la misma dirección IP. Este escenario no se limita a entornos de hosting, sino que también se aplica a entornos corporativos ordinarios.

Los profesionales de seguridad a veces reciben un conjunto de direcciones IP como objetivo para probar. Es discutible que este escenario sea más similar a un compromiso de tipo prueba de penetración, pero en cualquier caso se espera que tal asignación pruebe todas las aplicaciones web accesibles a través de este objetivo. El problema es que la dirección IP dada aloja un servicio HTTP en el puerto 80, pero si un probador accede a él especificando la dirección IP (que es todo lo que saben) informa "No hay servidor web configurado en esta dirección" o un mensaje similar. Pero ese sistema podría "ocultar" un número de aplicaciones web, asociadas a nombres simbólicos (DNS) no relacionados. Obviamente, el alcance del análisis se ve profundamente afectado dependiendo de si el probador prueba todas las aplicaciones o solo prueba las aplicaciones de las que son conscientes.

A veces, la especificación del objetivo es más rica. El probador puede recibir una lista de direcciones IP y sus correspondientes nombres simbólicos. Sin embargo, esta lista podría transmitir información parcial, es decir, podría omitir algunos nombres simbólicos y el cliente puede no ser siquiera consciente de eso (esto es más probable que suceda en organizaciones grandes).

Otros problemas que afectan el alcance de la evaluación están representados por aplicaciones web publicadas en URLs no obvias (por ejemplo, `https://www.example.com/some-strange-URL`), que no están referenciadas en otro lugar. Esto puede suceder ya sea por error (debido a configuraciones erróneas), o intencionalmente (por ejemplo, interfaces administrativas no publicitadas).

Para abordar estos problemas, es necesario realizar descubrimiento de aplicaciones web.

## Objetivos de Prueba

- Enumerar las aplicaciones dentro del alcance que existen en un servidor web.

## Cómo Probar

El descubrimiento de aplicaciones web es un proceso que apunta a identificar aplicaciones web en una infraestructura dada. Esta última se especifica usualmente como un conjunto de direcciones IP (tal vez un bloque de red), pero puede consistir en un conjunto de nombres simbólicos DNS o una mezcla de los dos. Esta información se entrega antes de la ejecución de una evaluación, ya sea una prueba de penetración de estilo clásico o una evaluación enfocada en aplicaciones. En ambos casos, a menos que las reglas de compromiso especifiquen lo contrario (por ejemplo, probar solo la aplicación ubicada en la URL `https://www.example.com/`), la evaluación debería esforzarse por ser lo más comprehensiva en alcance, es decir, debería identificar todas las aplicaciones accesibles a través del objetivo dado. Los siguientes ejemplos examinan algunas técnicas que pueden emplearse para lograr este objetivo.

> Algunas de las siguientes técnicas se aplican a servidores web orientados a Internet, a saber, servicios de búsqueda web basados en DNS y reverse-IP y el uso de motores de búsqueda. Los ejemplos usan direcciones IP privadas (como `192.168.1.100`), que, a menos que se indique lo contrario, representan direcciones IP *genéricas* y se usan solo con fines de anonimato.

Hay tres factores que influyen en cuántas aplicaciones están relacionadas con un nombre DNS dado (o una dirección IP):

1. **Diferente URL Base**

    El punto de entrada obvio para una aplicación web es `www.example.com`, es decir, con esta notación abreviada pensamos en la aplicación web originándose en `https://www.example.com/` (lo mismo aplica para HTTPS). Sin embargo, aunque esta es la situación más común, no hay nada forzando a la aplicación a comenzar en `/`.

    Por ejemplo, el mismo nombre simbólico puede estar asociado a tres aplicaciones web como: `https://www.example.com/app1` `https://www.example.com/app2` `https://www.example.com/app3`

    En este caso, la URL `https://www.example.com/` no estaría asociada a una página significativa. Las tres aplicaciones permanecerían **ocultas** a menos que el probador sepa explícitamente cómo acceder a ellas, es decir, el probador sabe *app1*, *app2* o *app3*. Usualmente no hay necesidad de publicar aplicaciones web de esta manera, a menos que el propietario no quiera que sean accesibles de manera estándar, y esté preparado para informar a los usuarios sobre su ubicación exacta. Esto no significa que estas aplicaciones sean secretas, solo que su existencia y ubicación no se publicita explícitamente.

2. **Puertos No Estándar**

    Mientras que las aplicaciones web usualmente viven en el puerto 80 (HTTP) y 443 (HTTPS), no hay nada fijo o obligatorio sobre estos números de puerto. De hecho, las aplicaciones web pueden estar asociadas con puertos TCP arbitrarios, y pueden ser referenciadas especificando el número de puerto como sigue: `http[s]://www.example.com:port/`. Por ejemplo, `https://www.example.com:20000/`.

3. **Hosts Virtuales**

    DNS permite que una sola dirección IP esté asociada con uno o más nombres simbólicos. Por ejemplo, la dirección IP `192.168.1.100` podría estar asociada a nombres DNS `www.example.com`, `helpdesk.example.com`, `webmail.example.com`. No es necesario que todos los nombres pertenezcan al mismo dominio DNS. Esta relación 1-a-N puede reflejarse para servir contenido diferente usando los llamados hosts virtuales. La información especificando el host virtual al que nos referimos está embebida en el [Host header](https://tools.ietf.org/html/rfc2616#section-14.23) de HTTP 1.1.

    Uno no sospecharía la existencia de otras aplicaciones web además del obvio `www.example.com`, a menos que conozcan `helpdesk.example.com` y `webmail.example.com`.

### Enfoques para Abordar el Problema 1 - URLs No Estándar

No hay manera de asegurar completamente la existencia de aplicaciones web nombradas no estándar. Siendo no estándar, no hay criterios fijos gobernando la convención de nomenclatura, sin embargo hay un número de técnicas que el probador puede usar para obtener alguna visión adicional.

Primero, si el servidor web está mal configurado y permite navegación de directorios, puede ser posible detectar estas aplicaciones. Los escáneres de vulnerabilidades pueden ayudar en este respecto.

Segundo, estas aplicaciones pueden ser referenciadas por otras páginas web y hay una posibilidad de que hayan sido spidered e indexadas por motores de búsqueda web. Si los probadores sospechan la existencia de tales aplicaciones **ocultas** en `www.example.com` podrían buscar usando el operador *site* y examinando el resultado de una consulta para `site: www.example.com`. Entre las URLs retornadas podría haber una apuntando a tal aplicación no obvia.

Otra opción es probar URLs que podrían ser candidatos probables para aplicaciones no publicadas. Por ejemplo, un frontend de web mail podría ser accesible desde URLs como `https://www.example.com/webmail`, `https://webmail.example.com/`, o `https://mail.example.com/`. Lo mismo se aplica para interfaces administrativas, que pueden ser publicadas en URLs ocultas (por ejemplo, una interfaz administrativa de Tomcat), y aún no referenciadas en ningún lugar. Así que hacer un poco de búsqueda estilo diccionario (o "adivinación inteligente") podría rendir algunos resultados. Los escáneres de vulnerabilidades pueden ayudar en este respecto.

### Enfoques para Abordar el Problema 2 - Puertos No Estándar

Es fácil verificar la existencia de aplicaciones web en puertos no estándar. Un escáner de puertos como Nmap es capaz de realizar reconocimiento de servicio mediante la opción `-sV`, e identificará servicios http[s] en puertos arbitrarios. Lo que se requiere es un escaneo completo de todo el espacio de direcciones de puerto TCP de 64k.

Por ejemplo, el siguiente comando buscará, con un escaneo TCP connect, todos los puertos abiertos en IP `192.168.1.100` e intentará determinar qué servicios están ligados a ellos (solo se muestran interruptores *esenciales* – Nmap cuenta con un amplio conjunto de opciones, cuya discusión está fuera de alcance):

`nmap –Pn –sT –sV –p0-65535 192.168.1.100`

Es suficiente examinar la salida y buscar HTTP o la indicación de servicios envueltos en TLS (que deberían ser probados para confirmar que son HTTPS). Por ejemplo, la salida del comando anterior podría verse como:

```bash
Interesting ports on 192.168.1.100:
(The 65527 ports scanned but not shown below are in state: closed)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 3.5p1 (protocol 1.99)
80/tcp    open  http        Apache httpd 2.0.40 ((Red Hat Linux))
443/tcp   open  ssl         OpenSSL
901/tcp   open  http        Samba SWAT administration server
1241/tcp  open  ssl         Nessus security scanner
3690/tcp  open  unknown
8000/tcp  open  http-alt?
8080/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
```

De este ejemplo, uno puede ver que:

- Hay un servidor HTTP Apache ejecutándose en puerto 80.
- Parece que hay un servidor HTTPS en puerto 443 (pero esto necesita ser confirmado, por ejemplo, visitando `https://192.168.1.100` con un navegador).
- En puerto 901 hay una interfaz web Samba SWAT.
- El servicio en puerto 1241 no es HTTPS, pero es el daemon Nessus envuelto en TLS.
- Puerto 3690 cuenta con un servicio no especificado (Nmap da de vuelta su *fingerprint* - aquí omitido por claridad - junto con instrucciones para enviarlo para incorporación en la base de datos de fingerprints de Nmap, proporcionado que se sepa qué servicio representa).
- Otro servicio no especificado en puerto 8000; esto podría posiblemente ser HTTP, ya que no es inusual encontrar servidores HTTP en este puerto. Examinemos este problema:

```bash
$ telnet 192.168.10.100 8000
Trying 192.168.1.100...
Connected to 192.168.1.100.
Escape character is '^]'.
GET / HTTP/1.0

HTTP/1.0 200 OK
pragma: no-cache
Content-Type: text/html
Server: MX4J-HTTPD/1.0
expires: now
Cache-Control: no-cache

<html>
...
```

Esto confirma que de hecho es un servidor HTTP. Alternativamente, los probadores podrían haber visitado la URL con un navegador web; o usado los comandos GET o HEAD de Perl, que imitan interacciones HTTP como la dada arriba (sin embargo, las solicitudes HEAD pueden no ser honradas por todos los servidores).

- Apache Tomcat ejecutándose en puerto 8080.

La misma tarea puede ser realizada por escáneres de vulnerabilidades, pero primero verifica que el escáner de elección sea capaz de identificar servicios HTTP[S] ejecutándose en puertos no estándar. Por ejemplo, Nessus es capaz de identificarlos en puertos arbitrarios (proporcionado que se le instruya para escanear todos los puertos), y proporcionará, con respecto a Nmap, un número de pruebas en vulnerabilidades conocidas de servidor web, así como en la configuración TLS/SSL de servicios HTTPS. Como se insinuó antes, Nessus también es capaz de detectar aplicaciones populares o interfaces web que de otro modo pasarían desapercibidas (por ejemplo, una interfaz administrativa de Tomcat).

### Enfoques para Abordar el Problema 3 - Hosts Virtuales

Hay un número de técnicas que pueden usarse para identificar nombres DNS asociados a una dirección IP dada `x.y.z.t`.

#### Transferencias de Zona DNS

Esta técnica tiene uso limitado hoy en día, dado el hecho de que las transferencias de zona son ampliamente no honradas por servidores DNS. Sin embargo, aún podría valer la pena intentar. Primero de todo, los probadores deben determinar los servidores de nombres sirviendo `x.y.z.t`. Si un nombre simbólico es conocido para `x.y.z.t` (déjelo ser `www.example.com`), sus servidores de nombres pueden determinarse mediante herramientas como `nslookup`, `host`, o `dig`, solicitando registros DNS NS.

Si no se conocen nombres simbólicos para `x.y.z.t`, pero la definición del objetivo contiene al menos un nombre simbólico, los probadores pueden intentar aplicar el mismo proceso y consultar el servidor de nombres de ese nombre (esperando que `x.y.z.t` sea servido también por ese servidor de nombres). Por ejemplo, si el objetivo consiste en la dirección IP `x.y.z.t` y el nombre `mail.example.com`, determina los servidores de nombres para el dominio `example.com`.

El siguiente ejemplo muestra cómo identificar los servidores de nombres para `www.owasp.org` usando el comando `host`:

```bash
$ host -t ns www.owasp.org
www.owasp.org is an alias for owasp.org.
owasp.org name server ns1.secure.net.
owasp.org name server ns2.secure.net.
```

Una transferencia de zona puede ahora ser solicitada a los servidores de nombres para el dominio `example.com`. Si el probador es afortunado, puede recibir una lista de las entradas DNS para este dominio en respuesta. Esto incluirá el obvio `www.example.com` y el no tan obvio `helpdesk.example.com` y `webmail.example.com` (y posiblemente otros). Verifica todos los nombres retornados por la transferencia de zona y considera todos aquellos que están relacionados con el objetivo siendo evaluado.

Intentando solicitar una transferencia de zona para `owasp.org` desde uno de sus servidores de nombres:

```bash
$ host -l www.owasp.org ns1.secure.net
Using domain server:
Name: ns1.secure.net
Address: 192.220.124.10#53
Aliases:

Host www.owasp.org not found: 5(REFUSED)
; Transfer failed.
```

#### Consultas Inversas DNS

Este proceso es similar al anterior, pero se basa en registros DNS inversos (PTR). En lugar de solicitar una transferencia de zona, intenta establecer el tipo de registro a PTR y emite una consulta en la dirección IP dada. Si los probadores son afortunados, pueden recibir una entrada de nombre DNS en respuesta. Esta técnica se basa en la existencia de mapas IP-a-nombre simbólico, lo cual no está garantizado.

#### Búsquedas DNS Basadas en Web

Este tipo de búsqueda es similar a la transferencia de zona DNS, pero se basa en servicios web que habilitan búsquedas basadas en nombres en DNS. Un servicio tal es el servicio [Netcraft Search DNS](https://searchdns.netcraft.com/?host). El probador puede consultar una lista de nombres pertenecientes a su dominio de elección, como `example.com`. Luego verificarán si los nombres que obtuvieron son pertinentes al objetivo que están examinando.

#### Servicios Reverse-IP

Los servicios Reverse-IP son similares a las consultas DNS inversas, con la diferencia de que los probadores consultan una aplicación web en lugar de un servidor de nombres. Hay un número de tales servicios disponibles. Dado que tienden a retornar resultados parciales (y a menudo diferentes), es mejor usar múltiples servicios para obtener un análisis más comprehensivo.

- [MxToolbox Reverse IP](https://mxtoolbox.com/ReverseLookup.aspx)
- [DNSstuff](https://www.dnsstuff.com/) (múltiples servicios disponibles)
- [Net Square](https://web.archive.org/web/20190515092354/https://www.net-square.com/mspawn.html) (múltiples consultas en dominios y direcciones IP, requiere instalación)

#### Googling

Siguiendo la recopilación de información de las técnicas anteriores, los probadores pueden depender de motores de búsqueda para posiblemente refinar e incrementar su análisis. Esto puede rendir evidencia de nombres simbólicos adicionales pertenecientes al objetivo, o aplicaciones accesibles vía URLs no obvias.

Por instancia, considerando el ejemplo anterior respecto a `www.owasp.org`, el probador podría consultar Google y otros motores de búsqueda buscando información (por lo tanto, nombres DNS) relacionados con los dominios recién descubiertos de `webgoat.org`, `webscarab.com`, y `webscarab.net`.

Las técnicas de Googling se explican en [Testing: Spiders, Robots, and Crawlers](01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md).

#### Certificados Digitales

Si el servidor acepta conexiones sobre HTTPS, entonces el Common Name (CN) y Subject Alternate Name (SAN) en el certificado pueden contener uno o más hostnames. Sin embargo, si el servidor web no tiene un certificado confiable, o se usan wildcards, esto puede no retornar información válida.

El CN y SAN pueden obtenerse inspeccionando manualmente el certificado, o mediante otras herramientas como OpenSSL:

```sh
openssl s_client -connect 93.184.216.34:443 </dev/null 2>/dev/null | openssl x509 -noout -text | grep -E 'DNS:|Subject:'

Subject: C = US, ST = California, L = Los Angeles, O = Internet Corporation for Assigned Names and Numbers, CN = www.example.org
DNS:www.example.org, DNS:example.com, DNS:example.edu, DNS:example.net, DNS:example.org, DNS:www.example.com, DNS:www.example.edu, DNS:www.example.net
```

## Herramientas

- Herramientas de búsqueda DNS como `nslookup`, `dig` y similares.
- Motores de búsqueda (Google, Bing, y otros motores de búsqueda principales).
- Servicio de búsqueda web especializado relacionado con DNS: ver texto.
- [Nmap](https://nmap.org/)
- [Nessus Vulnerability Scanner](https://www.tenable.com/products/nessus)
- [Nikto](https://www.cirt.net/nikto2)