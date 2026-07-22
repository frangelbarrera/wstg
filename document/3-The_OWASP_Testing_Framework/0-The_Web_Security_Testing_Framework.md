# El Marco de Pruebas de Seguridad Web

## Resumen

Esta sección describe un marco de pruebas típico que puede desarrollarse dentro de una organización. Puede verse como un marco de referencia compuesto por técnicas y tareas apropiadas en varias fases del ciclo de vida de desarrollo de software (SDLC). Las compañías y equipos de proyecto pueden usar este modelo para desarrollar su propio marco de pruebas y para definir el alcance de servicios de pruebas de proveedores. Este marco no debe verse como prescriptivo, sino como un enfoque flexible que puede extenderse y moldearse para adaptarse al proceso de desarrollo y cultura de una organización.

Esta sección tiene como objetivo ayudar a las organizaciones a construir un proceso de pruebas estratégico completo, y no está dirigida a consultores o contratistas que tienden a involucrarse en áreas más tácticas y específicas de pruebas.

Es crítico entender por qué construir un marco de pruebas de extremo a extremo es crucial para evaluar y mejorar la seguridad del software. En *Writing Secure Code*, Howard y LeBlanc notan que emitir un boletín de seguridad cuesta a Microsoft al menos $100,000, y cuesta a sus clientes colectivamente mucho más que eso implementar los parches de seguridad. También notan que el sitio web de CyberCrime del gobierno de EE.UU. detalla casos criminales recientes y la pérdida a las organizaciones. Las pérdidas típicas exceden los USD $100,000.

Con economías como esta, no es de extrañar que los proveedores de software se muevan de realizar únicamente pruebas de seguridad de caja negra, que solo pueden realizarse en aplicaciones que ya han sido desarrolladas, a concentrarse en pruebas en los ciclos tempranos de desarrollo de aplicaciones, como durante definición, diseño y desarrollo.

Muchos practicantes de seguridad aún ven las pruebas de seguridad en el ámbito de las pruebas de penetración. Como se discutió en el capítulo anterior, mientras las pruebas de penetración tienen un rol que desempeñar, generalmente son ineficientes para encontrar bugs y dependen excesivamente de la habilidad del probador. Debería considerarse solo como una técnica de implementación, o para aumentar la conciencia de problemas de producción. Para mejorar la seguridad de las aplicaciones, la calidad de seguridad del software debe mejorarse. Eso significa probar la seguridad durante las etapas de definición, diseño, desarrollo, despliegue y mantenimiento, y no depender de la costosa estrategia de esperar hasta que el código esté completamente construido.

Como se discutió en la introducción de este documento, hay muchas metodologías de desarrollo, como el Proceso Unificado Racional, desarrollo eXtreme y Agile, y metodologías tradicionales de cascada. La intención de esta guía es no sugerir una metodología de desarrollo particular, ni proporcionar orientación específica que se adhiera a cualquier metodología particular. En cambio, estamos presentando un modelo de desarrollo genérico, y el lector debería seguirlo de acuerdo con el proceso de su compañía.

Este marco de pruebas consiste en actividades que deberían tener lugar:

- Antes de que comience el desarrollo,
- Durante definición y diseño,
- Durante desarrollo,
- Durante despliegue, y
- Durante mantenimiento y operaciones.

## Fase 1 Antes de que Comience el Desarrollo

### Fase 1.1 Definir un SDLC

Antes de que comience el desarrollo de la aplicación, debe definirse un SDLC adecuado donde la seguridad sea inherente en cada etapa.

### Fase 1.2 Revisar Políticas y Estándares

Asegurar que haya políticas, estándares y documentación apropiados en su lugar. La documentación es extremadamente importante ya que da a los equipos de desarrollo directrices y políticas que pueden seguir. La gente solo puede hacer lo correcto si sabe cuál es lo correcto.

Si la aplicación va a desarrollarse en Java, es esencial que haya un estándar de codificación segura en Java. Si la aplicación va a usar criptografía, es esencial que haya un estándar de criptografía. Ninguna política o estándar puede cubrir cada situación que enfrentará el equipo de desarrollo. Al documentar los problemas comunes y predecibles, habrá menos decisiones que necesiten tomarse durante el proceso de desarrollo.

### Fase 1.3 Desarrollar Criterios de Medición y Métricas y Asegurar Trazabilidad

Antes de que comience el desarrollo, planificar el programa de medición. Al definir criterios que necesitan medirse, proporciona visibilidad en defectos tanto en el proceso como en el producto. Es esencial definir las métricas antes de que comience el desarrollo, ya que puede haber necesidad de modificar el proceso para capturar los datos.

## Fase 2 Durante Definición y Diseño

### Fase 2.1 Revisar Requisitos de Seguridad

Los requisitos de seguridad definen cómo funciona una aplicación desde una perspectiva de seguridad. Es esencial que los requisitos de seguridad sean probados. Probar en este caso significa probar las suposiciones que se hacen en los requisitos y probar para ver si hay brechas en las definiciones de requisitos.

Por ejemplo, si hay un requisito de seguridad que establece que los usuarios deben estar registrados antes de que puedan acceder a la sección de documentos técnicos de un sitio web, ¿significa esto que el usuario debe estar registrado con el sistema o debería el usuario estar autenticado? Asegurar que los requisitos sean lo más inequívocos posible.

Al buscar brechas en requisitos, considerar mecanismos de seguridad como:

- Gestión de usuarios
- Autenticación
- Autorización
- Confidencialidad de datos
- Integridad
- Responsabilidad
- Gestión de sesiones
- Seguridad de transporte
- Segregación de sistema en capas
- Cumplimiento legislativo y de estándares (incluyendo privacidad, gobierno y estándares de la industria)

### Fase 2.2 Revisar Diseño y Arquitectura

Las aplicaciones deberían tener un diseño y arquitectura documentados. Esta documentación puede incluir modelos, documentos textuales y otros artefactos similares. Es esencial probar estos artefactos para asegurar que el diseño y arquitectura apliquen el nivel apropiado de seguridad como definido en los requisitos.

Identificar fallas de seguridad en la fase de diseño no solo es uno de los lugares más costo-eficientes para identificar fallas, sino que puede ser uno de los lugares más efectivos para hacer cambios. Por ejemplo, si se identifica que el diseño llama a decisiones de autorización a hacerse en múltiples lugares, puede ser apropiado considerar un componente de autorización central. Si la aplicación está realizando validación de datos en múltiples lugares, puede ser apropiado desarrollar un marco de validación central (es decir, arreglar validación de entrada en un lugar, en lugar de en cientos de lugares, es mucho más barato).

Si se descubren debilidades, deberían darse al arquitecto del sistema para enfoques alternativos.

### Fase 2.3 Crear y Revisar Modelos UML

Una vez que el diseño y arquitectura esté completo, construir modelos de Lenguaje de Modelado Unificado (UML) que describan cómo funciona la aplicación. En algunos casos, estos pueden ya estar disponibles. Usar estos modelos para confirmar con los diseñadores de sistemas una comprensión exacta de cómo funciona la aplicación. Si se descubren debilidades, deberían darse al arquitecto del sistema para enfoques alternativos.

### Fase 2.4 Crear y Revisar Modelos de Amenazas

Armados con revisiones de diseño y arquitectura y los modelos UML explicando exactamente cómo funciona el sistema, emprender un ejercicio de modelado de amenazas. Desarrollar escenarios de amenazas realistas. Analizar el diseño y arquitectura para asegurar que estas amenazas han sido mitigadas, aceptadas por el negocio, o asignadas a un tercero, como una firma de seguros. Cuando las amenazas identificadas no tienen estrategias de mitigación, revisar el diseño y arquitectura con el arquitecto del sistema para modificar el diseño.

## Fase 3 Durante Desarrollo

Teóricamente, el desarrollo es la implementación de un diseño. Sin embargo, en el mundo real, muchas decisiones de diseño se hacen durante el desarrollo de código. Estas son a menudo decisiones más pequeñas que fueron demasiado detalladas para describirse en el diseño, o problemas donde no se ofreció orientación de política o estándar. Si el diseño y arquitectura no fueron adecuados, el desarrollador se enfrentará con muchas decisiones. Si hubo políticas y estándares insuficientes, el desarrollador se enfrentará con aún más decisiones.

### Fase 3.1 Revisión de Código

El equipo de seguridad debería realizar una revisión de código con los desarrolladores, y en algunos casos, los arquitectos del sistema. Una revisión de código es una mirada de alto nivel al código durante la cual los desarrolladores pueden explicar la lógica y flujo del código implementado. Permite al equipo de revisión de código obtener una comprensión general del código, y permite a los desarrolladores explicar por qué ciertas cosas fueron desarrolladas de la manera en que fueron.

El propósito no es realizar una revisión de código, sino entender a un alto nivel el flujo, el diseño y la estructura del código que compone la aplicación.

### Fase 3.2 Revisiones de Código

Armados con una buena comprensión de cómo está estructurado el código y por qué ciertas cosas fueron codificadas de la manera en que fueron, el probador puede ahora examinar el código real en busca de defectos de seguridad.

Las revisiones de código estático validan el código contra un conjunto de listas de verificación, incluyendo:

- Requisitos de negocio para disponibilidad, confidencialidad e integridad;
- Guía OWASP o listas de verificación Top 10 para exposiciones técnicas (dependiendo de la profundidad de la revisión);
- Problemas específicos relacionados con el lenguaje o marco en uso, como el papel Scarlet para PHP o [listas de verificación de codificación segura de Microsoft para ASP.NET](https://msdn.microsoft.com/en-us/library/ff648269.aspx); y
- Cualquier requisito específico de la industria, como Sarbanes-Oxley 404, COPPA, ISO/IEC 27002, APRA, HIPAA, directrices de Visa Merchant, u otros regímenes regulatorios.

En términos de retorno sobre recursos invertidos (mayormente tiempo), las revisiones de código estático producen retornos de calidad mucho más altos que cualquier otro método de revisión de seguridad y dependen menos de la habilidad del revisor. Sin embargo, no son una bala de plata y necesitan considerarse cuidadosamente dentro de un régimen de pruebas de espectro completo.

Para más detalles sobre listas de verificación OWASP, por favor referirse a la última edición del [OWASP Top 10](https://owasp.org/www-project-top-ten/).

## Fase 4 Durante Despliegue

### Fase 4.1 Pruebas de Penetración de Aplicación

Habiendo probado los requisitos, analizado el diseño y realizado revisión de código, podría asumirse que todos los problemas han sido capturados. Esperemos que este sea el caso, pero probar la penetración de la aplicación después de que ha sido desplegada proporciona una verificación adicional para asegurar que nada ha sido perdido.

### Fase 4.2 Pruebas de Gestión de Configuración

La prueba de penetración de la aplicación debería incluir un examen de cómo la infraestructura fue desplegada y asegurada. Es importante revisar aspectos de configuración, no importa cuán pequeños, para asegurar que ninguno se deja en una configuración predeterminada que puede ser vulnerable a explotación.

## Fase 5 Durante Mantenimiento y Operaciones

### Fase 5.1 Realizar Revisiones de Gestión Operacional

Necesita haber un proceso en lugar que detalle cómo se gestiona el lado operacional tanto de la aplicación como de la infraestructura.

### Fase 5.2 Realizar Verificaciones de Salud Periódicas

Deberían realizarse verificaciones de salud mensuales o trimestrales tanto en la aplicación como en la infraestructura para asegurar que no se han introducido nuevos riesgos de seguridad y que el nivel de seguridad sigue intacto.

### Fase 5.3 Asegurar Verificación de Cambios

Después de que cada cambio ha sido aprobado y probado en el entorno QA y desplegado en el entorno de producción, es vital que el cambio sea verificado para asegurar que el nivel de seguridad no ha sido afectado por el cambio. Esto debería integrarse en el proceso de gestión de cambios.

## Un Flujo de Trabajo Típico de Pruebas SDLC

La siguiente figura muestra un Flujo de Trabajo Típico de Pruebas SDLC.

![Flujo de Trabajo Típico de Pruebas SDLC](images/Typical_SDLC_Testing_Workflow.gif)\
*Figura 3-1: Flujo de trabajo típico de pruebas SDLC*