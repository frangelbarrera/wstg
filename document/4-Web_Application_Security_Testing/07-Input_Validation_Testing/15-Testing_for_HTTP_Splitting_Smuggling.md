# Pruebas de división y contrabando HTTP

|ID          |
|------------|
|WSTG-INPV-15|

## Resumen

Esta sección ilustra ejemplos de ataques que aprovechan características específicas del protocolo HTTP, ya sea explotando debilidades de la aplicación web o peculiaridades en la forma en que diferentes agentes interpretan los mensajes HTTP.
Esta sección analizará dos ataques diferentes que apuntan a encabezados HTTP específicos:

- División HTTP
- Contrabando HTTP

El primer ataque explota la falta de sanitización de entrada que permite a un intruso insertar caracteres CR y LF en los encabezados de la respuesta de la aplicación y 'dividir' esa respuesta en dos mensajes HTTP diferentes. El objetivo del ataque puede variar desde un envenenamiento de caché hasta scripting entre sitios (XSS).

En el segundo ataque, el atacante explota el hecho de que algunos mensajes HTTP especialmente elaborados pueden analizarse e interpretarse de diferentes maneras dependiendo del agente que los reciba. El contrabando HTTP requiere cierto nivel de conocimiento sobre los diferentes agentes que manejan los mensajes HTTP (servidor web, proxy, firewall) y, por lo tanto, se incluirá solo en la sección de pruebas de caja gris.

## Objetivos de la prueba

- Evaluar si la aplicación es vulnerable a la división, identificando qué ataques posibles son alcanzables.
- Evaluar si la cadena de comunicación es vulnerable al contrabando, identificando qué ataques posibles son alcanzables.

## Cómo probar

### Pruebas de caja negra

#### División HTTP

Algunas aplicaciones web usan parte de la entrada del usuario para generar los valores de algunos encabezados de sus respuestas. El ejemplo más directo lo proporcionan las redirecciones en las que la URL de destino depende de algún valor enviado por el usuario. Digamos, por ejemplo, que se le pide al usuario que elija si prefiere una interfaz web estándar o avanzada. La elección se pasará como un parámetro que se usará en el encabezado de respuesta para activar la redirección a la página correspondiente.

Más específicamente, si el parámetro 'interface' tiene el valor 'advanced', la aplicación responderá con lo siguiente:

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: https://victim.com/main.jsp?interface=advanced
<snip>
```

Al recibir este mensaje, el navegador llevará al usuario a la página indicada en el encabezado Location. Sin embargo, si la aplicación no filtra la entrada del usuario, será posible insertar en el parámetro 'interface' la secuencia %0d%0a, que representa la secuencia CRLF que se usa para separar diferentes líneas. En este punto, los evaluadores podrán activar una respuesta que será interpretada como dos respuestas diferentes por cualquiera que la analice, por ejemplo, una caché web entre nosotros y la aplicación. Esto puede ser aprovechado por un atacante para envenenar esta caché web para que proporcione contenido falso en todas las solicitudes posteriores.

Digamos que en el ejemplo anterior el evaluador pasa los siguientes datos como el parámetro interface:

`advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a%0d%0a<html>Sorry,%20System%20Down</html>`

La respuesta resultante de la aplicación vulnerable será, por lo tanto, la siguiente:

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: https://victim.com/main.jsp?interface=advanced
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 35

<html>Sorry,%20System%20Down</html>
<other data>
```

La caché web verá dos respuestas diferentes, por lo que si el atacante envía, inmediatamente después de la primera solicitud, una segunda pidiendo `/index.html`, la caché web coincidirá con esta solicitud con la segunda respuesta y almacenará en caché su contenido, de modo que todas las solicitudes posteriores dirigidas a `victim.com/index.html` que pasen por esa caché web recibirán el mensaje "system down". De esta manera, un atacante podría deface efectivamente el sitio para todos los usuarios que usen esa caché web (todo Internet, si la caché web es un proxy inverso para la aplicación web).

Alternativamente, el atacante podría pasar a esos usuarios un fragmento de JavaScript que monte un ataque de scripting entre sitios (XSS), por ejemplo, para robar las cookies. Tenga en cuenta que aunque la vulnerabilidad está en la aplicación, el objetivo aquí son sus usuarios. Por lo tanto, para buscar esta vulnerabilidad, el evaluador necesita identificar toda entrada controlada por el usuario que influya en uno o más encabezados en la respuesta, y verificar si pueden inyectar exitosamente una secuencia CR+LF en ella.

Los encabezados que son los candidatos más probables para este ataque son:

- `Location`
- `Set-Cookie`

Debe notarse que una explotación exitosa de esta vulnerabilidad en un escenario del mundo real puede ser bastante compleja, ya que varios factores deben tenerse en cuenta:

1. El pentester debe configurar correctamente los encabezados en la respuesta falsa para que se almacene en caché exitosamente (por ejemplo, un encabezado Last-Modified con una fecha establecida en el futuro). También podrían tener que destruir versiones previamente almacenadas en caché de las páginas de destino, emitiendo una solicitud preliminar con `Pragma: no-cache` en los encabezados de solicitud
2. La aplicación, aunque no filtra la secuencia CR+LF, podría filtrar otros caracteres necesarios para un ataque exitoso (por ejemplo, `<` y `>`). En este caso, el evaluador puede intentar usar otras codificaciones (por ejemplo, UTF-7)
3. Algunos objetivos (por ejemplo, ASP) codificarán en URL la parte de ruta del encabezado Location (por ejemplo, `www.victim.com/redirect.asp`), haciendo inútil una secuencia CRLF. Sin embargo, fallan en codificar la sección de consulta (por ejemplo, ?interface=advanced), lo que significa que un signo de interrogación inicial es suficiente para eludir este filtrado

Para una discusión más detallada sobre este ataque y otra información sobre posibles escenarios y aplicaciones, consulte los documentos referenciados al final de esta sección.

### Pruebas de caja gris

#### División HTTP

Una explotación exitosa de la división HTTP se ayuda enormemente conociendo algunos detalles de la aplicación web y del objetivo del ataque. Por ejemplo, diferentes objetivos pueden usar diferentes métodos para decidir cuándo termina el primer mensaje HTTP y cuándo comienza el segundo. Algunos usarán los límites del mensaje, como en el ejemplo anterior. Otros asumirán que diferentes mensajes serán transportados por diferentes paquetes. Otros asignarán para cada mensaje un número de fragmentos de longitud predeterminada: en este caso, el segundo mensaje tendrá que comenzar exactamente al inicio de un fragmento y esto requerirá que el evaluador use relleno entre los dos mensajes. Esto podría causar algunos problemas cuando el parámetro vulnerable deba enviarse en la URL, ya que una URL muy larga es probable que se trunque o filtre. Un escenario de caja gris puede ayudar al atacante a encontrar una solución alternativa: varios servidores de aplicaciones, por ejemplo, permitirán que la solicitud se envíe usando POST en lugar de GET.

#### Contrabando HTTP

Como se mencionó en la introducción, el contrabando HTTP aprovecha las diferentes formas en que un mensaje HTTP particularmente elaborado puede analizarse e interpretarse por diferentes agentes (navegadores, cachés web, firewalls de aplicaciones). Este tipo relativamente nuevo de ataque fue descubierto por primera vez por Chaim Linhart, Amit Klein, Ronen Heled y Steve Orrin en 2005. Hay varias aplicaciones posibles y analizaremos una de las más espectaculares: el bypass de un firewall de aplicaciones. Consulte el documento original (enlace al final de esta página) para información más detallada y otros escenarios.

##### Bypass de firewall de aplicaciones

Hay varios productos que permiten a un administrador de sistema detectar y bloquear una solicitud web hostil dependiendo de algún patrón malicioso conocido que esté incrustado en la solicitud. Por ejemplo, considere el infame, antiguo [ataque de traversal de directorio Unicode contra servidor IIS](https://www.securityfocus.com/bid/1806), en el que un atacante podría salir del raíz www emitiendo una solicitud como:

`https://target/scripts/..%c1%1c../winnt/system32/cmd.exe?/c+<command_to_execute>`

Por supuesto, es bastante fácil detectar y filtrar este ataque por la presencia de cadenas como ".." y "cmd.exe" en la URL. Sin embargo, IIS 5.0 es bastante exigente con las solicitudes POST cuyo cuerpo es de hasta 48K bytes y trunca todo el contenido que está más allá de este límite cuando el encabezado Content-Type es diferente de application/x-www-form-urlencoded. El pentester puede aprovechar esto creando una solicitud muy grande, estructurada de la siguiente manera:

```html
POST /target.asp HTTP/1.1        <-- Request #1
Host: target
Connection: Keep-Alive
Content-Length: 49225
<CRLF>
<49152 bytes of garbage>
```

```html
POST /target.asp HTTP/1.0        <-- Request #2
Connection: Keep-Alive
Content-Length: 33
<CRLF>
```

```html
POST /target.asp HTTP/1.0        <-- Request #3
xxxx: POST /scripts/..%c1%1c../winnt/system32/cmd.exe?/c+dir HTTP/1.0   <-- Request #4
Connection: Keep-Alive
<CRLF>
```

Lo que sucede aquí es que la `Request #1` está compuesta de 49223 bytes, que incluye también las líneas de `Request #2`. Por lo tanto, un firewall (o cualquier otro agente además de IIS 5.0) verá Request #1, fallará en ver `Request #2` (sus datos serán solo parte de #1), verá `Request #3` y perderá `Request #4` (porque el POST será solo parte del encabezado falso xxxx).

Ahora, ¿qué le sucede a IIS 5.0? Detendrá el análisis de `Request #1` justo después de los 49152 bytes de basura (ya que habrá alcanzado el límite de 48K=49152 bytes) y, por lo tanto, analizará `Request #2` como una nueva solicitud separada. `Request #2` afirma que su contenido es de 33 bytes, que incluye todo hasta "xxxx: ", haciendo que IIS pierda `Request #3` (interpretado como parte de `Request #2`) pero detecte `Request #4`, ya que su POST comienza justo después del byte 33 de `Request #2`. Es un poco complicado, pero el punto es que la URL de ataque no será detectada por el firewall (se interpretará como el cuerpo de una solicitud anterior) pero será analizada correctamente (y ejecutada) por IIS.

Mientras que en el caso mencionado anteriormente la técnica explota un error de un servidor web, hay otros escenarios en los que podemos aprovechar las diferentes formas en que diferentes dispositivos habilitados para HTTP analizan mensajes que no son 100% RFC compliant. Por ejemplo, el protocolo HTTP permite solo un encabezado Content-Length, pero no especifica cómo manejar un mensaje que tiene dos instancias de este encabezado. Algunas implementaciones usarán la primera mientras que otras preferirán la segunda, limpiando el camino para ataques de contrabando HTTP. Otro ejemplo es el uso del encabezado Content-Length en un mensaje GET.

Tenga en cuenta que el contrabando HTTP `*no*` explota ninguna vulnerabilidad en la aplicación web de destino. Por lo tanto, podría ser algo complicado, en un compromiso de pentest, convencer al cliente de que se busque una contramedida de todos modos.

## Referencias

### Documentos técnicos

- [Amit Klein, "Divide and Conquer: HTTP Response Splitting, Web Cache Poisoning Attacks, and Related Topics"](https://packetstormsecurity.com/files/32815/Divide-and-Conquer-HTTP-Response-Splitting-Whitepaper.html)
- [Amit Klein: "HTTP Message Splitting, Smuggling and Other Animals"](https://www.slideserve.com/alicia/http-message-splitting-smuggling-and-other-animals-powerpoint-ppt-presentation)
- [Amit Klein: "HTTP Request Smuggling - ERRATA (the IIS 48K buffer phenomenon)"](https://www.securityfocus.com/archive/1/411418)
- [Amit Klein: "HTTP Response Smuggling"](https://www.securityfocus.com/archive/1/425593)
- [Chaim Linhart, Amit Klein, Ronen Heled, Steve Orrin: "HTTP Request Smuggling"](https://www.cgisecurity.com/lib/http-request-smuggling.pdf)