# Introducción a la Lógica de Negocio

Probar las fallas de lógica de negocio en una aplicación web dinámica multifuncional requiere pensar en métodos no convencionales. Si el mecanismo de autenticación de una aplicación se desarrolla con la intención de realizar los pasos 1, 2, 3 en ese orden específico para autenticar a un usuario. ¿Qué sucede si el usuario va del paso 1 directamente al paso 3? En este ejemplo simplista, ¿la aplicación proporciona acceso fallando abierta, deniega el acceso, o simplemente arroja un error con un mensaje 500?

Hay muchos ejemplos que se pueden hacer, pero la lección constante es "pensar fuera de la sabiduría convencional". Este tipo de vulnerabilidad no puede ser detectada por un escáner de vulnerabilidades y depende de las habilidades y creatividad del penetration tester. Además, este tipo de vulnerabilidad es usualmente una de las más difíciles de detectar, y usualmente específica de la aplicación pero, al mismo tiempo, usualmente una de las más perjudiciales para la aplicación, si se explota.

La clasificación de las fallas de lógica de negocio ha sido poco estudiada; aunque la explotación de fallas de negocio ocurre frecuentemente en sistemas del mundo real, y muchos investigadores de vulnerabilidades aplicadas las investigan. El mayor enfoque está en aplicaciones web. Hay debate dentro de la comunidad sobre si estos problemas representan conceptos particularmente nuevos, o si son variaciones de principios bien conocidos.

Las pruebas de fallas de lógica de negocio son similares a los tipos de pruebas usados por testers funcionales que se enfocan en pruebas lógicas o de estado finito. Estos tipos de pruebas requieren que los profesionales de seguridad piensen un poco diferente, desarrollen casos de abuso y uso indebido y usen muchas de las técnicas de pruebas abrazadas por testers funcionales. La automatización de casos de abuso de lógica de negocio no es posible y permanece como un arte manual que depende de las habilidades del tester y su conocimiento del proceso de negocio completo y sus reglas.

## Límites y Restricciones de Negocio

Considerar las reglas para la función de negocio siendo proporcionada por la aplicación. ¿Hay límites o restricciones en el comportamiento de las personas? Entonces considerar si la aplicación impone esas reglas. Generalmente es bastante fácil identificar los casos de prueba y análisis para verificar la aplicación si se está familiarizado con el negocio. Si se es un tester de terceros, entonces se tendrá que usar el sentido común o preguntar al negocio si diferentes operaciones deberían estar permitidas por la aplicación.

A veces, en aplicaciones muy complejas, el tester no tendrá un entendimiento completo de cada aspecto de la aplicación inicialmente. En estas situaciones, es mejor hacer que el cliente lleve al tester a través de la aplicación, para que puedan ganar un mejor entendimiento de los límites y funcionalidad prevista de la aplicación antes de que comience la prueba real. Adicionalmente, tener una línea directa con los desarrolladores (si es posible) durante las pruebas ayudará mucho, si surgen preguntas respecto a la funcionalidad de la aplicación.

## Desafíos de las Pruebas de Lógica

Las herramientas automatizadas encuentran difícil entender el contexto, por lo tanto depende de una persona realizar estos tipos de pruebas. Los siguientes dos ejemplos ilustrarán cómo entender la funcionalidad de la aplicación, las intenciones del desarrollador, y algo de pensamiento creativo "fuera de la caja" puede romper la lógica de la aplicación. El primer ejemplo comienza con una manipulación de parámetros simplista, mientras que el segundo es un ejemplo del mundo real de un proceso multi-paso que conduce a subvertir completamente la aplicación.

**Ejemplo 1**:

Suponer que un sitio de e-commerce permite a los usuarios seleccionar ítems para comprar, ver una página de resumen y luego liquidar la venta. ¿Qué pasa si un atacante fue capaz de volver a la página de resumen, manteniendo la misma sesión válida e inyectar un costo más bajo para un ítem y completar la transacción, y luego hacer checkout?

**Ejemplo 2**:

Retener/bloquear recursos y mantener a otros de comprar estos ítems en línea podría resultar en atacantes comprando ítems a un precio más bajo. La contramedida a este problema es implementar timeouts y mecanismos para asegurar que solo el precio correcto pueda ser cobrado.

**Ejemplo 3**:

¿Qué pasa si un usuario fue capaz de iniciar una transacción vinculada a su cuenta de club/lealtad y luego después de que los puntos se han añadido a su cuenta cancelar la transacción? ¿Los puntos/créditos todavía se aplicarán a su cuenta?

## Herramientas

Mientras que hay herramientas para probar y verificar que los procesos de negocio funcionan correctamente en situaciones válidas, estas herramientas son incapaces de detectar vulnerabilidades lógicas. Por ejemplo, las herramientas no tienen medios de detectar si un usuario es capaz de evadir el flujo del proceso de negocio a través de la edición de parámetros, predicción de nombres de recursos o escalada de privilegios para acceder a recursos restringidos ni tienen ningún mecanismo para ayudar a los testers humanos a sospechar de este estado de cosas.

Las siguientes son algunos tipos comunes de herramientas que pueden ser útiles para identificar problemas de lógica de negocio.

Al instalar addons siempre se debe ser diligente al considerar los permisos que solicitan y los hábitos de uso del navegador.

### Proxy de Intercepción

Para Observar los Bloques de Solicitud y Respuesta del Tráfico HTTP

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Proxy](https://portswigger.net/burp)

### Plug-ins de Navegador Web

Para ver y modificar encabezados HTTP/HTTPS, parámetros post, y observar el DOM del Navegador

- [Tamper Data for FF Quantum](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum)
- [Tamper Chrome (for Google Chrome)](https://chrome.google.com/webstore/detail/tamper-chrome-extension/hifhgpdkfodlpnlmlnmhchnkepplebkb)

## Herramientas Varias de Prueba

- [Web Developer toolbar](https://chrome.google.com/webstore/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - La extensión Web Developer añade un botón de toolbar al navegador con varias herramientas de desarrollador web. Este es el port oficial de la extensión Web Developer para Firefox.
- [HTTP Request Maker for Chrome](https://chrome.google.com/webstore/detail/kajfghlhfkcocafkcjlajldicbikpgnp)
- [HTTP Request Maker for Firefox](https://addons.mozilla.org/en-US/firefox/addon/http-request-maker)
    - Request Maker es una herramienta para pruebas de penetración. Con ella se puede capturar fácilmente solicitudes hechas por páginas web, alterar la URL, encabezados y datos POST y, por supuesto, hacer nuevas solicitudes
- [Cookie Editor for Chrome](https://chrome.google.com/webstore/detail/fngmhnnpilhplaeedifhccceomclgfbg)
- [Cookie Editor for Firefox](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor)
    - Cookie Editor es un gestor de cookies. Se pueden añadir, eliminar, editar, buscar, proteger y bloquear cookies

## Referencias

### Whitepapers

- [The Common Misuse Scoring System (CMSS): Metrics for Software Feature Misuse Vulnerabilities - NISTIR 7864](https://csrc.nist.gov/publications/detail/nistir/7864/final)
- [Finite State testing of Graphical User Interfaces, Fevzi Belli](https://pdfs.semanticscholar.org/b57c/6c8022abfd2cb17ec785d3622027b3edfaaf.pdf)
- [Principles and Methods of Testing Finite State Machines - A Survey, David Lee, Mihalis Yannakakis](https://ieeexplore.ieee.org/document/533956)
- [Security Issues in Online Games, Jianxin Jeff Yan and Hyun-Jin Choi](https://www.researchgate.net/publication/220677013_Security_issues_in_online_games)
- [Securing Virtual Worlds Against Real Attack, Dr. Igor Muttik, McAfee](https://www.info-point-security.com/open_downloads/2008/McAfee_wp_online_gaming_0808.pdf)
- [Seven Business Logic Flaws That Put Your Website At Risk – Jeremiah Grossman Founder and CTO, WhiteHat Security](https://www.slideshare.net/jeremiahgrossman/seven-business-logic-flaws-that-put-your-website-at-risk-harvard-07062008)
- [Toward Automated Detection of Logic Vulnerabilities in Web Applications - Viktoria Felmetsger Ludovico Cavedon Christopher Kruegel Giovanni Vigna](https://www.usenix.org/legacy/event/sec10/tech/full_papers/Felmetsger.pdf)

### Relacionado con OWASP

- [How to Prevent Business Flaws Vulnerabilities in Web Applications, Marco Morana](https://www.slideshare.net/slideshow/issa-louisville-2010morana/5391600)

### Sitios Útiles

- [Business logic](https://en.wikipedia.org/wiki/Business_logic)
- [Business Logic Flaws and Yahoo Games](https://blog.jeremiahgrossman.com/2006/12/business-logic-flaws.html)
- [CWE-840: Business Logic Errors](https://cwe.mitre.org/data/definitions/840.html)
- [Defying Logic: Theory, Design, and Implementation of Complex Systems for Testing Application Logic](https://pdfs.semanticscholar.org/d14a/18f08f6488f903f2f691a1d159e95d8ee04f.pdf)

### Libros

- [The Decision Model: A Business Logic Framework Linking Business and Technology, By Barbara Von Halle, Larry Goldberg, Published by CRC Press, ISBN1420082817 (2010)](https://isbnsearch.org/isbn/1420082817)
