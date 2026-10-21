# Introducción y objetivos

Esta sección describe la metodología de pruebas de seguridad de aplicaciones web de OWASP y explica cómo buscar evidencia de vulnerabilidades en la aplicación causadas por deficiencias en los controles de seguridad identificados.

## ¿Qué son las pruebas de seguridad de aplicaciones web?

Una prueba de seguridad es un método para evaluar la seguridad de un sistema informático o una red mediante la validación y verificación metódicas de la eficacia de los controles de seguridad de la aplicación. Una prueba de seguridad de aplicaciones web se centra únicamente en evaluar la seguridad de una aplicación web. El proceso implica analizar activamente la aplicación para detectar debilidades, fallas técnicas o vulnerabilidades. Los problemas de seguridad que se encuentren se presentarán al propietario del sistema, junto con una evaluación del impacto y una propuesta de mitigación o una solución técnica.

## ¿Qué es una vulnerabilidad?

Una vulnerabilidad es una falla o debilidad en el diseño, la implementación, la operación o la gestión de un sistema que podría explotarse para comprometer los objetivos de seguridad del sistema.

## ¿Qué es una amenaza?

Una amenaza es cualquier cosa (un atacante externo malicioso, un usuario interno, una inestabilidad del sistema, etc.) que puede dañar los activos que pertenecen a una aplicación (recursos de valor, como los datos de una base de datos o del sistema de archivos) mediante la explotación de una vulnerabilidad.

## ¿Qué es una prueba?

Una prueba es una acción que demuestra que una aplicación cumple los requisitos de seguridad de sus partes interesadas.

## El enfoque para redactar esta guía

El enfoque de OWASP es abierto y colaborativo:

- Abierto: cualquier experto en seguridad puede participar en el proyecto y aportar su experiencia. Todo es gratuito.
- Colaborativo: se intercambian ideas antes de redactar los artículos para que el equipo pueda compartir propuestas y desarrollar una visión colectiva del proyecto. Esto permite alcanzar un consenso general, llegar a una audiencia más amplia y aumentar la participación.

Este enfoque suele dar lugar a una metodología de pruebas definida que será:

- Consistente
- Reproducible
- Rigurosa
- Sujeta a control de calidad

Los problemas que se deben abordar están completamente documentados y probados. Es importante utilizar diversos métodos para probar todas las vulnerabilidades conocidas y documentar todas las actividades de pruebas de seguridad.

## ¿Qué es la metodología de pruebas de OWASP?

Las pruebas de seguridad nunca serán una ciencia exacta que permita definir una lista completa de todos los posibles problemas que se deben probar. De hecho, las pruebas de seguridad son solo una de varias técnicas adecuadas para probar la seguridad de aplicaciones web en determinadas circunstancias. El objetivo de este proyecto es recopilar todas las técnicas de prueba posibles, explicarlas y mantener la guía actualizada. La metodología de pruebas de seguridad de aplicaciones web de OWASP se basa en el enfoque de caja negra. El evaluador tiene poca o ninguna información sobre la aplicación que se va a probar.

El modelo de pruebas consta de:

- Evaluador: quien realiza las actividades de prueba
- Herramientas y metodología: el núcleo de este proyecto de la Guía de Pruebas
- Aplicación: la caja negra que se va a probar

Las pruebas pueden clasificarse como pasivas o activas:

### Pruebas pasivas

Durante las pruebas pasivas, el evaluador intenta comprender la lógica de la aplicación y la explora como usuario final. Se pueden utilizar herramientas para recopilar información. Por ejemplo, se puede utilizar un proxy HTTP(S) para observar todas las solicitudes y respuestas HTTP(S). Al final de esta fase, el evaluador debería comprender, por lo general, todos los puntos de acceso y las funcionalidades del sistema (p. ej., encabezados HTTP, parámetros, cookies, API, uso de tecnologías y patrones, etc.). La sección [Recopilación de información](../01-Information_Gathering/README.md) explica cómo realizar pruebas pasivas.

Por ejemplo, un evaluador puede encontrar una página en la siguiente URL: `https://www.example.com/login/auth_form`

Esto puede indicar que hay un formulario de autenticación en el que la aplicación solicita un nombre de usuario y una contraseña.

Los siguientes parámetros representan dos puntos de acceso a la aplicación: `https://www.example.com/appx?a=1&b=1`

En este caso, la aplicación tiene dos puntos de acceso (los parámetros `a` y `b`). Todos los puntos de entrada encontrados en esta fase representan objetivos de prueba. Llevar un registro del directorio o árbol de llamadas de la aplicación y de todos los puntos de acceso puede ser útil durante las pruebas activas.

### Pruebas activas

Durante las pruebas activas, el evaluador utiliza las metodologías descritas en las secciones siguientes.

El conjunto de pruebas activas se ha dividido en 12 categorías:

- Recopilación de información
- Pruebas de gestión de la configuración y despliegue
- Pruebas de gestión de identidades
- Pruebas de autenticación
- Pruebas de autorización
- Pruebas de gestión de sesiones
- Pruebas de inyección
- Manejo de errores
- Criptografía débil
- Pruebas de lógica de negocio
- Pruebas del lado del cliente
- Pruebas de API