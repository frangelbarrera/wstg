# Recurso de herramientas de pruebas

## Introducción

Este apéndice tiene como objetivo proporcionar una lista de herramientas comunes que se usan para probar aplicaciones web. No pretende ser una referencia completa de herramientas, y la inclusión de una herramienta aquí no debe considerarse un respaldo específico de esa herramienta por parte de OWASP.

La lista contiene solo herramientas que están disponibles gratuitamente para descargar y usar (aunque pueden tener licencias que restrinjan su uso con fines comerciales).

## Pruebas web generales

### Proxies web

- [ZAP](https://www.zaproxy.org)
    - Zed Attack Proxy (ZAP) es una herramienta integrada de pruebas de penetración fácil de usar para encontrar vulnerabilidades en aplicaciones web. Está diseñada para personas con distintos niveles de experiencia en seguridad y, por ello, es ideal para desarrolladores y probadores funcionales que se inician en las pruebas de penetración.
        - ZAP proporciona escáneres automatizados y un conjunto de herramientas que permiten encontrar vulnerabilidades de seguridad manualmente.
- [Burp Suite Community Edition](https://portswigger.net/burp/communitydownload)
    - Burp Suite es un proxy de interceptación para pruebas de seguridad. Permite interceptar y modificar todo el tráfico HTTP(S) que circula en ambas direcciones; puede trabajar con certificados TLS personalizados y clientes que no reconocen proxies.
- [Telerik Fiddler](https://www.telerik.com/fiddler)
    - Fiddler es un proxy web de interceptación dirigido principalmente a desarrolladores, más que a probadores de penetración, pero aun así ofrece funciones útiles. También se conecta directamente a las API HTTP de Windows, lo que le permite interceptar el tráfico de algunos programas que no permiten configurar proxies personalizados.
- [Gori](https://github.com/hahwul/gori)
    - Gori es un proxy de interceptación controlado desde una interfaz de usuario de terminal (TUI), con pestañas para el historial, el repetidor y el fuzzer, la repetición de solicitudes de control de acceso, JWT, el análisis de tokens de sesión y las pruebas fuera de banda. El mismo proyecto se puede usar mediante scripts con `gori run` y está disponible para herramientas de IA a través de un servidor Model Context Protocol.

### Extensiones de Firefox

- [Firefox HTTP Header Live](https://addons.mozilla.org/en-US/firefox/addon/http-header-live)
    - Permite ver los encabezados HTTP de una página y mientras se navega.
- [Firefox Multi-Account Containers](https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers/)
    - Permite crear varios contenedores, cada uno con sus propias cookies y sesiones aisladas. Es útil para probar el control de acceso entre distintos usuarios.
- [Firefox Web Developer](https://addons.mozilla.org/en-US/firefox/addon/web-developer/)
    - La extensión Web Developer agrega varias herramientas para desarrolladores web al navegador.

### Extensiones de Chrome

- [Chrome Web Developer](https://chromewebstore.google.com/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - La extensión Web Developer agrega al navegador un botón en la barra de herramientas con varias herramientas para desarrolladores web. Este es el port oficial de la extensión Web Developer para Chrome.
- [Cookie Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
    - Una extensión de navegador potente y fácil de usar que permite crear, editar y eliminar rápidamente las cookies de la pestaña actual. Es útil para desarrollar, probar o administrar cookies manualmente.

### Vulnerabilidades específicas

#### Inyección SQL

- [sqlmap](https://sqlmap.org)

#### TLS

- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [testssl.sh](https://github.com/drwetter/testssl.sh)
- [SSLScan](https://github.com/rbsec/sslscan)
- [SSLLabs](https://www.ssllabs.com/ssltest/)

#### Ataques de fuerza bruta

##### Herramientas para descifrar hashes

- [John the Ripper](https://github.com/openwall/john)
- [hashcat](https://hashcat.net/hashcat/)

##### Fuerza bruta remota

- [ZAP](https://www.zaproxy.org)
- [Patator](https://github.com/lanjelot/patator)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Burp Suite Community Edition (Intruder)](https://portswigger.net/burp/communitydownload)

#### Fuzzers

- [Ffuf](https://github.com/ffuf/ffuf)
- [Wfuzz](https://github.com/xmendez/wfuzz)
- [Jdam](https://gitlab.com/michenriksen/jdam)

#### Google Hacking

- [Base de datos de Google Hacking](https://www.exploit-db.com/google-hacking-database/)

#### HTTP lento

- [Slowloris](https://github.com/gkbrk/slowloris)
- [slowhttptest](https://github.com/shekyan/slowhttptest)

### Espejo de sitios

- [wget](https://www.gnu.org/software/wget)
- [curl](https://curl.se)

### Descubrimiento de contenido

- [Gobuster](https://github.com/OJ/gobuster)
- [Waybackurls](https://github.com/tomnomnom/waybackurls)
    - Waybackurls obtiene todas las URL conocidas por Wayback Machine para un dominio determinado; es útil para el reconocimiento.
    - **Uso:**

```bash
waybackurls example.com
```

- [GAU (Get All URLs)](https://github.com/lc/gau)
    - GAU recopila URL de varios archivos públicos, incluidos Wayback Machine y Common Crawl.
    - **Uso:**

```bash
gau example.com
```

- [Unfurl](https://github.com/tomnomnom/unfurl)
    - Unfurl extrae subdominios, rutas y parámetros de las URL para realizar un análisis más detallado.
    - **Uso:**

```bash
unfurl "https://example.com/page?query=123"
```

### Detección de la superficie de ataque

- [OWASP Noir](https://github.com/owasp-noir/noir)
    - Noir analiza el código fuente para enumerar endpoints, métodos y parámetros de muchos lenguajes y frameworks. Genera texto sin formato, JSON, OpenAPI o comandos curl, y puede reproducir los endpoints descubiertos mediante ZAP o Burp Suite.
- [OWASP Attack Surface Detector](https://owasp.org/projects/attack-surface-detector)
    - Attack Surface Detector usa análisis estático de código para identificar endpoints y parámetros, y está disponible como herramienta de línea de comandos y como complemento para ZAP y Burp Suite.

### Descubrimiento de puertos y servicios

- [Nmap](https://nmap.org/)

## Escáneres de vulnerabilidades

- [ZAP](https://www.zaproxy.org)
- [Nikto](https://cirt.net/Nikto2)
- [Nuclei](https://nuclei.projectdiscovery.io/)
- [SecOps Solution](https://secopsolution.com)

## Frameworks de explotación

- [Metasploit](https://github.com/rapid7/metasploit-framework)
- [BeEF](https://github.com/beefproject/beef/)

## Distribuciones de Linux

- [Kali](https://www.kali.org)
- [Parrot](https://www.parrotsec.org)
- [Samurai](https://github.com/SamuraiWTF/samuraiwtf)
- [Santoku](https://sourceforge.net/projects/santoku/)
- [BlackArch](https://blackarch.org/downloads.html)

## Analizadores de código fuente

- [Spotbugs](https://spotbugs.github.io)
- [Find Security Bugs](https://find-sec-bugs.github.io)
- [phpcs-security-audit](https://github.com/squizlabs/PHP_CodeSniffer)
- [PMD](https://pmd.github.io)
- [Analizadores .NET de Microsoft](https://learn.microsoft.com/en-us/visualstudio/code-quality/install-net-analyzers)
- [SonarQube Community Edition](https://www.sonarsource.com/products/sonarqube/)

## Herramientas de automatización del navegador

Las herramientas de automatización del navegador se usan para validar la funcionalidad de las aplicaciones web. Algunas siguen un enfoque basado en scripts y suelen usar un framework de pruebas unitarias para crear suites de pruebas y casos de prueba. La mayoría, si no todas, pueden adaptarse para realizar pruebas específicas de seguridad además de las pruebas funcionales.

### Herramientas de código abierto

- [HtmlUnit](https://htmlunit.sourceforge.io) - HtmlUnit es un navegador sin interfaz gráfica para programas Java. Modela documentos HTML y proporciona una API para acceder a páginas, rellenar formularios, hacer clic en enlaces e interactuar con JavaScript y bibliotecas AJAX complejas. Puede simular Chrome, Firefox o Edge, según la configuración, y suele usarse para pruebas automatizadas o extracción de datos web. HtmlUnit también se puede usar como navegador compatible con Selenium mediante [htmlunit-driver](https://github.com/SeleniumHQ/htmlunit-driver). La versión estable más reciente es 4.21.0 (`org.htmlunit:htmlunit:4.21.0`).
    - Repositorio de GitHub: [HtmlUnit/htmlunit](https://github.com/HtmlUnit/htmlunit)
- [Selenium](https://www.selenium.dev)
    - Framework de pruebas basado en JavaScript, multiplataforma y que proporciona una GUI para crear pruebas.

## Recursos en línea para comparar herramientas de pruebas de seguridad

Además de las herramientas individuales que se enumeran en este apéndice, los profesionales suelen necesitar recursos que ayuden a comparar y evaluar herramientas de pruebas de seguridad en categorías como SAST, DAST, SCA y seguridad de API.

Los siguientes recursos gratuitos ofrecen comparaciones seleccionadas y orientación para la evaluación.

### Comparación de herramientas de AppSec Santa

- [AppSec Santa](https://appsecsanta.com)
    - Comparación seleccionada de más de 160 herramientas de seguridad de aplicaciones en categorías como SAST, DAST, SCA, seguridad de API, seguridad de contenedores y otras.

### Analizador de encabezados de seguridad

- [SecurityHeaders](https://securityheaders.com)
    - Herramienta en línea gratuita que analiza los encabezados de respuesta HTTP y señala protecciones de seguridad ausentes o mal configuradas, como Content Security Policy, HSTS y X-Frame-Options.

### Documentación de ZAP (Zed Attack Proxy)

Documentación oficial y recursos de aprendizaje de ZAP (Zed Attack Proxy), la herramienta de pruebas dinámicas de seguridad de aplicaciones.

- [Documentación de ZAP](https://www.zaproxy.org/docs/)

### Proyecto de plantillas de Nuclei

Un amplio repositorio de código abierto de plantillas de detección de vulnerabilidades que se usan para el análisis de seguridad automatizado.

- [Plantillas de Nuclei](https://github.com/projectdiscovery/nuclei-templates)