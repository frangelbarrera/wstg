# 4.0 Introducción y Objetivos

Esta sección describe la metodología de pruebas de seguridad de aplicaciones web de OWASP y explica cómo probar evidencia de vulnerabilidades dentro de la aplicación debido a deficiencias con controles de seguridad identificados.

## ¿Qué es Pruebas de Seguridad de Aplicaciones Web?

Una prueba de seguridad es un método de evaluar la seguridad de un sistema informático o red mediante la validación y verificación metódica de la efectividad de controles de seguridad de aplicaciones. Una prueba de seguridad de aplicación web se enfoca solo en evaluar la seguridad de una aplicación web. El proceso involucra un análisis activo de la aplicación para cualquier debilidad, falla técnica o vulnerabilidad. Cualquier problema de seguridad que se encuentre será presentado al propietario del sistema, junto con una evaluación del impacto, una propuesta para mitigación o una solución técnica.

## ¿Qué es una Vulnerabilidad?

Una vulnerabilidad es una falla o debilidad en el diseño, implementación, operación o gestión de un sistema que podría ser explotada para comprometer los objetivos de seguridad del sistema.

## ¿Qué es una Amenaza?

Una amenaza es cualquier cosa (un atacante externo malicioso, un usuario interno, una inestabilidad del sistema, etc.) que puede dañar los activos propiedad de una aplicación (recursos de valor, como los datos en una base de datos o en el sistema de archivos) explotando una vulnerabilidad.

## ¿Qué es una Prueba?

Una prueba es una acción para demostrar que una aplicación cumple con los requisitos de seguridad de sus stakeholders.

## El Enfoque en Escribir esta Guía

El enfoque OWASP es abierto y colaborativo:

- Abierto: cada experto en seguridad puede participar con su experiencia en el proyecto. Todo es gratuito.
- Colaborativo: se realiza brainstorming antes de que los artículos sean escritos para que el equipo pueda compartir ideas y desarrollar una visión colectiva del proyecto. Eso significa consenso aproximado, una audiencia más amplia y participación aumentada.

Este enfoque tiende a resultar en una Metodología de Pruebas Definida que será:

- Consistente
- Reproducible
- Rigurosa
- Bajo control de calidad

Los problemas a abordar están completamente documentados y probados. Es importante usar varios métodos para probar todas las vulnerabilidades conocidas y documentar todas las actividades de pruebas de seguridad.

## ¿Qué Es la Metodología de Pruebas OWASP?

Las pruebas de seguridad nunca serán una ciencia exacta donde se pueda definir una lista completa de todos los posibles problemas que deberían probarse. De hecho, las pruebas de seguridad son solo una de las varias técnicas adecuadas para probar la seguridad de aplicaciones web bajo ciertas circunstancias. El objetivo de este proyecto es recopilar todas las posibles técnicas de pruebas, explicar estas técnicas y mantener la guía actualizada. La metodología de pruebas de seguridad de aplicaciones web OWASP se basa en el enfoque black box. El tester tiene poca o ninguna información sobre la aplicación a probar.

El modelo de pruebas consiste en:

- Tester: Quién realiza las actividades de pruebas
- Herramientas y metodología: El núcleo de este proyecto de Guía de Pruebas
- Aplicación: La caja negra a probar

Las pruebas pueden categorizarse como pasivas o activas:

### Pruebas Pasivas

Durante las pruebas pasivas, un tester intenta entender la lógica de la aplicación y explora la aplicación como un usuario final. Las herramientas pueden usarse para recopilación de información. Por ejemplo, un proxy HTTP(S) puede usarse para observar todas las peticiones y respuestas HTTP(S). Al final de esta fase, el tester debería generalmente entender todos los puntos de acceso y funcionalidad del sistema (ej., encabezados HTTP, parámetros, cookies, APIs, uso/tecnología/patrones, etc). La sección [Recopilación de Información](../01-Information_Gathering/README.md) explica cómo realizar pruebas pasivas.

Por ejemplo, un tester puede encontrar una página en la siguiente URL: `https://www.example.com/login/auth_form`

Esto puede indicar un formulario de autenticación donde la aplicación solicita un nombre de usuario y contraseña.

Los siguientes parámetros representan dos puntos de acceso a la aplicación: `https://www.example.com/appx?a=1&b=1`

En este caso, la aplicación tiene dos puntos de acceso (parámetros `a` y `b`). Todos los puntos de entrada encontrados en esta fase representan objetivos para pruebas. Mantener un registro del directorio o árbol de llamadas de la aplicación y todos los puntos de acceso puede ser útil durante las pruebas activas.

### Pruebas Activas

Durante las pruebas activas, un tester usa las metodologías descritas en las siguientes secciones.

El conjunto de pruebas activas se ha dividido en 12 categorías:

- Recopilación de Información
- Pruebas de Gestión de Configuración y Despliegue
- Pruebas de Gestión de Identidad
- Pruebas de Autenticación
- Pruebas de Autorización
- Pruebas de Gestión de Sesión
- Pruebas de Validación de Entrada
- Pruebas para Manejo de Errores
- Pruebas para Criptografía Débil
- Pruebas de Lógica de Negocio
- Pruebas del Lado del Cliente
- Pruebas de API