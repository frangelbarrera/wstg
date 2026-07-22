# Pruebas de Contaminación de Parámetros HTTP

|ID          |
|------------|
|WSTG-INPV-04|

## Resumen

Las pruebas de Contaminación de Parámetros HTTP (HTTP Parameter Pollution, HPP) evalúan la respuesta de la aplicación al recibir múltiples parámetros HTTP con el mismo nombre; por ejemplo, si el parámetro `username` se incluye dos veces en los parámetros GET o POST.

Proporcionar múltiples parámetros HTTP con el mismo nombre puede hacer que una aplicación interprete los valores de formas no anticipadas. Al explotar estos efectos, un atacante puede ser capaz de eludir la validación de entrada, provocar errores en la aplicación o modificar los valores de variables internas. Como la HPP afecta un componente básico de todas las tecnologías web, existen ataques tanto del lado del servidor como del cliente.

Los estándares HTTP actuales no incluyen directrices sobre cómo interpretar múltiples parámetros de entrada con el mismo nombre. Por ejemplo, [RFC 3986](https://www.ietf.org/rfc/rfc3986.txt) simplemente define el término *Query String* como una serie de pares campo-valor y [RFC 2396](https://www.ietf.org/rfc/rfc2396.txt) define clases de caracteres de cadena de consulta reservados y no reservados. Sin un estándar establecido, los componentes de aplicaciones web manejan este caso límite de diversas maneras (véase la tabla a continuación para detalles).

Por sí solo, esto no es necesariamente una indicación de vulnerabilidad. Sin embargo, si el desarrollador no es consciente del problema, la presencia de parámetros duplicados puede producir un comportamiento anómalo en la aplicación que puede ser potencialmente explotado por un atacante. Como ocurre a menudo en seguridad, los comportamientos inesperados son una fuente habitual de debilidades que podrían llevar a ataques de HPP en este caso. Para introducir mejor esta clase de vulnerabilidades y el resultado de los ataques HPP, es interesante analizar algunos ejemplos reales que se han descubierto en el pasado.

### Elusión de Validación de Entrada y Filtros

En 2009, inmediatamente después de la publicación de la primera investigación sobre HTTP Parameter Pollution, la técnica recibió atención de la comunidad de seguridad como una posible forma de eludir firewalls de aplicaciones web.

Una de estas fallas, que afectaba a *ModSecurity SQL Injection Core Rules*, representa un ejemplo perfecto de la falta de coincidencia entre aplicaciones y filtros. El filtro ModSecurity aplicaría correctamente una lista de denegación para la siguiente cadena: `select 1,2,3 from table`, bloqueando así esta URL de ejemplo para que no sea procesada por el servidor web: `/index.aspx?page=select 1,2,3 from table`. Sin embargo, al explotar la concatenación de múltiples parámetros HTTP, un atacante podría hacer que el servidor de aplicaciones concatenara la cadena después de que el filtro ModSecurity ya hubiera aceptado la entrada. Como ejemplo, la URL `/index.aspx?page=select 1&page=2,3` from table no activaría el filtro ModSecurity, pero la capa de aplicación concatenaría la entrada de vuelta a la cadena maliciosa completa.

Otra vulnerabilidad HPP resultó afectar a *Apple Cups*, el conocido sistema de impresión utilizado por muchos sistemas Unix. Al explotar HPP, un atacante podría activar fácilmente una vulnerabilidad de Cross-Site Scripting utilizando la siguiente URL: `https://127.0.0.1:631/admin/?kerberos=onmouseover=alert(1)&kerberos`. El punto de control de validación de la aplicación podría eludirse agregando un argumento `kerberos` extra con una cadena válida (por ejemplo, cadena vacía). Como el punto de control de validación solo consideraría la segunda ocurrencia, el primer parámetro `kerberos` no se sanitizó correctamente antes de usarse para generar contenido HTML dinámico. Una explotación exitosa resultaría en la ejecución de código JavaScript bajo el contexto del sitio anfitrión.

### Elusión de Autenticación

Una vulnerabilidad HPP aún más crítica se descubrió en *Blogger*, la popular plataforma de blogs. El error permitía a usuarios maliciosos tomar posesión del blog de la víctima utilizando la siguiente solicitud HTTP (`https://www.blogger.com/add-authors.do`):

```html
POST /add-authors.do HTTP/1.1
[...]

security_token=attackertoken&blogID=attackerblogidvalue&blogID=victimblogidvalue&authorsList=goldshlager19test%40gmail.com(attacker email)&ok=Invite
```

La falla residía en el mecanismo de autenticación utilizado por la aplicación web, ya que la verificación de seguridad se realizaba en el primer parámetro `blogID`, mientras que la operación real utilizaba la segunda ocurrencia.

### Comportamiento Esperado por Servidor de Aplicaciones

La siguiente tabla ilustra cómo diferentes tecnologías web se comportan en presencia de múltiples ocurrencias del mismo parámetro HTTP.

Dada la URL y cadena de consulta: `https://example.com/?color=red&color=blue`

  | Backend del Servidor de Aplicaciones Web | Resultado del Análisis | Ejemplo |
  |--------------------------------|----------------|--------|
  | ASP.NET / IIS | Todas las ocurrencias concatenadas con una coma |  color=red,blue |
  | ASP / IIS     | Todas las ocurrencias concatenadas con una coma | color=red,blue |
  | .NET Core 3.1 / Kestrel | Todas las ocurrencias concatenadas con una coma | color=red,blue |
  | .NET 5 / Kestrel | Todas las ocurrencias concatenadas con una coma | color=red,blue |
  | PHP / Apache  | Solo la última ocurrencia | color=blue |
  | PHP / Zeus | Solo la última ocurrencia | color=blue |
  | JSP, Servlet / Apache Tomcat | Solo la primera ocurrencia | color=red |
  | JSP, Servlet / Oracle Application Server 10g | Solo la primera ocurrencia | color=red |
  | JSP, Servlet / Jetty  | Solo la primera ocurrencia | color=red |
  | IBM Lotus Domino | Solo la última ocurrencia | color=blue |
  | IBM HTTP Server | Solo la primera ocurrencia | color=red |
  | Node.js / express | Solo la primera ocurrencia | color=red |
  | mod_perl, libapreq2 / Apache | Solo la primera ocurrencia | color=red |
  | Perl CGI / Apache | Solo la primera ocurrencia | color=red |
  | mod_wsgi (Python) / Apache | Solo la primera ocurrencia | color=red |
  | Python / Zope | Todas las ocurrencias en tipo de datos Lista | color=['red','blue'] |

(Fuente: Appsec EU 2009 Carettoni & Paola)

## Objetivos de las Pruebas

- Identificar el backend y el método de análisis utilizado.
- Evaluar puntos de inyección e intentar eludir filtros de entrada utilizando HPP.

## Cómo Probar

Afortunadamente, dado que la asignación de parámetros HTTP se maneja típicamente a través del servidor de aplicaciones web, y no del código de la aplicación en sí, probar la respuesta a la contaminación de parámetros debería ser estándar en todas las páginas y acciones. Sin embargo, como se requiere conocimiento profundo de la lógica de negocio, probar HPP requiere pruebas manuales. Las herramientas automáticas solo pueden asistir parcialmente a los auditores ya que tienden a generar demasiados falsos positivos. Además, HPP puede manifestarse en componentes del lado del cliente y del servidor.

### HPP del Lado del Servidor

Para probar vulnerabilidades HPP, identifica cualquier formulario o acción que permita entrada proporcionada por el usuario. Los parámetros de cadena de consulta en solicitudes HTTP GET son fáciles de modificar en la barra de navegación del navegador. Si la acción del formulario envía datos vía POST, el probador necesitará usar un proxy interceptador para manipular los datos POST a medida que se envían al servidor. Habiendo identificado un parámetro de entrada particular para probar, se puede editar los datos GET o POST interceptando la solicitud, o cambiar la cadena de consulta después de que la página de respuesta se cargue. Para probar vulnerabilidades HPP simplemente agrega el mismo parámetro a los datos GET o POST pero con un valor diferente asignado.

Por ejemplo: si se prueba el parámetro `search_string` en la cadena de consulta, la URL de solicitud incluiría ese nombre y valor de parámetro:

```text
https://example.com/?search_string=kittens
```

El parámetro particular podría estar oculto entre varios otros parámetros, pero el enfoque es el mismo; deja los otros parámetros en su lugar y agrega el duplicado:

```text
https://example.com/?mode=guest&search_string=kittens&num_results=100
```

Agrega el mismo parámetro con un valor diferente:

```text
https://example.com/?mode=guest&search_string=kittens&num_results=100&search_string=puppies
```

y envía la nueva solicitud.

Analiza la página de respuesta para determinar qué valor(es) se analizaron. En el ejemplo anterior, los resultados de búsqueda pueden mostrar `kittens`, `puppies`, alguna combinación de ambos (`kittens,puppies` o `kittens~puppies` o `['kittens','puppies']`), pueden dar un resultado vacío o página de error.

Este comportamiento, ya sea usando el primero, último o combinación de parámetros de entrada con el mismo nombre, es muy probable que sea consistente en toda la aplicación. Si este comportamiento predeterminado revela una vulnerabilidad potencial depende de la validación de entrada específica y el filtrado específico de una aplicación particular. Como regla general: si la validación de entrada existente y otros mecanismos de seguridad son suficientes en entradas únicas, y si el servidor asigna solo los primeros o últimos parámetros contaminados, entonces la contaminación de parámetros no revela una vulnerabilidad. Si los parámetros duplicados se concatenan, diferentes componentes de aplicaciones web usan diferentes ocurrencias o las pruebas generan un error, hay una mayor probabilidad de poder usar la contaminación de parámetros para activar vulnerabilidades de seguridad.

Un análisis más profundo requeriría tres solicitudes HTTP para cada parámetro HTTP:

1. Envía una solicitud HTTP que contenga el nombre y valor de parámetro estándar, y registra la respuesta HTTP. Ej. `page?par1=val1`
2. Reemplaza el valor del parámetro con un valor manipulado, envía y registra la respuesta HTTP. Ej. `page?par1=HPP_TEST1`
3. Envía una nueva solicitud combinando el paso (1) y (2). Nuevamente, guarda la respuesta HTTP. Ej. `page?par1=val1&par1=HPP_TEST1`
4. Compara las respuestas obtenidas durante todos los pasos anteriores. Si la respuesta de (3) es diferente de (1) y la respuesta de (3) también es diferente de (2), hay una falta de coincidencia que puede ser eventualmente abusada para activar vulnerabilidades HPP.

Crear un exploit completo a partir de una debilidad de contaminación de parámetros está más allá del alcance de este texto. Véase las referencias para ejemplos y detalles.

### HPP del Lado del Cliente

De manera similar a HPP del lado del servidor, las pruebas manuales son la única técnica confiable para auditar aplicaciones web con el fin de detectar vulnerabilidades de contaminación de parámetros que afectan componentes del lado del cliente. Mientras que en la variante del lado del servidor el atacante aprovecha una aplicación web vulnerable para acceder a datos protegidos o realizar acciones que no están permitidas o no se supone que se ejecuten, los ataques del lado del cliente apuntan a subvertir componentes y tecnologías del lado del cliente.

Para probar vulnerabilidades HPP del lado del cliente, identifica cualquier formulario o acción que permita entrada de usuario y muestre un resultado de esa entrada de vuelta al usuario. Una página de búsqueda es ideal, pero un cuadro de inicio de sesión podría no funcionar (ya que podría no mostrar un nombre de usuario inválido de vuelta al usuario).

De manera similar a HPP del lado del servidor, contamina cada parámetro HTTP con `%26HPP_TEST` y busca ocurrencias *decodificadas de URL* de la carga útil proporcionada por el usuario:

- `&HPP_TEST`
- `&HPP_TEST`
- etc.

En particular, presta atención a respuestas que tengan vectores HPP dentro de atributos `data`, `src`, `href` o acciones de formularios. Nuevamente, si este comportamiento predeterminado revela una vulnerabilidad potencial depende de la validación de entrada específica, el filtrado y la lógica de negocio de la aplicación. Además, es importante notar que esta vulnerabilidad también puede afectar parámetros de cadena de consulta utilizados en XMLHttpRequest (XHR), creación de atributos en tiempo de ejecución y otras tecnologías de plugins (por ejemplo, variables flashvars de Adobe Flash).

## Herramientas

- [Escáneres Pasivos/Activos de ZAP](https://www.zaproxy.org)

## Referencias

### Documentos Técnicos

- [HTTP Parameter Pollution - Luca Carettoni, Stefano di Paola](https://www.acunetix.com/websitesecurity/HTTP-Parameter-Pollution-WhitePaper.pdf)
- [Ejemplo de Contaminación de Parámetros HTTP del Lado del Cliente (Falla de Yahoo! Classic Mail) - Stefano di Paola](https://blog.mindedsecurity.com/2009/05/client-side-http-parameter-pollution.html)
- [Cómo Detectar Ataques de Contaminación de Parámetros HTTP - Chrysostomos Daniel](https://www.acunetix.com/blog/whitepaper-http-parameter-pollution/)
- [CAPEC-460: HTTP Parameter Pollution (HPP) - Evgeny Lebanidze](https://capec.mitre.org/data/definitions/460.html)
- [Descubrimiento Automatizado de Vulnerabilidades de Contaminación de Parámetros en Aplicaciones Web - Marco Balduzzi, Carmen Torrano Gimenez, Davide Balzarotti, Engin Kirda](https://s3.eurecom.fr/docs/ndss11_hpp.pdf)