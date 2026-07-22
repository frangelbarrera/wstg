# Identificación de Superficie de Ataque

|ID          |
|------------|
|WSTG-INFO-04|

## Resumen

Identificar la superficie de ataque de una aplicación web involucra descubrir todas las aplicaciones, dominios, virtual hosts, y servicios expuestos externamente asociados con la infraestructura objetivo. Este proceso se extiende más allá de identificar aplicaciones hospedadas e incluye enumeración DNS, descubrimiento de subdominios, análisis de virtual hosts, puertos no estándar, y la revisión de certificados digitales y logs de Certificate Transparency.

Con la proliferación de virtual hosting e infraestructura compartida, la relación tradicional 1:1 entre una dirección IP y un servidor web ha desaparecido en gran medida. Una sola dirección IP podría hospedar múltiples aplicaciones a través de diferentes dominios, entornos, o interfaces administrativas. Fallar en identificar estos activos puede resultar en evaluaciones incompletas y vulnerabilidades pasadas por alto.

A los profesionales de seguridad a veces se les da un conjunto de direcciones IP como objetivo para probar. Es discutible que este escenario sea más parecido a un engagement de tipo prueba de penetración, pero en cualquier caso se espera que tal asignación pruebe todas las aplicaciones web accesibles a través de este objetivo. El problema es que la dirección IP dada hospeda un servicio HTTP en el puerto 80, pero si un tester debería acceder especificando la dirección IP (que es todo lo que conocen) reporta "No web server configured at this address" o un mensaje similar. Pero ese sistema podría "esconder" un número de aplicaciones web, asociadas a nombres simbólicos (DNS) no relacionados. Obviamente, el alcance del análisis está profundamente afectado dependiendo de si el tester prueba todas las aplicaciones o solo prueba las aplicaciones de las que está al tanto.

A veces, la especificación del objetivo es más rica. Al tester se le podría dar una lista de direcciones IP y sus correspondientes nombres simbólicos. Sin embargo, esta lista podría transmitir información parcial, i.e., podría omitir algunos nombres simbólicos y el cliente podría ni siquiera estar al tanto de eso (esto es más probable que ocurra en grandes organizaciones).

Otros problemas que afectan el alcance de la evaluación están representados por aplicaciones web publicadas en URLs no obvias (por ejemplo, `https://www.example.com/some-strange-URL`), que no están referenciadas en otro lugar. Esto podría ocurrir por error (debido a malas configuraciones), o intencionalmente (por ejemplo, interfaces administrativas no anunciadas).

Para abordar estos problemas, debe realizarse un proceso integral de identificación de superficie de ataque.

## Objetivos de Prueba

- Enumerar todas las aplicaciones web dentro del alcance.
- Identificar nombres DNS, dominios, y virtual hosts asociados con el objetivo.
- Descubrir dominios y subdominios adicionales usando técnicas DNS pasivas y activas.
- Analizar certificados digitales y logs de Certificate Transparency para hostnames adicionales.

## Cómo Probar

El descubrimiento de aplicaciones web es un proceso que apunta a identificar aplicaciones web en una infraestructura dada. Esta última usualmente se especifica como un conjunto de direcciones IP (quizás un bloque de red), pero podría consistir en un conjunto de nombres simbólicos DNS o una mezcla de ambos. Esta información se entrega antes de la ejecución de una evaluación, sea una prueba de penetración de estilo clásico o una evaluación enfocada en aplicación. En ambos casos, a menos que las reglas de engagement especifiquen lo contrario (por ejemplo, probar solo la aplicación ubicada en la URL `https://www.example.com/`), la evaluación debería esforzarse por ser la más completa en alcance, i.e. debería identificar todas las aplicaciones accesibles a través del objetivo dado. Los siguientes ejemplos examinan algunas técnicas que pueden emplearse para lograr este objetivo.

> Algunas de las siguientes técnicas aplican a servidores web expuestos a Internet, a saber DNS y servicios de búsqueda web reverse-IP y el uso de motores de búsqueda. Los ejemplos hacen uso de direcciones IP privadas (tales como `192.168.1.100`), las cuales, a menos que se indique lo contrario, representan direcciones IP *genéricas* y se usan solo para propósitos de anonimato.

Hay tres factores que influyen en cuántas aplicaciones están relacionadas con un nombre DNS dado (o una dirección IP):

1. **URL Base Diferente**

    El punto de entrada obvio para una aplicación web es `www.example.com`, i.e., con esta notación abreviada pensamos en la aplicación web originándose en `https://www.example.com/` (lo mismo aplica para HTTPS). Sin embargo, aunque esta es la situación más común, no hay nada forzando a la aplicación a comenzar en `/`.

    Por ejemplo, el mismo nombre simbólico podría estar asociado a tres aplicaciones web tales como: `https://www.example.com/app1` `https://www.example.com/app2` `https://www.example.com/app3`

    En este caso, la URL `https://www.example.com/` no estaría asociada con una página significativa. Las tres aplicaciones permanecerían **ocultas** a menos que el tester sepa explícitamente cómo accederlas, i.e., el tester conoce *app1*, *app2* o *app3*. Usualmente no hay necesidad de publicar aplicaciones web de esta manera, a menos que el propietario no quiera que sean accesibles de manera estándar, y esté preparado para informar a los usuarios sobre su ubicación exacta. Esto no significa que estas aplicaciones sean secretas, solo que su existencia y ubicación no están explícitamente anunciadas.

2. **Puertos No Estándar**

    Mientras que las aplicaciones web usualmente viven en el puerto 80 (HTTP) y 443 (HTTPS), no hay nada fijo o mandatorio sobre estos números de puerto. De hecho, las aplicaciones web podrían estar asociadas con puertos TCP arbitrarios, y pueden ser referenciadas especificando el número de puerto de la siguiente manera: `http[s]://www.example.com:port/`. Por ejemplo, `https://www.example.com:20000/`.

3. **Virtual Hosts**

    DNS permite que una sola dirección IP se asocie con uno o más nombres simbólicos. Por ejemplo, la dirección IP `192.168.1.100` podría estar asociada a nombres DNS `www.example.com`, `helpdesk.example.com`, `webmail.example.com`. No es necesario que todos los nombres pertenezcan al mismo dominio DNS. Esta relación 1-a-N podría reflejarse para servir contenido diferente usando los llamados virtual hosts. La información que especifica el virtual host al que nos referimos está embebida en el [encabezado Host](https://datatracker.ietf.org/doc/html/rfc7230#section-5.4) HTTP 1.1.

    Uno no sospecharía la existencia de otras aplicaciones web además del obvio `www.example.com`, a menos que conozca `helpdesk.example.com` y `webmail.example.com`.

### Enfoques para Abordar el Problema 1 - URLs No Estándar

No hay manera de ascertainar completamente la existencia de aplicaciones web con nombres no estándar. Siendo no estándar, no hay criterios fijos gobernando la convención de nomenclatura, sin embargo hay varias técnicas que el tester puede usar para ganar algo de entendimiento adicional.

Primero, si el servidor web está mal configurado y permite directory browsing, podría ser posible detectar estas aplicaciones. Los escáneres de vulnerabilidades podrían ayudar en este respecto.

Segundo, estas aplicaciones podrían estar referenciadas por otras páginas web y hay una posibilidad de que hayan sido rastreadas e indexadas por motores de búsqueda web. Si los testers sospechan la existencia de tales aplicaciones **ocultas** en `www.example.com` podrían buscar usando el operador *site* y examinar el resultado de una consulta por `site: www.example.com`. Entre las URLs devueltas podría haber una apuntando a tal aplicación no obvia.

Otra opción es probar URLs que podrían ser candidatos probables para aplicaciones no publicadas. Por ejemplo, un frontend de web mail podría ser accesible desde URLs tales como `https://www.example.com/webmail`, `https://webmail.example.com/`, o `https://mail.example.com/`. Lo mismo aplica para interfaces administrativas, que podrían estar publicadas en URLs ocultas (por ejemplo, una interfaz administrativa de Tomcat), y sin embargo no referenciadas en ningún lugar. Así que hacer un poco de búsqueda estilo diccionario (o "adivinación inteligente") podría producir algunos resultados. Los escáneres de vulnerabilidades podrían ayudar en este respecto.

### Enfoques para Abordar el Problema 2 - Puertos No Estándar

Es fácil verificar la existencia de aplicaciones web en puertos no estándar. Un escáner de puertos tal como Nmap es capaz de realizar reconocimiento de servicios por medio de la opción `-sV`, e identificará servicios http[s] en puertos arbitrarios. Lo que se requiere es un escaneo completo de todo el espacio de direcciones de puertos TCP de 64k.

Por ejemplo, el siguiente comando buscará, con un escaneo TCP connect, todos los puertos abiertos en la IP `192.168.1.100` e intentará determinar qué servicios están vinculados a ellos (solo se muestran los switches *esenciales* – Nmap presenta un amplio conjunto de opciones, cuya discusión está fuera del alcance):

`nmap –Pn –sT –sV –p0-65535 192.168.1.100`

Es suficiente examinar la salida y buscar HTTP o la indicación de servicios envueltos en TLS (que deberían probarse para confirmar que son HTTPS). Por ejemplo, la salida del comando anterior podría verse como:

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

De este ejemplo, se puede ver que:

- Hay un servidor Apache HTTP ejecutándose en el puerto 80.
- Parece que hay un servidor HTTPS en el puerto 443 (pero esto necesita confirmarse, por ejemplo, visitando `https://192.168.1.100` con un navegador).
- En el puerto 901 hay una interfaz web de Samba SWAT.
- El servicio en el puerto 1241 no es HTTPS, sino el daemon Nessus envuelto en TLS.
- El puerto 3690 presenta un servicio no especificado (Nmap devuelve su *fingerprint* - aquí omitido por claridad - junto con instrucciones para enviarlo para incorporación en la base de datos de fingerprints de Nmap, siempre que se sepa qué servicio representa).
- Otro servicio no especificado en el puerto 8000; esto podría posiblemente ser HTTP, ya que no es raro encontrar servidores HTTP en este puerto. Examinemos este problema:

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

Esto confirma que de hecho es un servidor HTTP. Alternativamente, los testers podrían haber visitado la URL con un navegador web; o usado los comandos Perl GET o HEAD, que imitan interacciones HTTP tales como la dada arriba (sin embargo las solicitudes HEAD podrían no ser honradas por todos los servidores).

- Apache Tomcat ejecutándose en el puerto 8080.

La misma tarea podría ser realizada por escáneres de vulnerabilidades, pero primero verificar que el escáner de elección sea capaz de identificar servicios HTTP[S] ejecutándose en puertos no estándar. Por ejemplo, Nessus es capaz de identificarlos en puertos arbitrarios (siempre que se le instruya escanear todos los puertos), y proporcionará, con respecto a Nmap, varias pruebas en vulnerabilidades conocidas de servidor web, así como en la configuración TLS/SSL de servicios HTTPS. Como se insinuó antes, Nessus también es capaz de detectar aplicaciones populares o interfaces web que de otra manera podrían pasar desapercibidas (por ejemplo, una interfaz administrativa de Tomcat).

### Enfoques para Abordar el Problema 3 - Virtual Hosts

Hay varias técnicas que pueden usarse para identificar nombres DNS asociados a una dirección IP dada `x.y.z.t`.

#### Enumeración DNS

La enumeración DNS apunta a identificar dominios, subdominios, y registros DNS relacionados asociados con la organización objetivo para expandir el alcance de la evaluación. La enumeración DNS juega un rol clave en identificar virtual hosts adicionales mapeados a la misma dirección IP. Esto podría revelar sistemas de desarrollo, entornos de staging, servicios heredados, o interfaces administrativas.

Tanto técnicas pasivas como activas pueden usarse.

#### Enumeración DNS Pasiva

Las técnicas pasivas no interactúan directamente con la infraestructura objetivo y en su lugar dependen de fuentes de datos públicamente disponibles. Los ejemplos incluyen:

- Registros DNS públicos (A, AAAA, MX, TXT, NS)
- Lookups DNS inversos (registros PTR)
- Motores de búsqueda
- Bases de datos DNS pasivas
- Logs de Certificate Transparency

Las técnicas pasivas se prefieren durante fases tempranas de reconocimiento para evitar detección.

#### Enumeración DNS Activa

Las técnicas activas consultan directamente la infraestructura DNS del objetivo y podrían generar logs en los sistemas objetivo. Estas incluyen:

- Fuerza bruta de subdominios
- Intentos de transferencia de zona DNS
- Enumeración de registros DNS usando herramientas

Herramientas comunes usadas para enumeración DNS incluyen:

- `amass`
- `subfinder`
- `dnsrecon`
- `fierce`
- `dig`
- `nslookup`

Ejemplo usando `dig`: `dig example.com ANY`

#### Transferencias de Zona DNS

Esta técnica tiene uso limitado hoy en día, dado el hecho de que las transferencias de zona no son honradas en gran medida por los servidores DNS. Sin embargo, todavía podría valer la pena intentarlo. Primero que todo, los testers deben determinar los servidores de nombres que sirven `x.y.z.t`. Si se conoce un nombre simbólico para `x.y.z.t` (que sea `www.example.com`), sus servidores de nombres pueden determinarse por medio de herramientas tales como `nslookup`, `host`, o `dig`, solicitando registros NS de DNS.

Si no se conocen nombres simbólicos para `x.y.z.t`, pero la definición del objetivo contiene al menos un nombre simbólico, los testers podrían intentar aplicar el mismo proceso y consultar el servidor de nombres de ese nombre (esperando que `x.y.z.t` también sea servido por ese servidor de nombres). Por ejemplo, si el objetivo consiste en la dirección IP `x.y.z.t` y el nombre `mail.example.com`, determinar los servidores de nombres para el dominio `example.com`.

El siguiente ejemplo muestra cómo identificar los servidores de nombres para `www.owasp.org` usando el comando `host`:

```bash
$ host -t ns www.owasp.org
www.owasp.org is an alias for owasp.org.
owasp.org name server ns1.secure.net.
owasp.org name server ns2.secure.net.
```

Una transferencia de zona puede ahora solicitarse a los servidores de nombres para el dominio `example.com`. Si el tester es afortunado, podría recibir una lista de las entradas DNS para este dominio en respuesta. Esto incluirá el obvio `www.example.com` y el no tan obvio `helpdesk.example.com` y `webmail.example.com` (y posiblemente otros). Verificar todos los nombres devueltos por la transferencia de zona y considerar todos aquellos que estén relacionados con el objetivo siendo evaluado.

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

#### Consultas DNS Inversas

Este proceso es similar al anterior, pero depende de registros DNS inversos (PTR). En lugar de solicitar una transferencia de zona, intentar establecer el tipo de registro a PTR y emitir una consulta en la dirección IP dada. Si los testers son afortunados, podrían recibir una entrada de nombre DNS en respuesta. Esta técnica depende de la existencia de mapas de IP a nombre simbólico, lo cual no está garantizado.

#### Búsquedas DNS Basadas en Web

Este tipo de búsqueda es similar a la transferencia de zona DNS, pero depende de servicios basados en web que habilitan búsquedas basadas en nombre en DNS. Uno de tales servicios es el servicio [Netcraft Search DNS](https://searchdns.netcraft.com/?host). El tester podría consultar por una lista de nombres pertenecientes a su dominio de elección, tal como `example.com`. Luego verificarán si los nombres que obtuvieron son pertinentes al objetivo que están examinando.

#### Servicios Reverse-IP

Los servicios Reverse-IP son similares a las consultas DNS inversas, con la diferencia de que los testers consultan una aplicación basada en web en lugar de un servidor de nombres. Hay varios de tales servicios disponibles. Ya que tienden a devolver resultados parciales (y a menudo diferentes), es mejor usar múltiples servicios para obtener un análisis más completo.

- [MxToolbox Reverse IP](https://mxtoolbox.com/ReverseLookup.aspx)
- [DNSstuff](https://www.dnsstuff.com/) (múltiples servicios disponibles)
- [Net Square](https://web.archive.org/web/20190515092354/https://www.net-square.com/mspawn.html) (múltiples consultas en dominios y direcciones IP, requiere instalación)

#### Googling

Siguiendo la recolección de información de las técnicas anteriores, los testers pueden confiar en motores de búsqueda para posiblemente refinar e incrementar su análisis. Esto podría producir evidencia de nombres simbólicos adicionales pertenecientes al objetivo, o aplicaciones accesibles vía URLs no obvias.

Por ejemplo, considerando el ejemplo anterior respecto a `www.owasp.org`, el tester podría consultar Google y otros motores de búsqueda buscando información (por lo tanto, nombres DNS) relacionada con los dominios recién descubiertos de `webgoat.org`, `webscarab.com`, y `webscarab.net`.

Las técnicas de Googling se explican en [Pruebas: Spiders, Robots, y Crawlers](01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md).

#### Certificados Digitales

Si el servidor acepta conexiones sobre HTTPS, entonces el Common Name (CN) y Subject Alternate Name (SAN) en el certificado podrían contener uno o más hostnames. Sin embargo, si el servidor web no tiene un certificado confiable, o se usan wildcards, esto podría no devolver ninguna información válida.

El CN y SAN pueden obtenerse inspeccionando manualmente el certificado, o a través de otras herramientas tales como OpenSSL:

```sh
openssl s_client -connect 93.184.216.34:443 </dev/null 2>/dev/null | openssl x509 -noout -text | grep -E 'DNS:|Subject:'

Subject: C = US, ST = California, L = Los Angeles, O = Internet Corporation for Assigned Names and Numbers, CN = www.example.org
DNS:www.example.org, DNS:example.com, DNS:example.edu, DNS:example.net, DNS:example.org, DNS:www.example.com, DNS:www.example.edu, DNS:www.example.net
```

#### Logs de Certificate Transparency

Los logs de Certificate Transparency (CT) son registros públicamente accesibles de certificados TLS emitidos. Estos logs pueden buscarse para identificar hostnames y subdominios asociados con una organización objetivo, incluyendo sistemas de staging, interfaces administrativas, sistemas heredados, u otros servicios externamente alcanzables.

Revisar los logs de CT podría revelar hostnames que no son directamente descubribles a través de transferencias de zona DNS, lookups inversos, o consultas de motores de búsqueda solos. Los testers deberían extraer los hostnames descubiertos y validarlos a través de resolución DNS para determinar si están activos y dentro del alcance definido de la evaluación.

Al revisar datos de logs de CT, considerar:

- Hostnames que indican entornos de desarrollo, staging, o pruebas.
- Interfaces administrativas o de gestión.
- Sistemas obsoletos o heredados que podrían aún ser accesibles.
- Certificados wildcard que podrían implicar subdominios adicionales no descubiertos.

La información recolectada de logs de CT debería validarse para confirmar propiedad y relevancia antes de pruebas adicionales.

Se debe tener cuidado de respetar las limitaciones de alcance definidas en el engagement.

Los activos descubiertos deberían validarse y documentarse antes de actividades de prueba adicionales.

Un enfoque común para consultar logs de CT es usar portales de búsqueda públicamente disponibles que agregan datos de certificados. Por ejemplo, un tester podría buscar certificados emitidos a `example.com` y revisar los subdominios listados.

Por ejemplo: `https://crt.sh/?q=%25.example.com`

![Ejemplo de Búsqueda en Log de CT](images/Figure-4.1.4-CT-logs-example.png)

*Figura 4.1.4-1: Ejemplo de resultados de búsqueda en log de Certificate Transparency.*

Los resultados podrían listar subdominios tales como `dev.example.com`, `staging.example.com`, u otros hostnames que no están directamente referenciados desde el sitio primario. Los hostnames descubiertos deberían validarse a través de resolución DNS antes de pruebas adicionales.

## Herramientas

- Herramientas de lookup DNS tales como `nslookup`, `dig`, y `host`
- Herramientas de enumeración de subdominios tales como `amass`, `subfinder`, `dnsrecon`, y `fierce`
- Motores de búsqueda (Google, Bing, y otros motores de búsqueda principales)
- Servicios de lookup Reverse IP
- [Nmap](https://nmap.org/)
- [Nessus Vulnerability Scanner](https://www.tenable.com/products/nessus)
- [Nikto](https://github.com/sullo/nikto)
