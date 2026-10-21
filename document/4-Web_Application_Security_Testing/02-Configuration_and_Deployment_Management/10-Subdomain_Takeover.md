# Secuestro de subdominios

|ID          |
|------------|
|WSTG-CONF-10|

## Resumen

La explotación exitosa de este tipo de vulnerabilidad permite que un adversario reclame y tome el control del subdominio de la víctima. Este ataque depende de lo siguiente:

1. El registro de subdominio del servidor DNS externo de la víctima está configurado para apuntar a un recurso, servicio externo o endpoint inexistente o inactivo. La proliferación de productos XaaS (Anything as a Service) y de servicios de nube pública ofrece muchos objetivos potenciales que se deben considerar.
2. El proveedor del servicio que hospeda el recurso, servicio externo o endpoint no gestiona adecuadamente la verificación de la propiedad del subdominio.

Si el secuestro del subdominio tiene éxito, es posible realizar una amplia variedad de ataques (servir contenido malicioso, phishing, robar cookies de sesión de usuario, credenciales, etc.). Esta vulnerabilidad podría explotarse en una amplia variedad de registros de recursos DNS, incluidos `A`, `CNAME`, `MX`, `NS`, `TXT`, etc. En cuanto a la gravedad del ataque, el secuestro de un subdominio `NS` (aunque es menos probable) tiene el mayor impacto, porque un ataque exitoso podría dar como resultado el control total de toda la zona DNS y del dominio de la víctima.

### GitHub

1. La víctima (victim.com) usa GitHub para el desarrollo y configuró un registro DNS (`coderepo.victim.com`) para acceder a él.
2. La víctima decide migrar su repositorio de código de GitHub a una plataforma comercial y no elimina `coderepo.victim.com` de su servidor DNS.
3. Un adversario descubre que `coderepo.victim.com` está hospedado en GitHub y lo reclama mediante GitHub Pages y su propia cuenta de GitHub.

### Dominio expirado

1. La víctima (victim.com) es propietaria de otro dominio (victimotherdomain.com) y usa un registro CNAME (www) para hacer referencia al otro dominio (`www.victim.com` --> `victimotherdomain.com`)
2. En algún momento, victimotherdomain.com expira y queda disponible para que cualquiera lo registre. Como el registro CNAME no se elimina de la zona DNS de victim.com, cualquiera que registre `victimotherdomain.com` tiene el control total de `www.victim.com` hasta que se elimine o actualice el registro DNS.

## Objetivos de las pruebas

- Enumerar todos los dominios posibles, anteriores y actuales, incluidos los nombres cubiertos por registros comodín.
- Identificar dominios olvidados o mal configurados, así como cualquier señal de un servicio colgante que no esté cubierta por una huella de respuesta conocida.
- Cuando se confirme que un secuestro es viable, evaluar si las cookies, la política CORS, la validación de redireccionamientos OAuth o la lista de permitidos de CSP de la aplicación principal amplían la confianza al subdominio reclamado.

## Cómo probar

### Pruebas de caja negra

Las pruebas de secuestro de subdominios constan de tres fases: enumeración de subdominios, detección automatizada basada en huellas y validación manual.

Un registro DNS colgante ocurre cuando una entrada DNS apunta a un recurso externo que ya no existe o que se ha dado de baja. Por ejemplo, un registro CNAME que apunta a un sitio de GitHub Pages eliminado por su propietario sigue resolviendo, pero el recurso subyacente no está reclamado. Un atacante puede registrar ese recurso y tomar el control del subdominio.

#### Enumeración de subdominios

Usa [subfinder](https://github.com/projectdiscovery/subfinder) para descubrir subdominios del dominio objetivo: `subfinder -d victim.com -o subdomains.txt`

Esto genera una lista de subdominios que se utilizará en la fase de detección.

Resuelve y filtra la lista para incluir solo los subdominios que tengan un registro `CNAME`, `NS` o `MX` antes de buscar huellas, ya que estos son los tipos de registro que con más frecuencia quedan colgantes. [dnsx](https://github.com/projectdiscovery/dnsx) puede hacerlo en bloque: `dnsx -l subdomains.txt -cname -resp -o resolved.txt`

#### Detección basada en huellas

La detección basada en huellas compara la respuesta HTTP de cada subdominio con una base de datos de respuestas conocidas de servicios vulnerables. El proyecto [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) mantiene esta base de datos y cataloga las cadenas de respuesta específicas que devuelven proveedores de servicios como GitHub Pages, AWS S3, Heroku y Fastly cuando un recurso no está reclamado.

Usa [subzy](https://github.com/LukaSikic/subzy) para un escaneo inicial rápido: `subzy run --targets subdomains.txt`

A continuación, usa [nuclei](https://github.com/projectdiscovery/nuclei) con las plantillas específicas para secuestros, a fin de obtener un resultado más preciso: `nuclei -l subdomains.txt -t takeovers/`

Un resultado positivo de cualquiera de las herramientas indica que la respuesta de un subdominio coincidió con una huella vulnerable conocida, lo que sugiere un registro DNS colgante que apunta a un recurso no reclamado en un servicio de terceros.

Por ejemplo, un subdominio que apunta a un sitio de GitHub Pages no reclamado devuelve la siguiente respuesta:

```http
HTTP/1.1 404 Not Found
...
<p>There isn't a GitHub Pages site here.</p>
```

Esta cadena específica aparece en can-i-take-over-xyz como la huella de GitHub Pages. Cuando subzy o nuclei detecta esta respuesta, marca el subdominio como potencialmente vulnerable.

#### Validación manual

Las herramientas automatizadas producen falsos positivos. Valida manualmente cada hallazgo antes de reportarlo.

1. Confirma el registro DNS y adónde apunta: `dig CNAME subdomain.victim.com`
2. Confirma que la respuesta coincide con la huella esperada de ese proveedor de servicios, según se indica en [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz): `curl -i http://subdomain.victim.com`
3. Confirma que el recurso no está reclamado en la plataforma del proveedor de servicios. No lo reclames.

En el caso específico de los registros `NS`, comprueba si hay un servidor de nombres delegado sin registrar: `dig NS subdomain.victim.com`; luego comprueba si el dominio del servidor de nombres devuelto está disponible para registrarse. Si el dominio del servidor de nombres no está registrado, cualquiera puede registrarlo y responder a las consultas DNS de la zona delegada, lo que constituye la variante de secuestro de mayor impacto.

#### Secuestros específicos de proveedores de nube

Los principales proveedores de nube y los servicios PaaS/JAMstack presentan patrones de secuestro particulares que merecen atención específica. Esta lista cambia a medida que los proveedores corrigen las brechas de verificación, así que contrasta el estado actual con [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) antes de reportar:

- AWS S3: Un CNAME que apunta a una URL de bucket de S3 (por ejemplo, `bucket.s3.amazonaws.com`) cuyo bucket ya no existe devuelve una respuesta `NoSuchBucket`. Cualquiera que cree un bucket con el mismo nombre en cualquier cuenta de AWS puede reclamar el subdominio.
- Azure: Los CNAME colgantes que apuntan a recursos de Azure dados de baja, como App Services, endpoints de Azure CDN/Front Door o perfiles de Traffic Manager, pueden reclamarse registrando el mismo nombre de recurso en otra suscripción de Azure.
- GCP: Existen patrones similares para los buckets de Cloud Storage y los endpoints de Firebase Hosting.
- GitHub Pages: explicado anteriormente en [GitHub](#github); sigue siendo uno de los hallazgos más comunes en entornos reales.
- Vercel y Netlify: Un CNAME que apunta a un dominio de proyecto (p. ej., `*.vercel.app`, `*.netlify.app`) eliminado o cuyo nombre se cambió normalmente puede volver a reclamarse creando un proyecto con el nombre correspondiente.
- Dominios personalizados de Cloudflare Pages / Workers: Los CNAME colgantes que apuntan a `*.pages.dev` a veces pueden reclamarse, según las protecciones a nivel de cuenta; comprueba el comportamiento actual, ya que Cloudflare ha reforzado estas protecciones con el tiempo.
- Heroku: Un CNAME a `*.herokuapp.com` correspondiente a una aplicación eliminada devuelve una respuesta "No such app" y puede volver a reclamarse creando una aplicación con el nombre correspondiente (sujeto a las reglas de nomenclatura de Heroku).
- Fastly y Shopify: ambos han aparecido históricamente en informes de secuestros debido a configuraciones colgantes de dominios personalizados; trata cualquier coincidencia con sus huellas en can-i-take-over-xyz igual que las de los proveedores anteriores.

#### Señales de servicios colgantes

Además de las huellas de cadenas de respuesta ya descritas, un registro colgante también puede manifestarse mediante señales que no consisten en una coincidencia del cuerpo de una respuesta HTTP:

- Una respuesta `NXDOMAIN`, `SERVFAIL` o una respuesta `A` vacía al resolver el propio hostname de un servicio al que apunta el CNAME del objetivo (por ejemplo, si el propio destino CNAME no se puede resolver) es una señal más sólida que una coincidencia de huella HTTP, ya que indica que el recurso ascendente ha desaparecido por completo y no que simplemente devuelve una página de «no encontrado».
- Los errores de certificado TLS (hostname que no coincide o certificado de un dominio no relacionado) en un subdominio que debería servir un proveedor específico pueden indicar que el proveedor ya no tiene un certificado emitido para ese hostname, lo que concuerda con un recurso dado de baja.
- Las páginas de error específicas del proveedor que todavía no coinciden con una huella conocida de can-i-take-over-xyz (servicios nuevos o menos comunes) también justifican una investigación manual con los mismos pasos de validación de dig/curl; considera la base de datos de huellas como un punto de partida, no como una lista exhaustiva.

#### Riesgo de CNAME comodín

Un registro DNS comodín (por ejemplo, `*.victim.com CNAME target.example-paas.com`) que apunta a un servicio vulnerable a secuestros multiplica el impacto de una sola configuración colgante: cualquier subdominio que un atacante decida solicitar (`anything.victim.com`, `admin.victim.com`, `login.victim.com`) se resuelve mediante el comodín, por lo que reclamar el recurso de destino permite controlar un conjunto ilimitado de subdominios, en lugar de uno solo. Al enumerar, comprueba explícitamente si hay registros comodín (`dig CNAME nonexistent-random-string.victim.com`) y compara el resultado con el de un subdominio real conocido, ya que los nombres cubiertos por comodines suelen faltar en las listas de subdominios obtenidas mediante reconocimiento pasivo y solo aparecen al probar directamente una etiqueta arbitraria.

#### Impacto más allá del control del contenido

Un secuestro exitoso no se limita a servir contenido controlado por el atacante en el subdominio recuperado; como el subdominio comparte el límite de confianza del dominio principal en varios mecanismos de navegador y protocolo, el impacto puede extenderse a lo siguiente:

- Cookies: una cookie cuyo ámbito no esté restringido mediante `Secure` o al hostname únicamente (por ejemplo, si se establece con un atributo `Domain` que abarca el dominio principal, o sin dicho atributo y, por tanto, queda limitada a un subdominio hermano mediante el comportamiento entre dominios relacionados) puede leerse o sobrescribirse desde el subdominio reclamado, lo que permite secuestrar sesiones de la aplicación principal.
- CORS: si la política CORS de la aplicación principal permite un patrón de origen que incluye el subdominio reclamado (una coincidencia con subdominios comodín o una comprobación de `Origin` reflejada que confía en `*.victim.com`), el atacante puede realizar solicitudes autenticadas de origen cruzado a la aplicación real desde el subdominio reclamado.
- OAuth: una configuración de cliente OAuth con un URI de redireccionamiento en el subdominio reclamado (o una validación del URI de redireccionamiento que solo comprueba el sufijo del dominio principal) permite al atacante recibir códigos de autorización o tokens destinados a la aplicación legítima.
- CSP: una política Content-Security-Policy de la aplicación principal que incluye el dominio principal con un comodín (`*.victim.com`) en la lista de permitidos como origen de script o conexión permite al atacante servir scripts o exfiltrar datos desde un origen considerado confiable por la política.

Al reportar un hallazgo de secuestro, comprueba si alguno de estos casos se aplica al subdominio específico reclamado, ya que pueden cambiar el hallazgo de «suplantación de contenido en un subdominio aislado» a «compromiso de la sesión, la autorización o la confianza en scripts de la aplicación principal».

### Pruebas de caja gris

El evaluador tiene disponible el archivo de la zona DNS, por lo que no es necesario enumerar el DNS. La metodología de pruebas es la misma.

## Remediación

Para mitigar el riesgo de secuestro de subdominios, se deben eliminar de la zona DNS los registros de recursos DNS vulnerables. Se recomiendan la monitorización continua y las comprobaciones periódicas como buenas prácticas.

## Herramientas

- [subfinder - Herramienta de enumeración de subdominios](https://github.com/projectdiscovery/subfinder)
- [dnsx - Resolución y filtrado rápido de DNS en bloque](https://github.com/projectdiscovery/dnsx)
- [subzy - Herramienta de detección de secuestro de subdominios](https://github.com/LukaSikic/subzy)
- [nuclei - Escáner de vulnerabilidades con plantillas para secuestros](https://github.com/projectdiscovery/nuclei)
- [nuclei-templates - Plantillas comunitarias para secuestros](https://github.com/projectdiscovery/nuclei-templates)
- [can-i-take-over-xyz - Base de datos de huellas de servicios vulnerables](https://github.com/EdOverflow/can-i-take-over-xyz)
- [dig - Utilidad de consulta DNS](https://man.cx/dig)
- [OWASP Domain Protect](https://owasp.org/www-project-domain-protect)

## Referencias

- [HackerOne - Guía sobre el secuestro de subdominios](https://www.hackerone.com/blog/Guide-Subdomain-Takeovers)
- [Secuestro de subdominios: conceptos básicos](https://0xpatrik.com/subdomain-takeover-basics/)
- [Secuestro de subdominios: más allá de CNAME](https://0xpatrik.com/subdomain-takeover-ns/)
- [can-i-take-over-xyz - Lista de servicios vulnerables](https://github.com/EdOverflow/can-i-take-over-xyz/)
- [OWASP AppSec Europe 2017 - Frans Rosén: secuestro de DNS mediante proveedores de nube, sin necesidad de verificación](https://2017.appsec.eu/presos/Developer/DNS%20hijacking%20using%20cloud%20providers%20%E2%80%93%20no%20verification%20needed%20-%20Frans%20Rosen%20-%20OWASP_AppSec-Eu_2017.pdf)