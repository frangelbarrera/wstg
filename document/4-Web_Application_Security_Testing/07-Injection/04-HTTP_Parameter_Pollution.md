# Contaminación de Parámetros HTTP

|ID          |
|------------|
|WSTG-INJT-04|

## Resumen

La contaminación de parámetros HTTP evalúa la respuesta de las aplicaciones al recibir múltiples parámetros HTTP con el mismo nombre; por ejemplo, si el parámetro `username` se incluye dos veces en los parámetros GET o POST.

Proporcionar múltiples parámetros HTTP con el mismo nombre puede hacer que una aplicación interprete los valores de formas no previstas. Al explotar estos efectos, un atacante podría eludir la validación de entrada, provocar errores en la aplicación o modificar los valores de variables internas. Como la contaminación de parámetros HTTP (en adelante, *HPP*) afecta a un componente básico de todas las tecnologías web, existen ataques tanto del lado del servidor como del lado del cliente.

Los estándares HTTP actuales no incluyen directrices sobre cómo interpretar múltiples parámetros de entrada con el mismo nombre. Por ejemplo, [RFC 3986](https://www.ietf.org/rfc/rfc3986.txt) simplemente define el término *Query String* como una serie de pares de campo y valor, y [RFC 2396](https://www.ietf.org/rfc/rfc2396.txt) define clases de caracteres de cadena de consulta reservados y no reservados. Sin un estándar establecido, los componentes de las aplicaciones web manejan este caso límite de diversas maneras (véase la tabla siguiente para más detalles).

Por sí sola, esta situación no indica necesariamente que haya una vulnerabilidad. Sin embargo, si el desarrollador desconoce el problema, la presencia de parámetros duplicados puede producir un comportamiento anómalo en la aplicación que un atacante podría explotar. Como ocurre a menudo en seguridad, los comportamientos inesperados son una fuente habitual de debilidades que, en este caso, podrían dar lugar a ataques de HPP. Para presentar mejor esta clase de vulnerabilidades y los resultados de los ataques HPP, resulta interesante analizar algunos ejemplos reales descubiertos en el pasado.

### Elusión de la validación de entrada y de filtros

En 2009, inmediatamente después de la publicación de la primera investigación sobre contaminación de parámetros HTTP, la técnica recibió atención de la comunidad de seguridad como una posible forma de eludir los firewalls de aplicaciones web.

Una de estas fallas, que afectaba a *ModSecurity SQL Injection Core Rules*, representa un ejemplo perfecto de la falta de correspondencia entre aplicaciones y filtros. El filtro ModSecurity aplicaría correctamente una lista de denegación a la siguiente cadena: `select 1,2,3 from table`, bloqueando así el procesamiento por parte del servidor web de esta URL de ejemplo: `/index.aspx?page=select 1,2,3 from table`. Sin embargo, al explotar la concatenación de múltiples parámetros HTTP, un atacante podría hacer que el servidor de aplicaciones concatenara la cadena después de que el filtro ModSecurity ya hubiera aceptado la entrada. Por ejemplo, la URL `/index.aspx?page=select 1&page=2,3` from table no activaría el filtro ModSecurity, pero la capa de aplicación volvería a concatenar la entrada para formar la cadena maliciosa completa.

Otra vulnerabilidad HPP afectaba a *Apple Cups*, el conocido sistema de impresión utilizado por muchos sistemas Unix. Al explotar HPP, un atacante podría activar fácilmente una vulnerabilidad de Cross-Site Scripting utilizando la siguiente URL: `https://127.0.0.1:631/admin/?kerberos=onmouseover=alert(1)&kerberos`. El punto de control de validación de la aplicación podía eludirse agregando un argumento `kerberos` adicional con una cadena válida (por ejemplo, una cadena vacía). Como el punto de control de validación solo consideraba la segunda ocurrencia, el primer parámetro `kerberos` no se saneaba correctamente antes de utilizarse para generar contenido HTML dinámico. Si la explotación tenía éxito, se ejecutaba código JavaScript en el contexto del sitio anfitrión.

### Elusión de la autenticación

En *Blogger*, la popular plataforma de blogs, se descubrió una vulnerabilidad HPP aún más crítica. El error permitía a usuarios maliciosos tomar posesión del blog de la víctima mediante la siguiente solicitud HTTP (`https://www.blogger.com/add-authors.do`):

```html
POST /add-authors.do HTTP/1.1
[...]

security_token=attackertoken&blogID=attackerblogidvalue&blogID=victimblogidvalue&authorsList=goldshlager19test%40gmail.com(attacker email)&ok=Invite
```

La falla residía en el mecanismo de autenticación utilizado por la aplicación web, ya que la comprobación de seguridad se realizaba en el primer parámetro `blogID`, mientras que la operación real utilizaba la segunda ocurrencia.

### Comportamiento esperado según el servidor de aplicaciones

La forma de manejar los parámetros duplicados depende del framework, del servidor y de cómo la aplicación lee el valor (por ejemplo, si toma el primer valor, el último o la colección completa). Considere cualquier tabla de valores predeterminados solo como punto de partida: verifique el comportamiento real de la pila de destino, incluidos los proxies inversos, los firewalls de aplicaciones web (WAF) y las puertas de enlace de API, que pueden normalizar las cadenas de consulta de distintas maneras.

Dada la URL y la cadena de consulta: `https://example.com/?color=red&color=blue`

| Pila típica | Análisis predeterminado habitual | Ejemplo |
|---------------|------------------------|---------|
| ASP.NET / IIS | Todas las ocurrencias concatenadas con una coma | `color=red,blue` |
| ASP.NET Core / Kestrel | Todas las ocurrencias disponibles (`StringValues`; concatenadas con comas al convertirlas en cadena) | `color=red,blue` |
| PHP (Apache / php-fpm / Nginx) | Solo la última ocurrencia | `color=blue` |
| Java Servlet (Tomcat, Jetty y similares) | Solo la primera ocurrencia | `color=red` |
| Node.js / Express | Solo la primera ocurrencia | `color=red` |
| Python (Django, Flask y similares) | Todas las ocurrencias en una lista / MultiDict | `color=['red','blue']` |

El comportamiento de otros servidores y plataformas antiguas varía; confírmelo siempre en el sistema bajo prueba. La investigación histórica sobre este tema incluye AppSec EU 2009 (Carettoni & di Paola).

## Objetivos de las pruebas

- Identificar el backend y el método de análisis utilizado.
- Evaluar los puntos de inyección e intentar eludir los filtros de entrada mediante HPP.

## Cómo probar

Que una página o un endpoint de API específico sea vulnerable a HPP dependerá de la interacción entre varios componentes, incluida la aplicación, el servidor web, el firewall de aplicaciones web y cualquier middleware. Cuando estos componentes varían entre distintas partes de una aplicación (como en una arquitectura de microservicios), algunas partes pueden ser vulnerables y otras no.

Como se necesita un conocimiento profundo de la lógica de negocio, las pruebas de HPP requieren pruebas manuales. Las herramientas automáticas solo pueden ayudar parcialmente a los auditores, ya que tienden a generar demasiados falsos positivos. Además, HPP puede manifestarse en componentes tanto del lado del cliente como del lado del servidor.

### HPP del lado del servidor

Para probar vulnerabilidades HPP, identifique cualquier formulario o acción que permita introducir datos proporcionados por el usuario. Los parámetros de cadena de consulta en las solicitudes HTTP GET son fáciles de modificar en la barra de navegación del navegador. Si la acción del formulario envía datos mediante POST, el evaluador tendrá que usar un proxy de interceptación para manipular los datos POST mientras se envían al servidor. Una vez identificado un parámetro de entrada específico que se quiera probar, se pueden editar los datos GET o POST interceptando la solicitud, o cambiar la cadena de consulta después de que se cargue la página de respuesta. Para probar vulnerabilidades HPP, basta con agregar el mismo parámetro a los datos GET o POST, pero asignándole un valor diferente.

Por ejemplo, si se prueba el parámetro `search_string` en la cadena de consulta, la URL de la solicitud incluiría ese nombre y valor de parámetro:

```text
https://example.com/?search_string=kittens
```

El parámetro en cuestión podría estar oculto entre varios otros parámetros, pero el procedimiento es el mismo: deje los demás parámetros en su lugar y agregue el duplicado:

```text
https://example.com/?mode=guest&search_string=kittens&num_results=100
```

Agregue el mismo parámetro con un valor diferente:

```text
https://example.com/?mode=guest&search_string=kittens&num_results=100&search_string=puppies
```

Para los endpoints basados en JSON, la carga útil tendría este aspecto:

```http
POST /search HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "search_string": "kittens",
  "search_string": "puppies"
}
```

y envíe la nueva solicitud.

Analice la página de respuesta para determinar qué valores se analizaron. En el ejemplo anterior, los resultados de búsqueda podrían mostrar `kittens`, `puppies` o alguna combinación de ambos (`kittens,puppies`, `kittens~puppies` o `['kittens','puppies']`); también podrían no mostrar resultados o presentar una página de error.

Que este comportamiento predeterminado revele o no una posible vulnerabilidad depende de la validación y del filtrado de entrada específicos de cada aplicación. Como regla general: si la validación de entrada existente y otros mecanismos de seguridad son suficientes para entradas individuales, y el servidor asigna únicamente los primeros o los últimos parámetros contaminados, la contaminación de parámetros no revela una vulnerabilidad. Si los parámetros duplicados se concatenan, distintos componentes de la aplicación web utilizan ocurrencias diferentes o las pruebas generan un error, aumenta la probabilidad de que se pueda utilizar la contaminación de parámetros para activar vulnerabilidades de seguridad.

Un análisis más profundo requeriría cuatro solicitudes HTTP para cada parámetro HTTP:

1. Envíe una solicitud HTTP que contenga el nombre y el valor estándar del parámetro, y registre la respuesta HTTP. Por ejemplo: `page?par1=val1`
2. Sustituya el valor del parámetro por un valor manipulado, envíe la solicitud y registre la respuesta HTTP. Por ejemplo: `page?par1=HPP_TEST1`
3. Envíe una nueva solicitud que combine los pasos (1) y (2). Guarde también la respuesta HTTP. Por ejemplo: `page?par1=val1&par1=HPP_TEST1`
4. Compare las respuestas obtenidas en todos los pasos anteriores. Si la respuesta de (3) es diferente de la de (1) y también es diferente de la de (2), existe una falta de correspondencia que podría llegar a explotarse para activar vulnerabilidades HPP.

Crear un exploit completo a partir de una debilidad de contaminación de parámetros está fuera del alcance de este texto. Consulte las referencias para ver ejemplos y detalles.

### HPP del lado del cliente

Al igual que en el caso de HPP del lado del servidor, las pruebas manuales son la única técnica confiable para auditar aplicaciones web y detectar vulnerabilidades de contaminación de parámetros que afecten a componentes del lado del cliente. Mientras que en la variante del lado del servidor el atacante aprovecha una aplicación web vulnerable para acceder a datos protegidos o realizar acciones que no están permitidas o que no deberían ejecutarse, los ataques del lado del cliente buscan subvertir componentes y tecnologías del lado del cliente.

Para probar vulnerabilidades HPP del lado del cliente, identifique cualquier formulario o acción que permita introducir datos del usuario y muestre al usuario el resultado de esos datos. Una página de búsqueda es ideal, pero un cuadro de inicio de sesión podría no servir (por ejemplo, si no muestra al usuario un nombre de usuario no válido).

Al igual que en el caso de HPP del lado del servidor, contamine cada parámetro HTTP con `%26HPP_TEST` y busque apariciones *decodificadas de URL* de la carga útil proporcionada por el usuario:

- `&HPP_TEST`
- `&amp;HPP_TEST`
- etc.

En particular, preste atención a las respuestas que incluyan vectores HPP en los atributos `data`, `src`, `href` o en las acciones de formularios. Una vez más, que este comportamiento predeterminado revele o no una posible vulnerabilidad dependerá de la validación de entrada, el filtrado y la lógica de negocio específicos de la aplicación. Esto también puede afectar a los parámetros de cadena de consulta utilizados en `XMLHttpRequest` (XHR), `fetch` y la creación de atributos en tiempo de ejecución.

## Herramientas

- [Escáneres pasivos/activos de ZAP](https://www.zaproxy.org)

## Referencias

### Documentos técnicos

- [Contaminación de parámetros HTTP - Luca Carettoni, Stefano di Paola](https://www.acunetix.com/websitesecurity/HTTP-Parameter-Pollution-WhitePaper.pdf)
- [Ejemplo de contaminación de parámetros HTTP del lado del cliente (falla de Yahoo! Classic Mail) - Stefano di Paola](https://blog.mindedsecurity.com/2009/05/client-side-http-parameter-pollution.html)
- [Cómo detectar ataques de contaminación de parámetros HTTP - Chrysostomos Daniel](https://www.acunetix.com/blog/whitepaper-http-parameter-pollution/)
- [CAPEC-460: Contaminación de parámetros HTTP (HPP) - Evgeny Lebanidze](https://capec.mitre.org/data/definitions/460.html)
- [Detección automatizada de vulnerabilidades de contaminación de parámetros en aplicaciones web - Marco Balduzzi, Carmen Torrano Gimenez, Davide Balzarotti, Engin Kirda](https://s3.eurecom.fr/docs/ndss11_hpp.pdf)