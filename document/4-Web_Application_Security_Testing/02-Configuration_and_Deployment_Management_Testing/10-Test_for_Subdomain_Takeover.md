# Probar Subdomain Takeover

|ID          |
|------------|
|WSTG-CONF-10|

## Resumen

Una explotación exitosa de este tipo de vulnerabilidad permite a un adversario reclamar y tomar control del subdominio de la víctima. Este ataque depende de lo siguiente:

1. El registro de subdominio del servidor DNS externo de la víctima está configurado para apuntar a un recurso/servicio externo/endpoint no existente o no activo. La proliferación de productos XaaS (Anything as a Service) y servicios en la nube públicos ofrecen muchos objetivos potenciales a considerar.
2. El proveedor de servicios que hospeda el recurso/servicio externo/endpoint no maneja la verificación de propiedad del subdominio apropiadamente.

Si el subdomain takeover es exitoso, una amplia variedad de ataques son posibles (servir contenido malicioso, phishing, robar cookies de sesión de usuario, credenciales, etc.). Esta vulnerabilidad podría explotarse para una amplia variedad de registros de recursos DNS incluyendo: `A`, `CNAME`, `MX`, `NS`, `TXT` etc. En términos de severidad del ataque, un subdomain takeover de `NS` (aunque menos probable) tiene el mayor impacto, porque un ataque exitoso podría resultar en control completo sobre toda la zona DNS y el dominio de la víctima.

### GitHub

1. La víctima (victim.com) usa GitHub para desarrollo y configuró un registro DNS (`coderepo.victim.com`) para acceder a él.
2. La víctima decide migrar su repositorio de código de GitHub a una plataforma comercial y no remueve `coderepo.victim.com` de su servidor DNS.
3. Un adversario descubre que `coderepo.victim.com` está hospedado en GitHub y lo reclama usando GitHub Pages y su propia cuenta de GitHub.

### Dominio Expirado

1. La víctima (victim.com) posee otro dominio (victimotherdomain.com) y usa un registro CNAME (www) para referenciar el otro dominio (`www.victim.com` --> `victimotherdomain.com`)
2. En algún punto, victimotherdomain.com expira, volviéndose disponible para registro por cualquiera. Como el registro CNAME no se elimina de la zona DNS de victim.com, cualquiera que registre `victimotherdomain.com` tiene control completo sobre `www.victim.com` hasta que el registro DNS se remueva o actualice.

## Objetivos de Prueba

- Enumerar todos los dominios posibles (previos y actuales).
- Identificar cualquier dominio olvidado o mal configurado.

## Cómo Probar

### Pruebas de Caja Negra

Las pruebas para subdomain takeover siguen tres fases: enumeración de subdominios, detección automatizada basada en fingerprint, y validación manual.

Un registro DNS colgante ocurre cuando una entrada DNS apunta a un recurso externo que ya no existe o ha sido desprovisto. Por ejemplo, un registro CNAME apuntando a un sitio de GitHub Pages que el propietario eliminó todavía resuelve, pero el recurso subyacente está sin reclamar. Un atacante puede registrar ese recurso y tomar control del subdominio.

#### Enumeración de Subdominios

Usar [subfinder](https://github.com/projectdiscovery/subfinder) para descubrir subdominios para el dominio objetivo: `subfinder -d victim.com -o subdomains.txt`

Esto produce una lista de subdominios para usar en la fase de detección.

#### Detección Basada en Fingerprint

La detección basada en fingerprint funciona comparando la respuesta HTTP de cada subdominio contra una base de datos de respuestas de servicios vulnerables conocidos. El proyecto [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) mantiene esta base de datos, catalogando las cadenas de respuesta específicas devueltas por proveedores de servicios tales como GitHub Pages, AWS S3, Heroku y Fastly cuando un recurso está sin reclamar.

Usar [subzy](https://github.com/LukaSikic/subzy) para un escaneo inicial rápido: `subzy run --targets subdomains.txt`

Seguir con [nuclei](https://github.com/projectdiscovery/nuclei) usando las plantillas dedicadas de takeover para un resultado más preciso: `nuclei -l subdomains.txt -t takeovers/`

Un resultado positivo de cualquiera de las herramientas indica que la respuesta de un subdominio coincidió con un fingerprint vulnerable conocido, sugiriendo un registro DNS colgante apuntando a un recurso sin reclamar en un servicio de terceros.

Por ejemplo, un subdominio apuntando a un sitio de GitHub Pages sin reclamar devuelve la siguiente respuesta:

```http
HTTP/1.1 404 Not Found
...
<p>There isn't a GitHub Pages site here.</p>
```

Esta cadena específica está listada en can-i-take-over-xyz como el fingerprint de GitHub Pages. Cuando subzy o nuclei coincide esta respuesta, marca el subdominio como potencialmente vulnerable.

#### Validación Manual

Las herramientas automatizadas producen falsos positivos. Validar cada hallazgo manualmente antes de reportarlo.

1. Confirmar el registro DNS y a dónde apunta: `dig CNAME subdomain.victim.com`

1. Confirmar que la respuesta coincide con el fingerprint esperado para ese proveedor de servicios como se lista en [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz): `curl -i http://subdomain.victim.com`

1. Confirmar que el recurso está sin reclamar en la plataforma del proveedor de servicios. No reclamarlo.

#### Takeovers Específicos de la Nube

Los principales proveedores de la nube tienen patrones de takeover distintos que merecen atención específica:

- AWS S3: Un CNAME apuntando a una URL de bucket S3 (por ejemplo, `bucket.s3.amazonaws.com`) donde el bucket ya no existe devuelve una respuesta `NoSuchBucket`. Cualquiera que cree un bucket con el mismo nombre en cualquier cuenta de AWS puede reclamar el subdominio.
- Azure: CNAMEs colgantes apuntando a recursos de Azure desprovistos tales como App Services o endpoints de Traffic Manager pueden ser reclamados registrando el mismo nombre de recurso en una suscripción de Azure diferente.
- GCP: Patrones similares existen para buckets de Cloud Storage y endpoints de Firebase Hosting.

### Pruebas de Caja Gris

El tester tiene el archivo de zona DNS disponible, lo que significa que la enumeración DNS no es necesaria. La metodología de pruebas es la misma.

## Remediación

Para mitigar el riesgo de subdomain takeover, los registros de recursos DNS vulnerables deberían removerse de la zona DNS. Se recomiendan monitoreo continuo y verificaciones periódicas como mejores prácticas.

## Herramientas

- [subfinder - Herramienta de enumeración de subdominios](https://github.com/projectdiscovery/subfinder)
- [subzy - Herramienta de detección de subdomain takeover](https://github.com/LukaSikic/subzy)
- [nuclei - Escáner de vulnerabilidades con plantillas de takeover](https://github.com/projectdiscovery/nuclei)
- [nuclei-templates - Plantillas de takeover de la comunidad](https://github.com/projectdiscovery/nuclei-templates)
- [can-i-take-over-xyz - Base de datos de fingerprints de servicios vulnerables](https://github.com/EdOverflow/can-i-take-over-xyz)
- [dig - Utilidad de lookup DNS](https://man.cx/dig)
- [OWASP Domain Protect](https://owasp.org/www-project-domain-protect)

## Referencias

- [HackerOne - A Guide To Subdomain Takeovers](https://www.hackerone.com/blog/Guide-Subdomain-Takeovers)
- [Subdomain Takeover: Basics](https://0xpatrik.com/subdomain-takeover-basics/)
- [Subdomain Takeover: Going beyond CNAME](https://0xpatrik.com/subdomain-takeover-ns/)
- [can-i-take-over-xyz - A list of vulnerable services](https://github.com/EdOverflow/can-i-take-over-xyz/)
- [OWASP AppSec Europe 2017 - Frans Rosén: DNS hijacking using cloud providers – no verification needed](https://2017.appsec.eu/presos/Developer/DNS%20hijacking%20using%20cloud%20providers%20%E2%80%93%20no%20verification%20needed%20-%20Frans%20Rosen%20-%20OWASP_AppSec-Eu_2017.pdf)
