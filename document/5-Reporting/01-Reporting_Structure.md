# Informes

Realizar la parte técnica de la evaluación es solo la mitad del proceso de evaluación en su conjunto. El producto final es un informe bien redactado e informativo. Un informe debe ser fácil de entender y destacar todos los riesgos detectados durante la fase de evaluación. Debe resultar útil tanto para la dirección ejecutiva como para el personal técnico.

## Acerca de esta sección

Esta guía ofrece únicamente sugerencias sobre un posible enfoque para elaborar informes y no debe considerarse un conjunto de reglas estrictas que deban seguirse. Al considerar cualquiera de las recomendaciones siguientes, pregúntese siempre si mejoraría su informe.

Esta guía para elaborar informes se adapta mejor a los informes de consultoría. Puede ser excesiva para los informes internos o de programas de recompensas por errores.

Independientemente de quién sea el público destinatario, es recomendable proteger y cifrar el informe para garantizar que solo la parte receptora pueda utilizarlo.

Un buen informe ayuda al cliente a entender los hallazgos y destaca la calidad de las pruebas técnicas. La calidad de las pruebas técnicas es completamente irrelevante si el cliente no puede entender los hallazgos.

## 1. Introducción

### 1.1 Control de versiones

Establece los cambios del informe, presentados en su mayoría en una tabla como la siguiente.

| Versión | Descripción | Fecha | Autor |
|:-------:|-------------|------|--------|
| 1.0 | Informe inicial | DD/MM/YYYY | J. Doe |

### 1.2 Tabla de contenidos

Una página con la tabla de contenidos del documento.

### 1.3 El equipo

Una lista de los miembros del equipo que detalle su experiencia y sus cualificaciones.

### 1.4 Alcance

Los límites y las necesidades del trabajo acordados con la organización.

### 1.5 Limitaciones

Las limitaciones pueden ser:

- Áreas excluidas de las pruebas.
- Funcionalidad no operativa.
- Falta de cooperación.
- Falta de tiempo.
- Falta de acceso o de credenciales.

### 1.6 Cronograma

La duración del trabajo.

### 1.7 Descargo de responsabilidad

Puede incluir un descargo de responsabilidad sobre el servicio. Consulte siempre a un profesional legal para crear un documento jurídicamente vinculante.

El siguiente ejemplo es solo ilustrativo. No debe utilizarse tal cual y no constituye asesoramiento legal.

*Esta prueba es una evaluación realizada en un «momento determinado» y, como tal, es posible que el entorno haya cambiado desde que se llevó a cabo. No se garantiza que se hayan identificado todos los posibles problemas de seguridad ni que no se hayan descubierto nuevas vulnerabilidades desde que se realizaron las pruebas. Por lo tanto, este informe sirve como documento orientativo y no garantiza que represente plenamente los riesgos que amenazan los sistemas en cuestión.*

## 2. Resumen ejecutivo

Es como el discurso breve del informe y tiene como objetivo proporcionar a los ejecutivos:

- El objetivo de la prueba.
    - Describa la necesidad empresarial que motivó la prueba de seguridad.
    - Describa cómo las pruebas ayudaron a la organización a entender sus sistemas.
- Los hallazgos clave en un contexto empresarial, como posibles problemas de cumplimiento o daños a la reputación, entre otros. Céntrese en el impacto empresarial y deje de lado por ahora los detalles técnicos.
- Las recomendaciones estratégicas para que la empresa evite que los problemas vuelvan a ocurrir. Descríbalas en un contexto no técnico y deje de lado por ahora las recomendaciones técnicas específicas.

El resumen debe ser constructivo y significativo. Evite la jerga y las especulaciones negativas. Si se utilizan cifras, gráficos o ilustraciones, asegúrese de que ayuden a transmitir el mensaje con mayor claridad que el texto.

## 3. Hallazgos

Esta sección está dirigida al equipo técnico. Debe incluir toda la información necesaria para entender la vulnerabilidad, reproducirla y resolverla. La separación lógica puede ayudar a mejorar la legibilidad del informe. Por ejemplo, podría incluir secciones independientes tituladas «Acceso externo» y «Acceso interno».

Si se trata de una nueva prueba, podría crear una subsección que resuma los hallazgos de la prueba anterior, el estado actualizado de las vulnerabilidades identificadas previamente y cualquier referencia cruzada con la prueba actual.

### 3.1 Resumen de hallazgos

Una lista de los hallazgos con su nivel de riesgo. Se puede utilizar una tabla para facilitar su consulta a ambos equipos.

| ID de referencia | Título | Nivel de riesgo |
|:------------:|--------|------------|
| 1 | Bypass de autenticación de usuario | Alto |

### 3.2 Detalles de los hallazgos

Cada hallazgo debe detallarse con la siguiente información:

- ID de referencia, que se puede utilizar para la comunicación entre las partes y para las referencias cruzadas en el informe.
- El título de la vulnerabilidad, como «Bypass de autenticación de usuario».
- La probabilidad de que se explote el problema, basada en diversos factores, como:
    - Qué tan fácil es explotarlo.
    - Si existe código de exploit funcional.
    - El nivel de acceso necesario.
    - La motivación del atacante para explotarlo.
- El impacto de la vulnerabilidad en el sistema.
- El riesgo que representa la vulnerabilidad para la aplicación.
    - Algunos valores sugeridos son: Informativo, Bajo, Medio, Alto y Crítico. Asegúrese de detallar en un apéndice los valores que decida utilizar. Esto permite al lector entender cómo se determina cada puntuación.
    - En algunos trabajos se requiere una puntuación [CVSS](https://www.first.org/cvss/). Si no es obligatoria, a veces resulta útil incluirla y otras veces solo añade complejidad al informe.
- Una descripción detallada de la vulnerabilidad, de cómo explotarla y de los daños que podrían derivarse de su explotación. Se debe enmascarar cualquier dato posiblemente sensible, como contraseñas, información personal o datos de tarjetas de crédito.
- Pasos detallados para corregir la vulnerabilidad, posibles mejoras que podrían ayudar a fortalecer la postura de seguridad y prácticas de seguridad que no se hayan implementado.
- Recursos adicionales que puedan ayudar al lector a entender la vulnerabilidad, como una imagen, un video, un CVE o una guía externa.

Dé a esta sección el formato que mejor permita transmitir el mensaje.

Asegúrese siempre de que las descripciones proporcionen suficiente información para que el ingeniero que lea el informe pueda actuar en consecuencia. Explique el hallazgo exhaustivamente y proporcione todos los detalles técnicos que puedan ser necesarios para corregirlo.

## Factores humanos

Los factores humanos influyen considerablemente en la eficacia de los controles técnicos de seguridad. Los atacantes suelen explotar a las personas (mediante phishing, ingeniería social, reutilización de credenciales y procesos de recuperación social) en lugar de aprovechar únicamente las debilidades técnicas. En sus informes, los evaluadores deben señalar los problemas relacionados con los usuarios y los procesos, además de los hallazgos técnicos, y recomendar medidas correctivas que combinen controles técnicos, mejoras en la experiencia de usuario y cambios específicos en los procesos o la capacitación.

Lista de verificación (aspectos que se deben buscar e informar):

- Phishing y filtración de información: señale los textos de la interfaz de usuario, los mensajes de error o las páginas que filtren información útil para los atacantes (enumeración de cuentas, errores demasiado detallados o detalles de los flujos de recuperación).
- Experiencia de usuario de autenticación y recuperación: revise los flujos de recuperación (restablecimiento de contraseña, recuperación de cuentas, códigos de respaldo y alternativas por SMS) para comprobar si se pueden abusar de ellos o si fomentan soluciones alternativas inseguras.
- Flujos de 2FA y alternativas: verifique si las opciones de segundo factor se implementan de forma segura (sin vías alternativas débiles) y si la experiencia de usuario lleva a los usuarios a desactivar las protecciones.
- Exposición a la ingeniería social: identifique funciones o procesos (soporte o mesa de ayuda, enlaces de perfil no validados y cuentas compartidas) de los que podría abusar un atacante que se haga pasar por un usuario legítimo.
- Gestión de sesiones y credenciales: busque sesiones de larga duración, implementaciones de «recordarme» o exposición de tokens que aumenten el riesgo cuando se combinan con ingeniería social o malware.
- Medidas prácticas de mitigación: recomiende primero controles técnicos (limitación de frecuencia, bloqueo de cuentas, 2FA segura, flujos de recuperación reforzados y mensajes de error con información mínima) y compleméntelos con medidas específicas para los procesos (procedimientos de verificación del soporte) y orientaciones breves y prácticas para los usuarios cuando corresponda.

Notas para el informe:

Incluya ejemplos que no sean sensibles o capturas de pantalla redactadas para mostrar el problema y el contexto que ve el usuario.

Sugiera medidas correctivas concretas y priorizadas que los equipos de ingeniería y producto puedan aplicar.

Cuando corresponda, identifique quién dentro de la organización debería hacerse cargo de la corrección (por ejemplo, producto, soporte o seguridad) y las medidas de mitigación a corto plazo (indicadores de funcionalidad, límites de frecuencia temporales).

### Artefactos de prueba reproducibles

Como parte del informe, los evaluadores pueden incluir artefactos de prueba reproducibles para ayudar a los desarrolladores a verificar las correcciones.

Algunos ejemplos son:

- Comandos curl que demuestren la solicitud
- Prueba de concepto (PoC), como formularios CSRF
- Archivos HAR que capturen flujos de solicitud y respuesta
- Scripts de automatización sencillos (por ejemplo, Selenium, Puppeteer)

Estos artefactos pueden ayudar a los desarrolladores a validar y volver a probar las vulnerabilidades rápidamente.

## Apéndices

Se pueden añadir varios apéndices, como:

- La metodología de prueba utilizada.
- Explicaciones de la severidad y la clasificación de riesgos.
- Resultados pertinentes de las herramientas utilizadas.
    - Asegúrese de depurar los resultados y no limitarse a volcarlos sin procesar.
- Una lista de verificación de todas las pruebas realizadas, como las [listas de verificación WSTG](https://github.com/OWASP/wstg/tree/master/checklists). Se pueden incluir como anexos al informe.

## Referencias

Esta sección no forma parte del formato de informe sugerido. Los enlaces siguientes ofrecen más orientación para redactar informes.

- [NIST SP 800-63: Directrices sobre identidad digital](https://pages.nist.gov/800-63-4/) - comprobación de identidad y autenticación.
- [Colección de hojas de referencia de OWASP](https://cheatsheetseries.owasp.org) - considere enlazar hojas de referencia pertinentes sobre phishing, autenticación y prevención de la ingeniería social.
- [SANS: Consejos para crear un informe sólido de evaluación de ciberseguridad](https://www.sans.org/blog/tips-for-creating-a-strong-cybersecurity-assessment-report/)
- [SANS: Redacción de un informe de pruebas de penetración](https://www.sans.org/reading-room/whitepapers/bestprac/paper/33343)
- [Infosec Institute: El arte de redactar informes de pruebas de penetración](https://www.infosecinstitute.com/resources/penetration-testing/writing-penetration-testing-reports/)
- [Dummies: Cómo estructurar un informe de pruebas de penetración](https://www.dummies.com/article/technology/cybersecurity/how-to-structure-a-pen-test-report-270933/)
- [Rhino Security Labs: Cuatro aspectos que todo informe de pruebas de penetración debería incluir](https://rhinosecuritylabs.com/penetration-testing/four-things-every-penetration-test-report/)