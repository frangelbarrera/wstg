# 2. Introducción

## Propósito y alcance

La Guía de pruebas de seguridad web de OWASP (WSTG) es un marco integral, de código abierto y mantenido por la comunidad, además de un conjunto de técnicas prácticas para probar la seguridad de aplicaciones y servicios web. No es una lista de verificación rígida ni un estándar de cumplimiento, sino una metodología y una referencia de técnicas diseñadas para adaptarse al modelo de amenazas, la tolerancia al riesgo y las prácticas de desarrollo de su organización.

> «La seguridad no debería ser un arte oscuro ni un secreto cerrado que solo unos pocos puedan practicar». - Eoin Keary

La guía abarca:

- Aplicaciones web tradicionales y servicios modernos nativos de la nube.
- Configuración de infraestructura, implementaciones en la nube y contenedorización.
- Controles de identidad, autenticación, autorización y gestión de sesiones.
- Validación de entradas, codificación y gestión de salidas.
- Gestión de errores y excepciones.
- Implementaciones criptográficas.
- Seguridad de la lógica de negocio y de los flujos de trabajo.
- Tecnologías del lado del cliente y seguridad de la interfaz de usuario.
- Seguridad de API y puntos de integración.

La guía no reemplaza los marcos formales de gestión de riesgos, como el [Marco de gestión de riesgos de NIST](https://csrc.nist.gov/projects/risk-management/); las metodologías integrales de modelado de amenazas; las directrices detalladas de remediación (para ello, consulte la [Serie de hojas de referencia de OWASP](https://cheatsheetseries.owasp.org/)); las revisiones exhaustivas de seguridad del código fuente (que requieren acceso de caja blanca y conocimientos de arquitectura que van más allá de las pruebas); las políticas de seguridad específicas de cada organización; ni las pruebas especializadas de aplicaciones de IA/LLM (consulte la [Guía de pruebas de IA de OWASP](https://owasp.org/www-project-ai-testing-guide/) y el [Top 10 de OWASP para aplicaciones de LLM](https://genai.owasp.org/initiatives/top-10-for-llm-and-genai/)).

## Por qué son importantes las pruebas de seguridad de aplicaciones web

Las pruebas de seguridad de aplicaciones web se han vuelto fundamentales para gestionar los riesgos en el panorama de amenazas actual. Varios factores impulsan esta urgencia:

### Impacto económico

El [Informe sobre el costo de las filtraciones de datos de IBM de 2026](https://www.ibm.com/reports/data-breach) concluyó que el costo promedio mundial de una filtración de datos alcanzó los 4,99 millones de dólares (USD), mientras que el promedio de las filtraciones en EE. UU. fue de 11,5 millones de dólares (USD). Los ataques impulsados por IA —presentes actualmente en más de una cuarta parte de las filtraciones reportadas— añaden un promedio de 1 millón de dólares (USD) al costo de cada filtración. Además de los costos directos, las filtraciones provocan interrupciones operativas, multas regulatorias, daños a la reputación y pérdida de confianza de los clientes.

### Expansión de la superficie de ataque

El modelo tradicional de seguridad basado en el perímetro ya no es aplicable. Actualmente, las aplicaciones modernas exponen riesgos en:

- API públicas y microservicios
- Infraestructura nativa de la nube y funciones sin servidor
- Integraciones de terceros y dependencias de código abierto
- Código del lado del cliente que se ejecuta en navegadores y aplicaciones móviles
- Riesgos de la cadena de suministro, incluidos paquetes comprometidos y procesos de compilación manipulados

Un programa integral de pruebas debe abordar todas estas superficies.

### Ritmo del desarrollo frente al ritmo de las amenazas

Dos fuerzas opuestas generan urgencia:

1. El desarrollo asistido por IA acelera la producción de código. Una mayor velocidad de desarrollo, si no va acompañada de disciplina de seguridad, aumenta la probabilidad de que las vulnerabilidades clásicas (fallas de inyección, autenticación deficiente y errores de lógica de negocio) se introduzcan más rápido de lo que las pruebas tradicionales pueden detectarlas.

2. Las herramientas de pruebas aumentadas con IA aceleran ciertas tareas de detección. Las herramientas de pruebas dinámicas y estáticas de seguridad de aplicaciones (DAST/SAST) que utilizan aprendizaje automático pueden detectar con mayor eficiencia vulnerabilidades comunes y conocidas. Sin embargo, estas herramientas no sustituyen el análisis experto de las personas para detectar fallas de lógica de negocio, saltos de autorización y riesgos específicos del contexto que requieren comprender el comportamiento previsto de la aplicación. Además, los ataques asistidos por IA permiten que los adversarios sondeen las defensas más rápido y adapten sus contramedidas casi en tiempo real, lo que sin duda acelerará el ritmo de las filtraciones.

La solución no consiste en elegir entre las pruebas realizadas por personas y las herramientas, sino en combinar ambas: análisis automatizados para detectar problemas evidentes, junto con pruebas manuales específicas (posiblemente también complementadas con IA) y revisión de código para encontrar fallas de diseño y lógica.

## Quién debería utilizar esta guía

La WSTG está diseñada para:

- Probadores de penetración y consultores de seguridad que realizan evaluaciones de aplicaciones.
- Equipos internos de seguridad y aseguramiento de la calidad que prueban el software durante su desarrollo.
- Desarrolladores e ingenieros de DevSecOps que integran pruebas de seguridad en los procesos de CI/CD.
- Arquitectos de seguridad que diseñan estrategias de pruebas y programas de cumplimiento.
- Auditores y responsables de cumplimiento que validan los controles de seguridad de las aplicaciones.
- Responsables de producto y gerentes de ingeniería que definen el alcance, asignan recursos y priorizan las actividades de pruebas.

Entre el público secundario se incluyen los desarrolladores de herramientas y las organizaciones que ejecutan programas públicos de recompensas por errores. Las técnicas de pruebas y la taxonomía de esta guía se aplican por igual a las pruebas internas, las pruebas de penetración de terceros y las actividades coordinadas de divulgación o de recompensas por errores. Las diferencias radican en la autorización, las reglas de participación y la forma de informar y remediar los resultados.

## Principios fundamentales de las pruebas de seguridad eficaces

Las pruebas de seguridad eficaces se basan en estos principios:

### No hay una solución mágica

Las herramientas automatizadas (DAST, SAST y análisis de dependencias) son necesarias y valiosas, pero por sí solas no son suficientes. Ni siquiera las herramientas más sofisticadas de análisis de seguridad de aplicaciones pueden igualar el razonamiento y la creatividad de un probador de seguridad con experiencia. Las herramientas son excelentes para encontrar vulnerabilidades conocidas basadas en firmas; no comprenden la lógica de negocio ni el contexto. La seguridad es un proceso de mejora continua, no un producto que pueda «comprarse» una sola vez.

### Pruebas de seguridad continuas

El enfoque tradicional —probar solo cuando el desarrollo está «completo»— es un desperdicio de recursos e ineficaz. El costo de corregir un error de seguridad crece de forma exponencial a medida que avanza por el proceso: una falla detectada durante el diseño cuesta una fracción de lo que costaría corregirla en producción. Las prácticas modernas integran las pruebas de seguridad durante todo el ciclo de vida del desarrollo:

- Durante la revisión del diseño y la arquitectura (modelado de amenazas).
- Durante el desarrollo (revisión de código seguro y pruebas de seguridad a nivel de unidad).
- Durante las pruebas de integración y de sistema (pruebas de penetración y pruebas de aceptación de seguridad).
- En producción (supervisión continua y pruebas de regresión de seguridad).

Los procesos de integración continua y entrega continua (CI/CD) ofrecen la oportunidad de automatizar las comprobaciones de seguridad básicas (SAST y análisis de dependencias) en cada compilación, manteniendo la seguridad como prioridad para todo el equipo.

### Priorización basada en el riesgo y el impacto

Priorice las pruebas según el riesgo, no según la cantidad de hallazgos. Un único salto de autorización crítico que afecte los datos de los usuarios es más importante que decenas de hallazgos de secuencias de comandos en sitios cruzados (XSS) de bajo riesgo. Desarrolle un modelo de riesgo adaptado a su aplicación (por ejemplo, la confidencialidad y la integridad de los datos de los clientes pueden ser más importantes que la disponibilidad) y centre las pruebas en los controles y las superficies más importantes para ese riesgo.

### Enfoque equilibrado de las pruebas

Ninguna técnica de pruebas es suficiente por sí sola. Un programa maduro combina:

- Inspección manual y revisión del diseño: para evaluar las políticas, la arquitectura y los procesos de seguridad.
- Modelado de amenazas: para identificar posibles vectores de ataque al inicio del ciclo de vida.
- Revisión del código fuente: para encontrar fallas sutiles (errores de lógica de negocio, debilidades criptográficas y condiciones de carrera) que las herramientas suelen pasar por alto o que son difíciles de detectar desde una perspectiva de caja negra.
- Pruebas de penetración: para validar que las vulnerabilidades puedan explotarse realmente y comprender su impacto en el mundo real.
- Análisis automatizados (SAST, DAST y SCA): para identificar vulnerabilidades comunes y conocidas a escala.

El esfuerzo relativo dedicado a cada técnica debería variar según la fase del SDLC: más énfasis en las revisiones manuales y el modelado de amenazas al principio, y en la automatización y las pruebas más adelante.

### Informes claros y prácticos

Los hallazgos de las pruebas deben ser comprensibles y prácticos para los desarrolladores y las partes interesadas del negocio. Un informe debería:

- Explicar *qué* se probó y *cómo* se probó (para que sea posible reproducir los resultados).
- Describir la *causa raíz* de cada problema (para que, idealmente, sea menos probable que vuelva a ocurrir).
- Proporcionar *directrices concretas de remediación* o referencias a las directrices de codificación segura (para que los desarrolladores sepan qué corregir).
- Asignar una *clasificación de gravedad o riesgo* que ayude a priorizar los esfuerzos de remediación.
- Comunicar claramente el *impacto en el negocio* para que las partes interesadas no técnicas comprendan por qué es importante remediar el problema.

### Adaptabilidad al contexto

No existe un enfoque de pruebas universal. La profundidad y el alcance de las pruebas deben adaptarse a la arquitectura, la sensibilidad de los datos, el modelo de amenazas y la tolerancia al riesgo de su aplicación. Una pequeña herramienta interna requiere un rigor distinto al de una aplicación financiera orientada al cliente. Un equipo de DevSecOps maduro puede adelantar las pruebas e integrarlas en procesos automatizados; un equipo que recién comienza a realizar pruebas de seguridad puede empezar con enfoques más sencillos basados en listas de verificación y madurar a partir de ahí. La WSTG proporciona el marco y la flexibilidad necesarios para ayudar con diversas arquitecturas de aplicaciones.

## Organización de la guía

Este documento está estructurado de la siguiente manera:

- Esta introducción: establece el contexto, el alcance y los principios.
- El marco de pruebas de OWASP: explica cómo se integra el marco de pruebas con las fases de desarrollo (definir, diseñar, desarrollar, probar, desplegar y mantener); aborda las prácticas de integración continua, el diseño de programas de recompensas por errores y los flujos de trabajo de pruebas aumentados con IA.
- Categorías de pruebas de seguridad de aplicaciones web: proporciona procedimientos concretos, con instrucciones para probar cada área principal (por ejemplo, autenticación, validación de entradas y lógica de negocio).
- Informes: orientación sobre formatos de informes de pruebas, evaluación de la gravedad y comunicación de resultados a diferentes públicos.
- Apéndices: herramientas de pruebas, lecturas recomendadas e historial de versiones.

Quienes se inicien en las pruebas de seguridad deberían empezar aquí y luego continuar con la [sección 3](../3-The_OWASP_Testing_Framework/README.md) para comprender cuándo y cómo encajan las pruebas en las prácticas modernas de desarrollo. Quienes buscan procedimientos específicos para las pruebas deberían ir directamente a la [sección 4](../4-Web_Application_Security_Testing/README.md).

## Cómo hacer referencia a los escenarios de la WSTG

Cada escenario tiene un identificador con el formato `WSTG-<category>-<number>`, donde: 'category' es una cadena de 4 caracteres en mayúsculas que identifica el tipo de prueba o debilidad, y 'number' es un valor numérico de dos dígitos, entre 01 y 99. Por ejemplo: `WSTG-INFO-02` es la segunda prueba de recopilación de información.

Los identificadores pueden cambiar entre versiones; por lo tanto, es preferible que otros documentos, informes o herramientas utilicen el formato `WSTG-<version>-<category>-<number>`, donde: 'version' es la etiqueta de versión sin signos de puntuación. Por ejemplo: `WSTG-v42-INFO-02` se entendería específicamente como la segunda prueba de recopilación de información de la versión 4.2.

Si se utilizan identificadores sin incluir el elemento `<version>`, se debe suponer que hacen referencia al contenido más reciente de la Guía de pruebas de seguridad web. Evidentemente, a medida que la guía crece y cambia, esto se vuelve problemático; por eso, los autores o desarrolladores deberían incluir el elemento de versión.

### Enlaces

Los enlaces a los escenarios de la Guía de pruebas de seguridad web deberían usar versiones específicas, no `stable` ni `latest`, que sin duda cambiarán con el tiempo. Sin embargo, el equipo del proyecto tiene la intención de que los enlaces con versión no cambien. Por ejemplo: `https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server`. Nota: el elemento `v42` hace referencia a la versión 4.2.

## Comentarios y sugerencias

Como en todos los proyectos de OWASP, agradecemos los comentarios y las contribuciones. Si descubre un error, una laguna en la cobertura, una referencia desactualizada o una técnica que debería añadirse o revisarse, abra una incidencia o envíe una solicitud de incorporación de cambios al [repositorio de WSTG en GitHub](https://github.com/OWASP/wstg). Esta guía es documentación viva mantenida por la comunidad, y sus contribuciones ayudan a mantenerla precisa y pertinente. Valoramos especialmente los informes de pruebas en el mundo real, tanto de las que funcionan como de las que no, para que la guía siga siendo práctica para quienes ejercen la profesión.

## Proyectos relacionados de OWASP

La Guía de pruebas de seguridad web forma parte de un conjunto más amplio de proyectos de OWASP que apoyan la creación y las pruebas de aplicaciones seguras:

- [Estándar de verificación de seguridad de aplicaciones de OWASP (ASVS)](https://owasp.org/www-project-application-security-verification-standard/): catálogo de requisitos funcionales de seguridad que proporciona una base para determinar «qué probar». ASVS v5.0 (publicado en mayo de 2025) incluye arquitecturas modernas nativas de la nube y definiciones de controles más claras.
- [OWASP Top 10 (2025)](https://owasp.org/Top10/2025/) y [OWASP API Security Top 10 (2023)](https://owasp.org/www-project-api-security/): clasificaciones basadas en el consenso de los riesgos de seguridad más críticos para las aplicaciones web y las API. Ayudan a priorizar el enfoque de las pruebas.
- [OWASP Top 10 para aplicaciones de modelos de lenguaje grandes](https://genai.owasp.org/initiatives/top-10-for-llm-and-genai/) y [Guía de pruebas de IA de OWASP](https://owasp.org/www-project-ai-testing-guide/): orientación especializada para probar aplicaciones basadas en IA y LLM, incluidos los riesgos de inyección de instrucciones, alucinaciones y envenenamiento de modelos.
- [OWASP AI Exchange](https://owaspai.org/): base de conocimientos y marco impulsados por la comunidad para comprender y gestionar los riesgos de seguridad de la IA, incluidos la gobernanza de IA, las evaluaciones de riesgos y los modelos de amenazas específicos de la IA.
- [Seguridad de aplicaciones móviles de OWASP (MASVS/MASTG)](https://owasp.org/www-project-mobile-app-security/): estándar de verificación de seguridad y guía de pruebas para aplicaciones de iOS y Android.
- [Enumeración común abierta de requisitos de OWASP (OpenCRE)](https://www.opencre.org/): marco de referencia que vincula requisitos de seguridad, métodos de prueba y controles de diversos estándares y marcos (ASVS, NIST, ISO, CWE, etc.), y ayuda a situar las directrices de la WSTG en el contexto más amplio de la seguridad.
- [Modelo de madurez de aseguramiento de software de OWASP (SAMM)](https://owaspsamm.org/model/): marco para evaluar y mejorar las prácticas de seguridad de aplicaciones de una organización en todo el SDLC.
- [OWASP Threat Dragon](https://owasp.org/www-project-threat-dragon/) y [Modelado de amenazas con Python (pytm)](https://owasp.org/www-project-pytm/): herramientas y metodologías para crear y documentar modelos de amenazas.
- [Serie de hojas de referencia de OWASP](https://cheatsheetseries.owasp.org/): orientación concisa y autorizada sobre temas de seguridad específicos (por ejemplo, autenticación, criptografía y seguridad de API), con ejemplos de código y buenas prácticas.

## Referencias

- [Informe sobre el costo de las filtraciones de datos de IBM de 2026](https://www.ibm.com/reports/data-breach)
- [Marco de ciberseguridad de NIST](https://www.nist.gov/cyberframework)
- [Marco de gestión de riesgos de NIST (RMF)](https://csrc.nist.gov/projects/risk-management/)
- [Sistema de puntuación de vulnerabilidades comunes (CVSS) v3.1](https://www.first.org/cvss/)
- [Listas de verificación de la Guía de pruebas de OWASP](https://github.com/OWASP/wstg/tree/master/checklists)

Para consultar una lista completa de lecturas recomendadas, herramientas y antecedentes históricos, véanse los apéndices A, B y E.