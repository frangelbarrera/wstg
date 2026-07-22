# Estructura de Reportes

Realizar el lado técnico de la evaluación es solo la mitad del proceso general de evaluación. El producto final es la producción de un informe bien escrito e informativo. Un informe debe ser fácil de entender y debe resaltar todos los riesgos encontrados durante la fase de evaluación. El informe debe atraer tanto a la gerencia ejecutiva como al personal técnico.

## Acerca de Esta Sección

Esta guía proporciona solo sugerencias sobre un posible enfoque para reportar, y no debe tratarse como reglas estrictas que deben seguirse. Al considerar cualquiera de las recomendaciones a continuación, siempre pregúntese si la recomendación mejoraría su informe.

Esta guía para reportar es más adecuada para informes basados en consultoría. Puede ser excesiva para informes internos o de recompensas por errores.

Independientemente de la audiencia, es aconsejable asegurar el informe y encriptarlo para garantizar que solo la parte receptora pueda usarlo.

Un buen informe ayuda a su cliente a entender sus hallazgos y resalta la calidad de sus pruebas técnicas. La calidad de las pruebas técnicas es completamente irrelevante si el cliente no puede entender sus hallazgos.

## 1. Introducción

### 1.1 Control de Versiones

Establece cambios en el informe, mayormente presentados en un formato de tabla como el siguiente.

| Versión | Descripción | Fecha | Autor |
|:-------:|-------------|------|--------|
| 1.0 | Informe inicial | DD/MM/YYYY | J. Doe |

### 1.2 Tabla de Contenidos

Una página de tabla de contenidos para el documento.

### 1.3 El Equipo

Una lista de los miembros del equipo detallando su experiencia y calificaciones.

### 1.4 Alcance

Los límites y las necesidades del compromiso acordados con la organización.

### 1.5 Limitaciones

Las limitaciones pueden ser:

- Áreas fuera de límites en relación con las pruebas.
- Funcionalidad rota.
- Falta de cooperación.
- Falta de tiempo.
- Falta de acceso o credenciales.

### 1.6 Cronograma

La duración del compromiso.

### 1.7 Descargo de Responsabilidad

Puede desear proporcionar un descargo de responsabilidad para su servicio. Siempre consulte a un profesional legal para crear un documento legalmente vinculante.

El siguiente ejemplo es solo para fines ilustrativos. No debe usarse tal cual y no constituye asesoramiento legal.

*Esta prueba es una evaluación de "punto en el tiempo" y como tal, el entorno podría haber cambiado desde que se ejecutó la prueba. No hay garantía de que se hayan identificado todos los posibles problemas de seguridad, y que se puedan haber descubierto nuevas vulnerabilidades desde que se ejecutaron las pruebas. Como tal, este informe sirve como un documento guía y no una garantía de que el informe proporcione una representación completa de los riesgos que amenazan los sistemas en cuestión.*

## 2. Resumen Ejecutivo

Esto es como el discurso de elevador del informe, tiene como objetivo proporcionar a los ejecutivos con:

- El objetivo de la prueba.
    - Describa la necesidad comercial detrás de la prueba de seguridad.
    - Describa cómo las pruebas ayudaron a la organización a entender sus sistemas.
- Hallazgos clave en un contexto comercial, como posibles problemas de cumplimiento, daño a la reputación, etc. Enfóquese en el impacto comercial y deje los detalles técnicos por ahora.
- Las recomendaciones estratégicas sobre cómo el negocio puede evitar que los problemas vuelvan a suceder. Describa estas en un contexto no técnico y deje las recomendaciones técnicas específicas por ahora.

El resumen debe ser constructivo y significativo. Evite jerga y especulación negativa. Si se usan figuras, gráficos o ilustraciones, asegúrese de que ayuden a entregar un mensaje de manera más clara que el texto lo haría.

## 3. Hallazgos

Esta sección está dirigida al equipo técnico. Debe incluir toda la información necesaria para entender la vulnerabilidad, replicarla y resolverla. La separación lógica puede ayudar a mejorar la legibilidad del informe. Por ejemplo, podría tener secciones separadas tituladas "Acceso Externo" y "Acceso Interno".

Si esto es una re-prueba, podría crear una subsección que resuma los hallazgos de la prueba anterior, el estado actualizado de vulnerabilidades previamente identificadas, y cualquier referencia cruzada con la prueba actual.

### 3.1 Resumen de Hallazgos

Una lista de los hallazgos con su nivel de riesgo. Una tabla puede usarse para facilitar el uso por ambos equipos.

| ID Ref. | Título | Nivel de Riesgo |
|:------------:|--------|-----------------|
| 1 | Bypass de Autenticación de Usuario | Alto |

### 3.2 Detalles de Hallazgos

Cada hallazgo debe detallarse con la siguiente información:

- ID de Referencia, que puede usarse para comunicación entre partes y para referencias cruzadas en el informe.
- El título de la vulnerabilidad, como "Bypass de Autenticación de Usuario".
- La probabilidad o explotabilidad del problema, basada en varios factores como:
    - Qué tan fácil es explotar.
    - Si hay código de exploit funcional para ello.
    - El nivel de acceso requerido.
    - Motivación del atacante para explotarlo.
- El impacto de la vulnerabilidad en el sistema.
- Riesgo de la vulnerabilidad en la aplicación.
    - Algunos valores sugeridos son: Informativo, Bajo, Medio, Alto y Crítico. Asegúrese de detallar los valores que decide usar en un apéndice. Esto permite al lector entender cómo se determina cada puntuación.
    - En ciertos compromisos se requiere tener una puntuación [CVSS](https://www.first.org/cvss/). Si no se requiere, a veces es bueno tenerla, y otras veces solo agrega complejidad al informe.
- Descripción detallada de qué es la vulnerabilidad, cómo explotarla, y el daño que puede resultar de su explotación. Cualquier dato posiblemente sensible debe enmascararse, por ejemplo, contraseñas, información personal o detalles de tarjetas de crédito.
- Pasos detallados sobre cómo remediar la vulnerabilidad, posibles mejoras que podrían ayudar a fortalecer la postura de seguridad, y prácticas de seguridad faltantes.
- Recursos adicionales que podrían ayudar al lector a entender la vulnerabilidad, como una imagen, un video, un CVE, una guía externa, etc.

Formatee esta sección de manera que mejor entregue su mensaje.

Siempre asegúrese de que sus descripciones proporcionen suficiente información para que el ingeniero que lee este informe tome acción basada en él. Explique el hallazgo exhaustivamente y proporcione tanto detalle técnico como sea necesario para remediarlo.

## Apéndices

Múltiples apéndices pueden agregarse, como:

- Metodología de prueba usada.
- Explicaciones de severidad y calificación de riesgo.
- Salida relevante de herramientas usadas.
    - Asegúrese de limpiar la salida y no solo volcarla.
- Una lista de verificación de todas las pruebas realizadas, como las [listas de verificación WSTG](https://github.com/OWASP/wstg/tree/master/checklists). Estas pueden proporcionarse como adjuntos al informe.

## Referencias

Esta sección no es parte del formato de informe sugerido. Los enlaces a continuación proporcionan más guía para escribir sus informes.

- [SANS: Consejos para Crear un Informe de Evaluación de Ciberseguridad Fuerte](https://www.sans.org/blog/tips-for-creating-a-strong-cybersecurity-assessment-report/)
- [SANS: Escribiendo un Informe de Pruebas de Penetración](https://www.sans.org/reading-room/whitepapers/bestprac/paper/33343)
- [Instituto Infosec: El Arte de Escribir Informes de Pruebas de Penetración](https://resources.infosecinstitute.com/topic/writing-penetration-testing-reports/)
- [Dummies: Cómo Estructurar un Informe de Prueba de Pen](https://www.dummies.com/computers/macs/security/how-to-structure-a-pen-test-report/)
- [Rhino Security Labs: Cuatro Cosas que Todo Informe de Pruebas de Penetración Debe Tener](https://rhinosecuritylabs.com/penetration-testing/four-things-every-penetration-test-report/)