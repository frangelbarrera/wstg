# 0. Prefacio

*Por confirmar.*

## ¿Por qué OWASP?

Open Worldwide Application Security Project® (OWASP®) es una fundación sin fines de lucro cuya visión es que no haya más software inseguro. Su misión es ser la comunidad abierta global que impulsa el desarrollo de software seguro mediante la educación, las herramientas y la colaboración.

Crear una guía como esta requiere la experiencia de cientos de personas de todo el mundo. Hay muchas maneras de probar si existen fallos de seguridad, y esta guía intenta recoger el consenso sobre cómo realizar esas pruebas con rapidez, precisión y eficiencia. OWASP reúne a profesionales para desarrollar un enfoque basado en las mejores prácticas para abordar los problemas de seguridad del software, como comunidad independiente de proveedores y no como proveedor comercial ni organismo regulador.

Ofrecer esta guía de forma totalmente gratuita y abierta contribuye a esa misión. Cualquiera puede aprender las técnicas que se utilizan para probar problemas de seguridad comunes. La seguridad no debería ser un arte oscuro ni un secreto cerrado. Debería estar al alcance de los profesionales de seguridad, así como de desarrolladores, equipos de QA y responsables técnicos. Este proyecto mantiene esos conocimientos en manos de quienes crean y publican software.

Esta guía debe llegar a desarrolladores y testers de software. No hay suficientes especialistas en seguridad de aplicaciones para resolver el problema por sí solos. Gran parte de la responsabilidad de la seguridad de las aplicaciones recae en los desarrolladores, porque son quienes escriben el código. No se puede esperar que los desarrolladores produzcan software seguro si nunca prueban las clases de errores que introducen vulnerabilidades.

Es fundamental mantener esta información actualizada. La comunidad de OWASP mantiene y amplía esta guía mediante Issues y Pull Requests en [GitHub](https://github.com/OWASP/wstg/), para que pueda seguir el ritmo del panorama de amenazas a la seguridad de las aplicaciones.

Esta guía es testimonio de la pasión y la energía de los miembros de OWASP y de quienes colaboran como voluntarios en el proyecto. Ayuda a cambiar el mundo, una línea de código a la vez.

## Adaptación y priorización

Adopta esta guía en tu organización. Adapta el material a tus tecnologías, procesos y estructura.

Hay varios roles que suelen utilizar esta guía:

- Los desarrolladores deberían usar esta guía para ayudar a producir código seguro. Estas pruebas deberían formar parte de los procedimientos habituales de pruebas de código y pruebas unitarias.
- Los testers de software y los equipos de QA deberían usar esta guía para ampliar los casos de prueba que aplican. Detectar estas vulnerabilidades a tiempo ahorra mucho tiempo, esfuerzo y dinero más adelante.
- Los especialistas en seguridad deberían usar esta guía junto con otras técnicas para verificar que no se hayan pasado por alto problemas de seguridad.
- Los gerentes de proyecto deberían tratar los problemas de seguridad como errores en el código y el diseño, y planificar en consecuencia.

Al realizar pruebas de seguridad, vuelve a establecer las prioridades continuamente. Una aplicación puede fallar de infinitas maneras, y las organizaciones siempre disponen de tiempo y recursos limitados para las pruebas. Dedica ese tiempo a los problemas de seguridad que representen un riesgo real para la empresa. Contextualiza el riesgo en función de la aplicación y sus casos de uso.

Considera esta guía como un conjunto de técnicas para encontrar distintos tipos de problemas de seguridad. No todas las técnicas son igual de importantes para todos los sistemas. Evita tratar la guía como una lista de verificación. Siguen apareciendo nuevas vulnerabilidades, y ninguna guía puede incluir una lista exhaustiva de todo lo que se debe probar. Úsala como un buen punto de partida.

## El papel de las herramientas automatizadas

Las herramientas automatizadas de seguridad, como las de pruebas dinámicas de seguridad de aplicaciones (DAST), pruebas estáticas de seguridad de aplicaciones (SAST) y análisis de composición de software (SCA), ayudan a ampliar la escala de las pruebas y a encontrar muchas clases de problemas. Úsalas para aquello que hacen bien: ofrecer una cobertura amplia, realizar comprobaciones repetibles y hacer cumplir las políticas de referencia. Las herramientas abiertas del ecosistema, como [ZAP](https://www.zaproxy.org/), pueden ayudar en esta labor sin atar a los equipos a un único proveedor.

Estas herramientas siguen siendo genéricas. Están diseñadas para aplicaciones en general, no para tu código personalizado ni para tus reglas de negocio. Pueden encontrar muchos problemas comunes, pero a menudo carecen del contexto específico de la aplicación necesario para detectar fallos más profundos. Los problemas más graves suelen estar estrechamente relacionados con la lógica de negocio y el diseño personalizado, y es ahí donde más importan las técnicas de esta guía.

Por lo general, ejecutar una herramienta es rápido. Investigar y verificar cada hallazgo lleva tiempo. Si el objetivo es encontrar y eliminar los fallos más graves lo más rápido posible, combina el análisis automatizado con las técnicas manuales y semiautomatizadas descritas aquí. Los programas de seguridad de aplicaciones equilibrados utilizan ambos: herramientas para una cobertura amplia y esta guía para profundizar y ejercer el criterio.

## Llamado a la acción

Si desarrollas, diseñas o pruebas software, familiarízate con las directrices de pruebas de seguridad de este documento. Es una hoja de ruta para probar muchos de los problemas que afectan a las aplicaciones actuales, pero no es exhaustiva. Si encuentras errores o lagunas, abre un Issue o envía un Pull Request en el [repositorio de WSTG en GitHub](https://github.com/OWASP/wstg/). Así ayudarás a miles de personas que utilizan esta guía.

Considera [unirte a nosotros](https://owasp.org/membership/) como miembro individual o corporativo para que podamos seguir elaborando materiales como esta guía de pruebas y los demás proyectos de OWASP.

Gracias a todas las personas que han contribuido y contribuirán a esta guía. Su trabajo ayuda a que las aplicaciones de todo el mundo sean más seguras.

---

Open Worldwide Application Security Project y OWASP son marcas registradas de OWASP Foundation, Inc.