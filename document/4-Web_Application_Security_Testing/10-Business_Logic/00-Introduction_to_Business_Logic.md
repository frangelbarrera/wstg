# Introducción a la lógica de negocio

Identificar fallas de lógica de negocio en una aplicación web dinámica con múltiples funciones requiere pensar de manera poco convencional. Supongamos que el mecanismo de autenticación de una aplicación se desarrolló con la intención de realizar los pasos 1, 2 y 3, en ese orden específico, para autenticar a un usuario. ¿Qué sucede si el usuario pasa directamente del paso 1 al paso 3? En este ejemplo simplista, ¿la aplicación concede acceso al fallar de forma abierta, deniega el acceso o simplemente genera un error 500?

Se pueden plantear muchos ejemplos, pero la lección constante es «pensar más allá de lo convencional». Este tipo de vulnerabilidad no puede detectarse mediante un escáner de vulnerabilidades y depende de las habilidades y la creatividad del pentester. Además, suele ser una de las vulnerabilidades más difíciles de detectar y, por lo general, es específica de cada aplicación; al mismo tiempo, si se explota, suele ser una de las más perjudiciales para la aplicación.

La clasificación de las fallas de lógica de negocio ha sido poco estudiada, aunque la explotación de fallas de negocio ocurre con frecuencia en sistemas del mundo real y muchos investigadores de vulnerabilidades aplicadas las investigan. El principal enfoque está en las aplicaciones web. En la comunidad se debate si estos problemas representan conceptos particularmente nuevos o si son variaciones de principios bien conocidos.

Las pruebas de fallas de lógica de negocio son similares a las pruebas que realizan los evaluadores funcionales y que se centran en la lógica o en las pruebas de estados finitos. Este tipo de pruebas exige que los profesionales de seguridad piensen de manera diferente, desarrollen casos de abuso y uso indebido, y utilicen muchas de las técnicas de prueba que emplean los evaluadores funcionales. No es posible automatizar los casos de abuso de lógica de negocio; estos siguen siendo una tarea manual que depende de las habilidades del evaluador y de su conocimiento de todo el proceso de negocio y sus reglas.

## Límites y restricciones de negocio

Considere las reglas de la función de negocio que proporciona la aplicación. ¿Hay límites o restricciones para el comportamiento de las personas? Luego, considere si la aplicación hace cumplir esas reglas. Por lo general, es bastante fácil identificar los casos de prueba y análisis para verificar la aplicación si se conoce el negocio. Si es un evaluador externo, tendrá que usar el sentido común o preguntar a la empresa si la aplicación debería permitir distintas operaciones.

A veces, en aplicaciones muy complejas, el evaluador no tendrá inicialmente un conocimiento completo de todos los aspectos de la aplicación. En esas situaciones, lo mejor es que el cliente guíe al evaluador por la aplicación para que pueda comprender mejor los límites y la funcionalidad prevista de la aplicación antes de que comience la prueba propiamente dicha. Además, durante las pruebas, disponer de una vía de comunicación directa con los desarrolladores, si es posible, será de gran ayuda si surgen preguntas sobre la funcionalidad de la aplicación.

## Desafíos de las pruebas de lógica

A las herramientas automatizadas les resulta difícil comprender el contexto, por lo que estas pruebas debe realizarlas una persona. Los siguientes tres ejemplos ilustran cómo comprender la funcionalidad de la aplicación y las intenciones del desarrollador, junto con un poco de pensamiento creativo «fuera de lo común», puede quebrantar la lógica de la aplicación. El primer ejemplo parte de una manipulación sencilla de parámetros, mientras que el segundo es un ejemplo del mundo real de un proceso de varios pasos que lleva a subvertir por completo la aplicación.

**Ejemplo 1**:

Supongamos que un sitio de comercio electrónico permite a los usuarios seleccionar artículos para comprar, ver una página de resumen y luego cerrar la venta. ¿Qué ocurriría si un atacante pudiera volver a la página de resumen, conservar la misma sesión válida, introducir un precio más bajo para un artículo, completar la transacción y luego finalizar la compra?

**Ejemplo 2**:

Retener o bloquear recursos e impedir que otras personas compren esos artículos en línea puede permitir que los atacantes los compren a un precio más bajo. La contramedida para este problema consiste en implementar tiempos de espera y mecanismos que garanticen que solo se cobre el precio correcto.

**Ejemplo 3**:

¿Qué ocurriría si un usuario pudiera iniciar una transacción vinculada a su cuenta de club o de fidelidad y, después de que se añadieran puntos a su cuenta, cancelarla? ¿Seguirían aplicándose los puntos o créditos a su cuenta?

## Herramientas

Aunque existen herramientas para probar y verificar que los procesos de negocio funcionen correctamente en situaciones válidas, estas son incapaces de detectar vulnerabilidades lógicas. Por ejemplo, las herramientas no pueden detectar si un usuario puede eludir el flujo del proceso de negocio mediante la edición de parámetros, la predicción de nombres de recursos o la escalada de privilegios para acceder a recursos restringidos. Tampoco tienen ningún mecanismo para ayudar a los evaluadores humanos a sospechar que esto pueda ocurrir.

A continuación se presentan algunos tipos de herramientas comunes que pueden ser útiles para identificar problemas de lógica de negocio.

Al instalar complementos, siempre debe prestar atención a los permisos que solicitan y a sus hábitos de uso del navegador.

### Proxy de interceptación

Para observar los bloques de solicitud y respuesta del tráfico HTTP

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Proxy](https://portswigger.net/burp)

### Herramientas para desarrolladores del navegador

Para ver y modificar encabezados HTTP/HTTPS y parámetros POST, y observar el DOM del navegador

- Herramientas para desarrolladores integradas en el navegador (paneles Network, Application/Storage y Elements)

## Herramientas de prueba varias

- [Web Developer toolbar](https://chromewebstore.google.com/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - La extensión Web Developer añade un botón a la barra de herramientas del navegador que ofrece varias herramientas para desarrolladores web. Este es el port oficial de la extensión Web Developer para Firefox.
- [HTTP Request Maker for Firefox](https://addons.mozilla.org/en-US/firefox/addon/http-request-maker)
    - Request Maker es una herramienta para pruebas de penetración. Con ella se pueden capturar fácilmente las solicitudes realizadas por páginas web, manipular la URL, los encabezados y los datos POST y, por supuesto, realizar nuevas solicitudes.
- [Cookie Editor for Chrome](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
- [Cookie Editor for Firefox](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor)
    - Cookie Editor es un gestor de cookies. Permite añadir, eliminar, editar, buscar, proteger y bloquear cookies.

## Referencias

### Documentos técnicos

- [The Common Misuse Scoring System (CMSS): Metrics for Software Feature Misuse Vulnerabilities - NISTIR 7864](https://csrc.nist.gov/publications/detail/nistir/7864/final)
- [Finite State testing of Graphical User Interfaces, Fevzi Belli](https://www.semanticscholar.org/paper/b57c6c8022abfd2cb17ec785d3622027b3edfaaf)
- [Principles and Methods of Testing Finite State Machines - A Survey, David Lee, Mihalis Yannakakis](https://ieeexplore.ieee.org/document/533956)
- [Security Issues in Online Games, Jianxin Jeff Yan and Hyun-Jin Choi](https://www.researchgate.net/publication/220677013_Security_issues_in_online_games)
- [Securing Virtual Worlds Against Real Attack, Dr. Igor Muttik, McAfee](https://www.infopoint-security.de/open_downloads/2008/McAfee_wp_online_gaming_0808.pdf)
- [Seven Business Logic Flaws That Put Your Website At Risk - Jeremiah Grossman Founder and CTO, WhiteHat Security](https://www.slideshare.net/jeremiahgrossman/seven-business-logic-flaws-that-put-your-website-at-risk-harvard-07062008)
- [Toward Automated Detection of Logic Vulnerabilities in Web Applications - Viktoria Felmetsger Ludovico Cavedon Christopher Kruegel Giovanni Vigna](https://www.usenix.org/legacy/event/sec10/tech/full_papers/Felmetsger.pdf)

### Relacionado con OWASP

- [How to Prevent Business Flaws Vulnerabilities in Web Applications, Marco Morana](https://www.slideshare.net/slideshow/issa-louisville-2010morana/5391600)

### Sitios útiles

- [Business logic](https://en.wikipedia.org/wiki/Business_logic)
- [Business Logic Flaws and Yahoo Games](https://blog.jeremiahgrossman.com/2006/12/business-logic-flaws.html)
- [CWE-840: Business Logic Errors](https://cwe.mitre.org/data/definitions/840.html)
- [Defying Logic: Theory, Design, and Implementation of Complex Systems for Testing Application Logic](https://www.semanticscholar.org/paper/d14a18f08f6488f903f2f691a1d159e95d8ee04f)

### Libros

- [The Decision Model: A Business Logic Framework Linking Business and Technology, By Barbara Von Halle, Larry Goldberg, Published by CRC Press, ISBN1420082817 (2010)](https://isbnsearch.org/isbn/1420082817)