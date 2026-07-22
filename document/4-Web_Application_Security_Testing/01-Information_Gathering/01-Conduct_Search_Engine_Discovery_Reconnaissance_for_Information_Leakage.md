# Realizar Reconocimiento de Descubrimiento de Motores de Búsqueda para Fugas de Información

|ID          |
|------------|
|WSTG-INFO-01|

## Resumen

Para que los motores de búsqueda funcionen, los programas informáticos (o `robots`) obtienen datos regularmente (referido como [crawling](https://en.wikipedia.org/wiki/Web_crawler)) de miles de millones de páginas en la web. Estos programas encuentran contenido y funcionalidad web siguiendo enlaces desde otras páginas, o mirando sitemaps. Si un sitio usa un archivo especial llamado `robots.txt` para listar páginas que no quiere que los motores de búsqueda obtengan, entonces las páginas listadas allí serán ignoradas. Esta es una visión general básica - Google ofrece una explicación más en profundidad de [cómo funciona un motor de búsqueda](https://support.google.com/webmasters/answer/70897?hl=en).

Los testers pueden usar motores de búsqueda para realizar reconocimiento en sitios y aplicaciones web. Hay elementos directos e indirectos en el descubrimiento y reconocimiento de motores de búsqueda: los métodos directos se relacionan con buscar los índices y el contenido asociado desde cachés, mientras que los métodos indirectos se relacionan con aprender información sensible de diseño y configuración buscando foros, grupos de noticias, y sitios de licitación.

Una vez que un robot de motor de búsqueda ha completado el crawling, comienza indexando el contenido web basado en tags y atributos asociados, tales como `<TITLE>`, para retornar resultados de búsqueda relevantes. Si el archivo `robots.txt` no se actualiza durante la vida del sitio, y no se han usado meta tags HTML en línea que instruyan a los robots a no indexar contenido, entonces es posible que los índices contengan contenido web no intencionado para ser incluido por los propietarios. Los propietarios de sitios pueden usar el previamente mencionado `robots.txt`, meta tags HTML, autenticación, y herramientas proporcionadas por motores de búsqueda para remover tal contenido.

## Objetivos de Prueba

- Identificar qué información sensible de diseño y configuración de la aplicación, sistema, u organización está expuesta directamente (en el sitio de la organización) o indirectamente (vía servicios de terceros).

## Cómo Probar

Usa un motor de búsqueda para buscar información potencialmente sensible. Esto puede incluir:

- diagramas de red y configuraciones;
- posts archivados y emails por administradores u otro personal clave;
- procedimientos de logon y formatos de username;
- usernames, passwords, y private keys;
- archivos de configuración de servicios third-party, o cloud;
- contenido de mensajes de error reveladores; y
- aplicaciones no públicas (versiones de desarrollo, test, User Acceptance Testing (UAT), y staging de sitios).

### Search Engines

No limites las pruebas a solo un proveedor de motor de búsqueda, ya que diferentes motores de búsqueda pueden generar resultados diferentes. Los resultados de motores de búsqueda pueden variar de algunas formas, dependiendo de cuándo el motor crawleó contenido por última vez, y el algoritmo que el motor usa para determinar páginas relevantes. Considera usar los siguientes (listados alfabéticamente) motores de búsqueda:

- [Baidu](https://www.baidu.com/), el motor de búsqueda [más popular](https://en.wikipedia.org/wiki/Web_search_engine#Market_share) de China.
- [Bing](https://www.bing.com/), un motor de búsqueda propiedad y operado por Microsoft, y el segundo [más popular](https://en.wikipedia.org/wiki/Web_search_engine#Market_share) mundial. Soporta [palabras clave de búsqueda avanzadas](https://help.bing.microsoft.com/#apex/18/en-US/10001/-1).
- [binsearch.info](https://binsearch.info/), un motor de búsqueda para grupos de noticias binarios Usenet.
- [Common Crawl](https://commoncrawl.org/), "un repositorio abierto de datos de crawl web que puede ser accedido y analizado por cualquiera."
- [DuckDuckGo](https://duckduckgo.com/), un motor de búsqueda enfocado en privacidad que compila resultados de muchas [fuentes](https://help.duckduckgo.com/results/sources/) diferentes. Soporta [sintaxis de búsqueda](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/).
- [Google](https://www.google.com/), que ofrece el motor de búsqueda [más popular](https://en.wikipedia.org/wiki/Web_search_engine#Market_share) del mundo, y usa un sistema de ranking para intentar retornar los resultados más relevantes. Soporta [operadores de búsqueda](https://support.google.com/websearch/answer/2466433).
- [Internet Archive Wayback Machine](https://archive.org/web/), "construyendo una biblioteca digital de sitios de internet y otros artefactos culturales en forma digital."
- [Shodan](https://www.shodan.io/), un servicio para buscar dispositivos y servicios conectados a internet. Opciones de uso incluyen un plan gratuito limitado así como planes de suscripción pagados.

### Search Operators

Un operador de búsqueda es una palabra clave especial o sintaxis que extiende las capacidades de consultas de búsqueda regulares, y puede ayudar a obtener resultados más específicos. Generalmente toman la forma de `operator:query`. Aquí hay algunos operadores de búsqueda comúnmente soportados:

- `site:` limitará la búsqueda al dominio proporcionado.
- `inurl:` solo retornará resultados que incluyan la palabra clave en la URL.
- `intitle:` solo retornará resultados que tengan la palabra clave en el título de la página.
- `intext:` o `inbody:` solo buscará la palabra clave en el cuerpo de páginas.
- `filetype:` coincidirá solo con un tipo de archivo específico, i.e. `.png`, o `.php`.

Por ejemplo, para encontrar el contenido web de owasp.org como indexado por un motor de búsqueda típico, la sintaxis requerida es:

```text
site:owasp.org
```

![Ejemplo de Resultado de Operación de Sitio de Google](images/Google_site_Operator_Search_Results_Example_20200406.png)\
*Figure 4.1.1-1: Ejemplo de Resultado de Operación de Sitio de Google*

#### Internet Archive Wayback Machine

El [Internet Archive Wayback Machine](https://archive.org/web/) es la herramienta más comprehensiva para ver snapshots históricos de páginas web. Mantiene un extenso archivo de páginas web datando desde 1996.

Para ver versiones archivadas de un sitio, visita `https://web.archive.org/web/*/`
 seguido por la URL objetivo:

```text
https://web.archive.org/web/*/owasp.org
```

Esto mostrará una vista de calendario mostrando todos los snapshots disponibles del sitio a lo largo del tiempo.

#### Other Cached Content Services

Servicios adicionales para ver páginas web cached o archivadas incluyen:

- [archive.ph](https://archive.ph) (también conocido como archive.md) - Servicio de archivado on-demand que crea snapshots permanentes
- [CachedView](https://cachedview.com/) - Agrega páginas cached de múltiples fuentes incluyendo datos históricos de Google Cache, Wayback Machine, y otros

### Google Hacking or Dorking

Buscar con operadores puede ser una técnica de descubrimiento muy efectiva cuando combinada con la creatividad del tester. Los operadores pueden ser encadenados para efectivamente descubrir tipos específicos de archivos sensibles e información. Esta técnica, llamada [Google hacking](https://en.wikipedia.org/wiki/Google_hacking) o Dorking, también es posible usando otros motores de búsqueda, siempre que los operadores de búsqueda sean soportados.

Una base de datos de dorks, como la [Google Hacking Database](https://www.exploit-db.com/google-hacking-database), es un recurso útil que puede ayudar a descubrir información específica. Algunas categorías de dorks disponibles en esta base de datos incluyen:

- Footholds
- Files containing usernames
- Sensitive Directories
- Web Server Detection
- Vulnerable Files
- Vulnerable Servers
- Error Messages
- Files containing juicy info
- Files containing passwords
- Sensitive Online Shopping Info

## Remediación

Considera cuidadosamente la sensibilidad de información de diseño y configuración antes de que sea posteada online.

Revisa periódicamente la sensibilidad de información de diseño y configuración existente que está posteada online.