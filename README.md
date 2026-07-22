# OWASP Web Security Testing Guide - Traducción completa al Español

Esta es una traducción comunitaria completa al español (neutro, profesional) de la versión latest del OWASP Web Security Testing Guide (WSTG).

- Basada en la versión oficial: https://github.com/OWASP/wstg
- Todos los documentos en `/document/` traducidos al español.
- Terminología técnica estándar en español de ciberseguridad.

¡Bienvenidas mejoras y sugerencias! Abre issues o PRs.

Mantenedor: @frangelbarrera

Si usas esta traducción en reports, conferencias o cursos, ¡menciónanos!


# Guía de Pruebas de Seguridad Web de OWASP

[![Contribuciones Bienvenidas](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/frangelbarrera/wstg/issues)
[![Bandera OWASP](https://img.shields.io/badge/owasp-flagship-brightgreen.svg)](https://owasp.org/projects/)
[![Seguir en Twitter](https://img.shields.io/twitter/follow/owasp_wstg?style=social)](https://x.com/owasp_wstg)

[![Licencia Creative Commons](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/ "CC BY-SA 4.0")

Bienvenido al repositorio oficial del Proyecto de Seguridad de Aplicaciones Web Abierto Mundial® (OWASP®) Guía de Pruebas de Seguridad Web (WSTG). La WSTG es una guía completa para probar la seguridad de aplicaciones web y servicios web. Creada por los esfuerzos colaborativos de profesionales de seguridad y voluntarios dedicados, la WSTG proporciona un marco de mejores prácticas utilizado por probadores de penetración y organizaciones en todo el mundo.

Esta traducción está basada en la versión más reciente (latest) del repositorio oficial [OWASP/wstg](https://github.com/OWASP/wstg), que está trabajando en la versión de lanzamiento 5.0. Puedes [leer el documento original en inglés aquí](https://github.com/OWASP/wstg/tree/master/document).

Para la última versión estable del original, [consulta la versión 4.2](https://github.com/OWASP/wstg/releases/tag/v4.2). También disponible [en línea](https://owasp.org/www-project-web-security-testing-guide/v42/).

- [Cómo Referenciar Escenarios WSTG](#cómo-referenciar-escenarios-wstg)
    - [Enlazado](#enlazado)
- [Contribuciones, Solicitudes de Funciones y Comentarios](#contribuciones-solicitudes-de-funciones-y-comentarios)
- [Chatea Con Nosotros](#chatea-con-nosotros)
- [Líderes del Proyecto](#líderes-del-proyecto)
- [Equipo Principal](#equipo-principal)
- [Traducciones](#traducciones)

## Cómo Referenciar Escenarios WSTG

Cada escenario tiene un identificador en el formato `WSTG-<categoría>-<número>`, donde: 'categoría' es una cadena de 4 caracteres en mayúsculas que identifica el tipo de prueba o debilidad, y 'número' es un valor numérico rellenado con cero de 01 a 99. Por ejemplo: `WSTG-INFO-02` es la segunda prueba de Recopilación de Información.

Los identificadores pueden cambiar entre versiones. Por lo tanto, es preferible que otros documentos, informes o herramientas usen el formato: `WSTG-<versión>-<categoría>-<número>`, donde: 'versión' es la etiqueta de versión con puntuación removida. Por ejemplo: `WSTG-v42-INFO-02` se entendería como específicamente la segunda prueba de Recopilación de Información de la versión 4.2.

Si los identificadores se usan sin incluir el elemento `<versión>`, se debe asumir que se refieren al contenido más reciente de la Guía de Pruebas de Seguridad Web. A medida que la guía crece y cambia, esto se vuelve problemático, por lo que los escritores o desarrolladores deben incluir el elemento de versión.

### Enlazado

Los enlaces a escenarios de la Guía de Pruebas de Seguridad Web deben hacerse usando enlaces versionados, no `stable` o `latest`, que cambiarán con el tiempo. Sin embargo, es la intención del equipo del proyecto que los enlaces versionados no cambien. Por ejemplo: `https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server.html`. Nota: el elemento `v42` se refiere a la versión 4.2.

## Contribuciones, Solicitudes de Funciones y Comentarios

¡Estamos invitando activamente a nuevos contribuyentes! Para comenzar, lee la [guía de contribución](CONTRIBUTING.md).

¿Primera vez aquí? Aquí están las [sugerencias de GitHub para contribuyentes primerizos](https://github.com/frangelbarrera/wstg/contribute) a este repositorio.

Este proyecto solo es posible gracias al trabajo de muchos voluntarios dedicados. Todos están alentados a ayudar de maneras grandes y pequeñas. Aquí hay algunas formas en que puedes ayudar:

- Lee el contenido actual y ayúdanos a corregir cualquier error de ortografía o gramatical.
- Ayuda con los esfuerzos de [traducción](CONTRIBUTING.md#translation).
- Elige un problema existente y envía una solicitud de extracción para solucionarlo.
- Abre un nuevo problema para reportar una oportunidad de mejora.

Para aprender cómo contribuir exitosamente, lee la [guía de contribución](CONTRIBUTING.md).

Los contribuyentes exitosos aparecen en [la lista de autores, revisores o editores del proyecto](document/1-Frontispiece/README.md).

## Chatea Con Nosotros

Somos fáciles de encontrar en Slack:

1. Únete al Grupo OWASP Slack con este [enlace de invitación](https://owasp.org/slack/invite).
2. Únete al [canal de este proyecto, #testing-guide](https://app.slack.com/client/T04T40NHX/CJ2QDHLRJ).

Siéntete libre de hacer preguntas, sugerir ideas o compartir tus mejores recetas.

Puedes @mencionarnos en 𝕏 (Twitter) [@owasp_wstg](https://x.com/owasp_wstg).

También puedes unirte a nuestro [Grupo de Google](https://groups.google.com/a/owasp.org/forum/#!forum/testing-guide-project).

## Líderes del Proyecto

- [Rick Mitchell](https://github.com/kingthorin)
- [Elie Saad](https://github.com/ThunderSon)

## Equipo Principal

- [Rejah Rehim](https://github.com/rejahrehim)
- [Victoria Drake](https://github.com/victoriadrake)

## Traducciones

- **Español** (este repositorio): [frangelbarrera/wstg](https://github.com/frangelbarrera/wstg)
- [Portugués-BR](https://github.com/doverh/wstg-translations-pt)
- [Ruso](https://github.com/andrettv/WSTG/tree/master/WSTG-ru)
- [Persa (Farsi)](https://github.com/whoismh11/owasp-wstg-fa)
- [Turco](https://github.com/enoskom/Owasp-wstg)

---

El Proyecto de Seguridad de Aplicaciones Web Abierto Mundial y OWASP son marcas registradas de la Fundación OWASP, Inc.
