# Esquemas de nomenclatura de vulnerabilidades

Con un número cada vez mayor de activos de TI que administrar, los profesionales de seguridad necesitan herramientas nuevas y más potentes para realizar análisis automatizados y a gran escala. Gracias al software, es posible centrar la atención en los problemas más creativos y que suponen un mayor desafío intelectual. Desafortunadamente, lograr que las herramientas de evaluación de vulnerabilidades, el software antivirus y los sistemas de detección de intrusiones se comuniquen no es una tarea fácil. Esto ha dado lugar a varias complicaciones técnicas y ha hecho necesario contar con una forma estandarizada de identificar cada defecto de software, vulnerabilidad o problema de configuración detectado. La falta de capacidades de interoperabilidad puede provocar inconsistencias durante las evaluaciones de seguridad, informes confusos y esfuerzos adicionales de correlación, entre otros problemas, lo que supone un desperdicio importante de recursos y tiempo.

Un esquema de nomenclatura es una metodología sistemática que se utiliza para identificar cada una de esas vulnerabilidades y facilitar su identificación clara y el intercambio de información. Este objetivo se logra definiendo un nombre único, estructurado y eficiente para el software para cada vulnerabilidad. Existen varios esquemas que facilitan esta tarea; los más comunes son:

- Enumeración de plataformas comunes (`CPE`)
- Etiqueta de identificación de software (`SWID`)
- URL de paquete (`PURL`)

## Etiqueta de identificación de software

La etiqueta de identificación de software (`SWID`) es un estándar de la Organización Internacional de Normalización definido por la norma ISO/IEC 19770-2:2015. Las etiquetas `SWID` se utilizan para identificar claramente cada software como parte de los ciclos de vida integrales de gestión de activos de software. El Instituto Nacional de Estándares y Tecnología (NIST) recomienda utilizar este esquema de información como identificación principal para cualquier software desarrollado o instalado. A partir de `SWID` es posible generar otros esquemas, como `CPE`, que utiliza la Base de Datos Nacional de Vulnerabilidades (NVD); no es posible realizar el proceso inverso.

Cada etiqueta `SWID` se representa en un formato XML estandarizado. Una etiqueta `SWID` se compone de tres grupos de elementos. El primer bloque está compuesto por 7 elementos predefinidos necesarios para que la etiqueta se considere válida. A continuación, hay un bloque opcional que proporciona un conjunto de 30 posibles elementos predefinidos que el creador de la etiqueta puede utilizar para proporcionar información confiable y detallada. Por último, el grupo de elementos `Extended` permite al creador de la etiqueta definir cualquier elemento no predefinido que sea necesario para describir con precisión el software. El alto nivel de granularidad que ofrece `SWID` permite describir no solo un producto de software determinado, sino también su estado específico en el ciclo de vida del software.

### Ejemplos

- Parche _ACME Roadrunner Service Pack 1_, creado por ACME Corporation para el producto ya instalado identificado con `@tagId`: _com.acme.rms-ce-v4-1-5-0_:

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
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="https://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  xml:lang="en-US"
                  name="Red Hat Enterprise Linux"
                  tagId="com.redhat.RHEL-8-x86_64"
                  tagVersion="1"
                  version="8"
                  versionScheme="multipartnumeric"
                  media="(OS:linux)">
```

## Enumeración de plataformas comunes

El esquema de enumeración de plataformas comunes (`CPE`) es un esquema de nomenclatura estructurado para sistemas de tecnología de la información, software y paquetes, mantenido por `NVD`. Se utiliza habitualmente junto con los códigos de identificación de vulnerabilidades y exposiciones comunes (por ejemplo, `CVE-2017-0147`). Aunque se considera un esquema obsoleto, sustituido por `SWID`, `CPE` sigue siendo ampliamente utilizado por varias soluciones de seguridad.

Se define como un diccionario de valores registrados proporcionado por `NVD`. Cada código `CPE` puede definirse como un nombre con formato correcto o como una URL. Cada valor DEBE seguir esta estructura:

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

- Microsoft Internet Explorer 8.0.6001 Beta (cualquier edición): `wfn:[part="a",vendor="microsoft",product="internet_explorer", version="8\.0\.6001",update="beta",edition=ANY]`, que se vincula a la siguiente URL: `cpe:/a:microsoft:internet_explorer:8.0.6001:beta`.
- Foo\Bar Big$Money Manager 2010 Special Edition para iPod Touch de 80 GB: `wfn:[part="a",vendor="foo\\bar",product="big\$money_manager_2010", sw_edition="special",target_sw="ipod_touch",target_hw="80gb"]`, que se vincula a la siguiente URL: `cpe:/a:foo%5cbar:big%24money_manager_2010:::~~special~ipod_touch~80gb~`.

## URL de paquete

Package URL estandariza la representación de los metadatos de paquetes de software para que estos puedan localizarse universalmente, independientemente del proveedor, proyecto o ecosistema al que pertenezcan.

Una PURL es una cadena ASCII válida que cumple con `RFC3986`, definida como una URL compuesta por siete elementos. Cada elemento está separado por un carácter definido para que el software pueda manipularlo fácilmente.

`scheme:type/namespace/name@version?qualifiers#subpath`

La definición de cada componente es la siguiente:

- _scheme_: Valor constante conforme al esquema de URL "pkg". (**Obligatorio**).
- _type_: Tipo de paquete o protocolo de paquete, como maven, npm, nuget, gem, pypi, etc. (**Obligatorio**).
- _namespace_: Valor específico del tipo para un prefijo de paquete, como el nombre de su propietario, groupid, etc. (Opcional).
- _name_: Nombre del paquete. (**Obligatorio**).
- _version_: Versión del paquete. (Opcional).
- _qualifiers_: Datos adicionales para calificar un paquete, como un sistema operativo, una arquitectura, una distribución, etc. (Opcional).
- _subpath_: Subruta adicional dentro de un paquete, relativa a la raíz del paquete. (Opcional).

### Ejemplos

- Software Curl, empaquetado como paquete `.deb` para Debian Jessie y destinado a una arquitectura i386: `pkg:deb/debian/curl@7.50.3-1?arch=i386&distro=jessie`
- Imagen Docker de Apache Casandra firmada con el hash SHA256 244fd47e07d1004f0aed9c: `pkg:docker/cassandra@sha256:244fd47e07d1004f0aed9c`

## Usos recomendados

| USO  | RECOMENDACIÓN  |
|---|---|
| Aplicación cliente o servidor | CPE o SWID |
| Contenedor | PURL o SWID |
| Firmware | CPE o SWID* |
| Biblioteca o framework (paquete) | PURL |
| Biblioteca o framework (sin paquete) | SWID |
| Sistema operativo | CPE o SWID |
| Paquete de sistema operativo | PURL o SWID |

> Nota: Debido a que `CPE` está obsoleto, la industria parece recomendar que los proyectos nuevos implementen `SWID` cuando deban elegir entre los dos métodos. Aun así, `CPE` sigue siendo un esquema de nomenclatura ampliamente utilizado en proyectos y soluciones activos.

## Referencias

- [NISTIR 8060 - Directrices para la creación de etiquetas interoperables de identificación de software (SWID) (PDF)](https://nvlpubs.nist.gov/nistpubs/ir/2016/NIST.IR.8060.pdf)
- [NISTIR 8085 - Cómo formar nombres de enumeración de plataformas comunes (CPE) a partir de etiquetas de identificación de software (SWID)](https://csrc.nist.gov/CSRC/media/Publications/nistir/8085/draft/documents/nistir_8085_draft.pdf)
- [ISO/IEC 19770-2:2015 - Tecnología de la información: gestión de activos de software, parte 2: etiqueta de identificación de software](https://www.iso.org/standard/65666.html)
- [Diccionario oficial de enumeración de plataformas comunes (CPE)](https://nvd.nist.gov/products/cpe)
- [Enumeración de plataformas comunes: especificación del diccionario, versión 2.3](https://csrc.nist.gov/publications/detail/nistir/7697/final)
- [Especificación de PURL](https://github.com/package-url/purl-spec)

### Implementaciones conocidas

- [packageurl-go](https://github.com/package-url/packageurl-go)
- [packageurl-dotnet](https://github.com/package-url/packageurl-dotnet)
- [packageurl-java](https://github.com/package-url/packageurl-java), [package-url-java](https://github.com/sonatype/package-url-java)
- [packageurl-python](https://github.com/package-url/packageurl-python)
- [packageurl-rust](https://github.com/package-url/packageurl.rs)
- [packageurl-js](https://github.com/package-url/packageurl-js)