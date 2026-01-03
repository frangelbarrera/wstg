# Esquemas de Nomenclatura de Vulnerabilidades

Con un número constantemente creciente de activos de TI para administrar, los profesionales de seguridad requieren nuevas y más poderosas herramientas para realizar análisis automatizados y a gran escala. Gracias al software, la atención puede enfocarse en los problemas más creativos e intelectualmente desafiantes. Desafortunadamente, hacer que las herramientas de evaluación de vulnerabilidades, software antivirus y sistemas de detección de intrusiones se comuniquen no es un trabajo fácil. Ha resultado en varias complicaciones técnicas, requiriendo una forma estandarizada de identificar cada defecto de software, vulnerabilidad o problema de configuración identificado. La falta de estas capacidades de interoperabilidad puede causar inconsistencias durante la evaluación de seguridad, informes confusos y esfuerzos de correlación extra entre otros problemas que producirán una importante pérdida de recursos y tiempo.

Un esquema de nomenclatura es una metodología sistemática usada para identificar cada una de esas vulnerabilidades con el fin de facilitar la identificación clara y el intercambio de información. Este objetivo se logra mediante la definición de un nombre único, estructurado y eficiente para software para cada vulnerabilidad. Hay múltiples esquemas usados para facilitar este esfuerzo, los más comunes son:

- Enumeración de Plataforma Común (`CPE`)
- Etiqueta de Identificación de Software (`SWID`)
- URL de Paquete (`PURL`)

## Etiqueta de Identificación de Software

La Etiqueta de Identificación de Software (`SWID`) es un estándar de la Organización Internacional de Normalización definido por la ISO/IEC 19770-2:2015. Las etiquetas `SWID` se usan para identificar claramente cada software como parte de ciclos de vida completos de gestión de activos de software. Este esquema de información es recomendado por el Instituto Nacional de Estándares y Tecnología (NIST) como la identificación primaria para cualquier software desarrollado o instalado. Desde `SWID` es posible generar otros esquemas como el `CPE` usado por la Base de Datos Nacional de Vulnerabilidades (NVD) mientras que el proceso inverso no es posible.

Cada etiqueta `SWID` se representa como un formato XML estandarizado. Una etiqueta `SWID` se compone de tres grupos de elementos. El primer bloque compuesto por 7 elementos predefinidos requeridos para ser considerados una etiqueta válida. Seguido por un bloque opcional que proporciona un conjunto de 30 posibles elementos predefinidos que el creador de la etiqueta puede usar para proporcionar información confiable y detallada. Finalmente, el grupo de elementos `Extended` proporciona la oportunidad para que el creador de la etiqueta defina cualquier elemento no predefinido requerido para definir con precisión el software descrito. El alto nivel de granularidad proporcionado por `SWID`, no solo proporciona la capacidad de describir un producto dado de software, sino también su estado específico en el ciclo de vida del software.

### Ejemplos

- _ACME Roadrunner Service Pack 1_ parche creado por la Corporación ACME para el producto ya instalado identificado con el `@tagId`: _com.acme.rms-ce-v4-1-5-0_:

```xml
<SoftwareIdentity
                  xmlns="https://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  name="ACME Roadrunner Service Pack 1"
                  tagId="com.acme.rms-ce-sp1-v1-0-0"
                  patch="true"
                  version="1.0.0">
  <Entity
          name="The ACME Corporation"
          regid="acme.com"
          role="tagCreator softwareCreator"/>
  <Link
        rel="patches"
        href="swid:com.acme.rms-ce-v4-1-5-0">
    ...
  </SoftwareIdentity>
```

- Red Hat Enterprise Linux versión 8 para arquitectura x86-64:

```xml
<SoftwareIdentity
                  xmlns="https://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="https://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  xml:lang="en-US"
                  name="Red Hat Enterprise Linux"
                  tagId="com.redhat.RHEL-8-x86_64"
                  tagVersion="1"
                  version="8"
                  versionScheme="multipartnumeric"
                  media="(OS:linux)">
```

## Enumeración de Plataforma Común

El esquema de Enumeración de Plataforma Común (`CPE`) es un esquema de nomenclatura estructurado para sistemas de tecnología de la información, software y paquetes mantenido por `NVD`. Comúnmente usado en conjunto con los códigos de identificación de Exposiciones y Vulnerabilidades Comunes (ej. `CVE-2017-0147`). A pesar de ser considerado un esquema obsoleto reemplazado por `SWID`, `CPE` todavía es ampliamente usado por varias soluciones de seguridad.

Definido como un Diccionario de valores registrados proporcionado por `NVD`. Cada código `CPE` puede definirse como un nombre bien formateado o como una URL. Cada valor DEBE seguir esta estructura:

- _cpe-name_ = "cpe:" component-list
- _component-list_ = part ":" vendor ":" product ":" version ":" update ":" edition ":" lang
- _component-list_ = part ":" vendor ":" product ":" version ":" update ":" edition
- _component-list_ = part ":" vendor ":" product ":" version ":" update
- _component-list_ = part ":" vendor ":" product ":" version
- _component-list_ = part ":" vendor ":" product
- _component-list_ = part ":" vendor
- _component-list_ = part
- _component-list_ = empty
- _part_ = "h" / "o" / "a" = string
- _vendor_ = string
- _product_ = string
- _version_ = string
- _update_ = string
- _edition_ = string
- _lang_ LANGTAG / empty
- _string_ = *( unreserved / pct-encoded )
- _empty_ = ""
- _unreserved_ = ALPHA / DIGIT / "-" / "." / "_" / " ̃"
- _pct-encoded_ = "%" HEXDIG HEXDIG
- _ALPHA_ = %x41-5a / %x61-7a ; A-Z or a-z
- _DIGIT_ = %x30-39 ; 0-9
- _HEXDIG_ = DIGIT / "a" / "b" / "c" / "d" / "e" / "f"
- _LANGTAG_ = cf. [RFC5646]

### Ejemplos

- Microsoft Internet Explorer 8.0.6001 Beta (cualquier edición): `wfn:[part="a",vendor="microsoft",product="internet_explorer", version="8\.0\.6001",update="beta",edition=ANY]` que se une a la siguiente URL: `cpe:/a:microsoft:internet_explorer:8.0.6001:beta`.
- Foo\Bar Big$Money Manager 2010 Special Edition para iPod Touch 80GB: `wfn:[part="a",vendor="foo\\bar",product="big\$money_manager_2010", sw_edition="special",target_sw="ipod_touch",target_hw="80gb"]`, que se une a la siguiente URL:`cpe:/a:foo%5cbar:big%24money_manager_2010:::~~special~ipod_touch~80gb~`.

## URL de Paquete

La URL de Paquete estandariza cómo se representa los metadatos de paquetes de software para que los paquetes puedan ubicarse universalmente independientemente de qué proveedor, proyecto o ecosistema pertenezcan los paquetes.

Una PURL es una cadena ASCII válida `RFC3986` definida como URL compuesta de siete elementos. Cada uno de ellos está separado por un carácter definido para hacerlo fácilmente manipulable por software.

`scheme:type/namespace/name@version?qualifiers#subpath`

La definición para cada componente es:

- _scheme_: Valor constante conforme a URL de "pkg". (**Requerido**).
- _type_: Tipo de paquete o protocolo de paquete como maven, npm, nuget, gem, pypi, etc. (**Requerido**).
- _namespace_: Valor específico de tipo para un prefijo de paquete como nombre de propietario, groupid, etc. (Opcional).
- _name_: Nombre del paquete. (**Requerido**).
- _version_: Versión del paquete. (Opcional).
- _qualifiers_: Datos calificadores extra para un paquete como un OS, arquitectura, una distro, etc. (Opcional).
- _subpath_: Subruta extra dentro de un paquete, relativa a la raíz del paquete. (Opcional).

### Ejemplos

- Software Curl, empaquetado como paquete `.deb` para Debian Jessie destinado para arquitectura i386: `pkg:deb/debian/curl@7.50.3-1?arch=i386&distro=jessie`
- Imagen Docker de Apache Casandra firmada con el hash SHA256 244fd47e07d1004f0aed9c: `pkg:docker/cassandra@sha256:244fd47e07d1004f0aed9c`

## Usos Recomendados

| USO  | RECOMENDACIÓN  |
|---|----|
| Aplicación Cliente o Servidor | CPE o SWID |
| Contenedor | PURL o SWID |
| Firmware | CPE o SWID* |
| Biblioteca o Framework (paquete) | PURL |
| Biblioteca o Framework (no paquete) | SWID |
| Sistema Operativo | CPE o SWID |
| Paquete de Sistema Operativo | PURL o SWID |

> Nota: Debido al estado obsoleto de `CPE`, la industria recomienda que nuevos proyectos implementen `SWID` cuando necesiten decidir entre los dos métodos. Aunque `CPE` se conoce como un esquema de nomenclatura ampliamente usado dentro de proyectos y soluciones activos actuales.

## Referencias

- [NISTIR 8060 - Guías para la Creación de Etiquetas de Identificación de Software Interoperables (SWID) (PDF)](https://nvlpubs.nist.gov/nistpubs/ir/2016/NIST.IR.8060.pdf)
- [NISTIR 8085 - Formando Nombres de Enumeración de Plataforma Común (CPE) desde Etiquetas de Identificación de Software (SWID)](https://csrc.nist.gov/CSRC/media/Publications/nistir/8085/draft/documents/nistir_8085_draft.pdf)
- [ISO/IEC 19770-2:2015 - Tecnología de la Información— Gestión de Activos de Software—Parte 2: Etiqueta de Identificación de Software](https://www.iso.org/standard/65666.html)
- [Diccionario Oficial de Enumeración de Plataforma Común (CPE)](https://nvd.nist.gov/products/cpe)
- [Enumeración de Plataforma Común: Especificación de Diccionario Versión 2.3](https://csrc.nist.gov/publications/detail/nistir/7697/final)
- [Especificación PURL](https://github.com/package-url/purl-spec)

### Implementaciones Conocidas

- [packageurl-go](https://github.com/package-url/packageurl-go)
- [packageurl-dotnet](https://github.com/package-url/packageurl-dotnet)
- [packageurl-java](https://github.com/package-url/packageurl-java), [package-url-java](https://github.com/sonatype/package-url-java)
- [packageurl-python](https://github.com/package-url/packageurl-python)
- [packageurl-rust](https://github.com/package-url/packageurl.rs)
- [packageurl-js](https://github.com/package-url/packageurl-js)