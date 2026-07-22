# Recurso de Herramientas de Pruebas

## Introducción

Este apéndice tiene la intención de proporcionar una lista de herramientas comunes que se usan para pruebas de aplicaciones web. No tiene como objetivo ser una referencia completa de herramientas, y la inclusión de una herramienta aquí no debe verse como un respaldo específico de esa herramienta por parte de OWASP.

La lista contiene solo herramientas que están disponibles gratuitamente para descargar y usar (aunque pueden tener licencias que restrinjan su uso para actividad comercial).

## Pruebas Web Generales

### Proxies Web

- [ZAP](https://www.zaproxy.org)
    - El Zed Attack Proxy (ZAP) es una herramienta integrada de pruebas de penetración fácil de usar para encontrar vulnerabilidades en aplicaciones web. Está diseñado para ser usado por personas con una amplia gama de experiencia en seguridad y como tal es ideal para desarrolladores y probadores funcionales que son nuevos en pruebas de penetración.
    - ZAP proporciona escáneres automatizados así como un conjunto de herramientas que le permiten encontrar vulnerabilidades de seguridad manualmente.
- [Burp Suite Community Edition](https://portswigger.net/burp/communitydownload)
    - Burp Suite es un proxy interceptador para pruebas de seguridad. Permite interceptar y modificar todo el tráfico HTTP(S) pasando en ambas direcciones, puede trabajar con certificados TLS personalizados y clientes no conscientes de proxy.
- [Telerik Fiddler](https://www.telerik.com/fiddler)
    - Fiddler es un proxy web interceptador que está principalmente dirigido a desarrolladores en lugar de probadores de penetración, pero aún proporciona funcionalidad útil. También se conecta directamente a las APIs HTTP de Windows, permitiéndole interceptar tráfico de algunos software que no permiten configurar proxies personalizados.

### Extensiones de Firefox

- [Firefox HTTP Header Live](https://addons.mozilla.org/en-US/firefox/addon/http-header-live)
    - Ver encabezados HTTP de una página y mientras navega.
- [Firefox Multi-Account Containers](https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers/)
    - Crear múltiples contenedores, cada uno de los cuales tiene sus propias cookies y sesiones aisladas. Útil para probar control de acceso entre diferentes usuarios.
- [Firefox Tamper Data](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
    - Usar Tamper Data para ver y modificar encabezados HTTP/HTTPS y parámetros post
- [Firefox Web Developer](https://addons.mozilla.org/en-US/firefox/addon/web-developer/)
    - La extensión Web Developer agrega varias herramientas de desarrollador web al navegador.

### Extensiones de Chrome

- [Chrome Web Developer](https://chrome.google.com/webstore/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - La extensión Web Developer agrega un botón de barra de herramientas al navegador con varias herramientas de desarrollador web. Este es el puerto oficial de la extensión Web Developer para Chrome.
- [HTTP Request Maker](https://chrome.google.com/webstore/detail/kajfghlhfkcocafkcjlajldicbikpgnp?hl=en-US)
    - Request Maker es una herramienta para pruebas de penetración. Con ella puede capturar fácilmente solicitudes hechas por páginas web, manipular la URL, encabezados y datos POST y, por supuesto, hacer nuevas solicitudes
- [Cookie Editor](https://chrome.google.com/webstore/detail/fngmhnnpilhplaeedifhccceomclgfbg?hl=en-US)
    - Edit This Cookie es un gestor de cookies. Puede agregar, eliminar, editar, buscar, proteger y bloquear cookies

### Pruebas para Vulnerabilidades Específicas

#### Pruebas para Inyección SQL

- [sqlmap](https://sqlmap.org)

#### Pruebas TLS

- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [testssl.sh](https://github.com/drwetter/testssl.sh)
- [SSLScan](https://github.com/rbsec/sslscan)
- [SSLLabs](https://www.ssllabs.com/ssltest/)

#### Pruebas para Ataques de Fuerza Bruta

##### Crackers de Hash

- [John the Ripper](https://github.com/openwall/john)
- [hashcat](https://hashcat.net/hashcat/)

##### Fuerza Bruta Remota

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

#### HTTP Lento

- [Slowloris](https://github.com/gkbrk/slowloris)
- [slowhttptest](https://github.com/shekyan/slowhttptest)

### Espejo de Sitio

- [wget](https://www.gnu.org/software/wget/)
- [wget para windows](https://gnuwin32.sourceforge.net/packages/wget.htm)
- [curl](https://curl.haxx.se)

### Descubrimiento de Contenido

- [Gobuster](https://github.com/OJ/gobuster)
- [Waybackurls](https://github.com/tomnomnom/waybackurls)
    - Waybackurls obtiene todas las URLs conocidas por la Wayback Machine para un dominio dado, útil para reconocimiento.
    - **Uso:**

```bash
waybackurls example.com
```

- [GAU (Get All URLs)](https://github.com/lc/gau)
    - GAU recopila URLs de múltiples archivos públicos, incluyendo la Wayback Machine y Common Crawl.
    - **Uso:**

```bash
gau example.com
```

- [Unfurl](https://github.com/tomnomnom/unfurl)
    - Unfurl extrae subdominios, rutas y parámetros de URLs para análisis más profundo.
    - **Uso:**

```bash
unfurl "https://example.com/page?query=123"
```

### Descubrimiento de Puerto y Servicio

- [Nmap](https://nmap.org/)

## Escáneres de Vulnerabilidades

- [ZAP](https://www.zaproxy.org)
- [Nikto](https://cirt.net/Nikto2)
- [Nuclei](https://nuclei.projectdiscovery.io/)
- [SecOps Solution](https://secopsolution.com)

## Marcos de Explotación

- [Metasploit](https://github.com/rapid7/metasploit-framework)
- [BeEF](https://github.com/beefproject/beef/)

## Distribuciones Linux

- [Kali](https://www.kali.org)
- [Parrot](https://www.parrotsec.org)
- [Samurai](https://github.com/SamuraiWTF/samuraiwtf)
- [Santoku](https://sourceforge.net/projects/santoku/)
- [BlackArch](https://blackarch.org/downloads.html)

## Analizadores de Código Fuente

- [Spotbugs](https://spotbugs.github.io)
- [Find Security Bugs](https://find-sec-bugs.github.io)
- [phpcs-security-audit](https://github.com/squizlabs/PHP_CodeSniffer)
- [PMD](https://pmd.github.io)
- [Analizadores .NET de Microsoft](https://docs.microsoft.com/en-us/visualstudio/code-quality/install-net-analyzers)
- [SonarQube Community Edition](https://www.sonarqube.org)

## Herramientas de Automatización de Navegador

Las herramientas de automatización de navegador se usan para validar la funcionalidad de aplicaciones web. Algunas siguen un enfoque scriptado y típicamente hacen uso de un marco de pruebas unitarias para construir suites de pruebas y casos de prueba. La mayoría, si no todas, pueden adaptarse para realizar pruebas específicas de seguridad además de pruebas funcionales.

### Herramientas de Código Abierto

- [HtmlUnit](https://htmlunit.sourceforge.net)
    - Un marco basado en Java y JUnit que usa Apache HttpClient como transporte.
    - Muy robusto y configurable y se usa como motor para una serie de otras herramientas de prueba.
- [Selenium](https://www.selenium.dev)
    - Marco de pruebas basado en JavaScript, multiplataforma y proporciona una GUI para crear pruebas.