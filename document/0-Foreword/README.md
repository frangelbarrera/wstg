# Prefacio

El problema del software inseguro es quizás el desafío técnico más importante de nuestro tiempo. El aumento dramático de aplicaciones web habilitando negocios, redes sociales, etc. ha solo compuesto los requisitos para establecer un enfoque robusto para escribir y asegurar nuestro internet, aplicaciones web y datos.

En el Proyecto de Seguridad de Aplicaciones Web Abierto Mundial® (OWASP®), estamos intentando hacer del mundo un lugar donde el software inseguro sea la anomalía, no la norma. La Guía de Pruebas de OWASP tiene un papel importante que jugar en resolver este serio problema. Es vitalmente importante que nuestro enfoque para probar software en busca de problemas de seguridad se base en los principios de ingeniería y ciencia. Necesitamos un enfoque consistente, repetible y definido para probar aplicaciones web. Un mundo sin algunos estándares mínimos en términos de ingeniería y tecnología es un mundo en caos.

Va sin decir que no se puede construir una aplicación segura sin realizar pruebas de seguridad en ella. Las pruebas son parte de un enfoque más amplio para construir un sistema seguro. Muchas organizaciones de desarrollo de software no incluyen pruebas de seguridad como parte de su proceso estándar de desarrollo de software. Lo que es aún peor es que muchos proveedores de seguridad entregan pruebas con varios grados de calidad y rigor.

Las pruebas de seguridad, por sí mismas, no son una medida particularmente buena de cuán segura es una aplicación, porque hay un número infinito de maneras en que un atacante podría ser capaz de hacer que una aplicación falle, y simplemente no es posible probarlas todas. No podemos hackearnos a seguridad ya que solo tenemos un tiempo limitado para probar y defender donde un atacante no tiene tales restricciones.

En conjunto con otros proyectos de OWASP tales como la Guía de Revisión de Código, la Guía de Desarrollo y herramientas tales como [ZAP](https://www.zaproxy.org/), este es un gran comienzo hacia la construcción y mantenimiento de aplicaciones seguras. Esta Guía de Pruebas te mostrará cómo verificar la seguridad de tu aplicación en ejecución. Recomiendo altamente usar estas guías como parte de tus iniciativas de seguridad de aplicaciones.

## ¿Por qué OWASP?

Crear una guía como esta es una empresa enorme, requiriendo la experiencia de cientos de personas alrededor del mundo. Hay muchas maneras diferentes de probar en busca de fallos de seguridad y esta guía captura el consenso de los principales expertos sobre cómo realizar esta prueba rápidamente, con precisión y eficiencia. OWASP da a personas de seguridad con ideas afines la capacidad de trabajar juntas y formar un enfoque de práctica líder a un problema de seguridad.

La importancia de tener esta guía disponible de manera completamente libre y abierta es importante para la misión de la fundación. Da a cualquiera la capacidad de entender las técnicas usadas para probar en busca de problemas comunes de seguridad. La seguridad no debería ser un arte oscuro o secreto cerrado que solo unos pocos puedan practicar. Debería estar abierta a todos y no exclusiva para profesionales de seguridad sino también QA, Desarrolladores y Gerentes Técnicos. El proyecto para construir esta guía mantiene esta experiencia en las manos de las personas que la necesitan - tú, yo y cualquiera que esté involucrado en construir software.

Esta guía debe hacer su camino a las manos de desarrolladores y testers de software. No hay suficientes expertos de seguridad de aplicaciones en el mundo para hacer una abolladura significativa en el problema general. La responsabilidad inicial para la seguridad de aplicaciones debe caer sobre los hombros de los desarrolladores porque ellos escriben el código. No debería ser una sorpresa que los desarrolladores no estén produciendo código seguro si no están probando para ello o considerando los tipos de bugs que introducen vulnerabilidad.

Mantener esta información actualizada es un aspecto crítico de este proyecto de guía. Al adoptar el enfoque wiki, la comunidad de OWASP puede evolucionar y expandir la información en esta guía para mantener el ritmo con el panorama de amenazas de seguridad de aplicaciones de movimiento rápido.

Esta Guía es un gran testamento a la pasión y energía que nuestros miembros y voluntarios del proyecto tienen por este tema. Ciertamente ayudará a cambiar el mundo una línea de código a la vez.

## Adaptación y Priorización

Deberías adoptar esta guía en tu organización. Podrías necesitar adaptar la información para que coincida con las tecnologías, procesos y estructura organizacional de tu organización.

En general hay varios roles diferentes dentro de las organizaciones que podrían usar esta guía:

- Los desarrolladores deberían usar esta guía para asegurar que están produciendo código seguro. Estas pruebas deberían ser parte de los procedimientos normales de pruebas de código y unitarias.
- Los testers de software y QA deberían usar esta guía para expandir el conjunto de casos de prueba que aplican a aplicaciones. Atrapar estas vulnerabilidades temprano ahorra considerable tiempo y esfuerzo más tarde.
- Los especialistas de seguridad deberían usar esta guía en combinación con otras técnicas como una manera de verificar que no se hayan perdido agujeros de seguridad en una aplicación.
- Los Gerentes de Proyecto deberían considerar la razón por la que esta guía existe y que los problemas de seguridad se manifiestan vía bugs en código y diseño.

Lo más importante a recordar al realizar pruebas de seguridad es repriorizar continuamente. Hay maneras infinitas en que una aplicación podría fallar, y las organizaciones siempre tienen tiempo y recursos limitados de prueba. Asegurarse de que el tiempo y los recursos se gasten sabiamente. Intentar enfocarse en los agujeros de seguridad que son un riesgo real para tu negocio. Intentar contextualizar el riesgo en términos de la aplicación y sus casos de uso.

Esta guía es mejor vista como un conjunto de técnicas que puedes usar para encontrar diferentes tipos de agujeros de seguridad. Pero no todas las técnicas son igualmente importantes. Intentar evitar usar la guía como una lista de verificación, nuevas vulnerabilidades siempre se están manifestando y ninguna guía puede ser una lista exhaustiva de "cosas para probar", sino más bien un gran lugar para comenzar.

## El Papel de Herramientas Automatizadas

Hay un número de compañías vendiendo herramientas automatizadas de análisis y prueba de seguridad. Recordar las limitaciones de estas herramientas para que puedas usarlas para lo que son buenas. Como Michael Howard lo puso en la Conferencia OWASP AppSec 2006 en Seattle, "¡Las herramientas no hacen el software seguro! Ellas ayudan a escalar el proceso y ayudan a imponer políticas."

Lo más importante, estas herramientas son genéricas - lo que significa que no están diseñadas para tu código personalizado, sino para aplicaciones en general. Eso significa que mientras pueden encontrar algunos problemas genéricos, no tienen suficiente conocimiento de tu aplicación para permitirles detectar la mayoría de los fallos. En mi experiencia, los problemas de seguridad más serios son los que no son genéricos, sino profundamente entrelazados en tu lógica de negocio y diseño de aplicación personalizado.

Estas herramientas también pueden ser muy útiles, ya que encuentran muchos problemas potenciales. Mientras que ejecutar las herramientas no toma mucho tiempo, cada uno de los problemas potenciales toma tiempo para investigar y verificar. Si la meta es encontrar y eliminar los fallos más serios lo más rápido posible, considerar si tu tiempo se gasta mejor con herramientas automatizadas o con las técnicas descritas en esta guía. Aún así, estas herramientas son ciertamente parte de un programa de seguridad de aplicaciones bien equilibrado. Usadas sabiamente, pueden apoyar tus procesos generales para producir código más seguro.

## Llamado a la Acción

Si estás construyendo, diseñando o probando software, te alentamos fuertemente a familiarizarte con la guía de pruebas de seguridad en este documento. Es una gran hoja de ruta para probar los problemas más comunes que las aplicaciones enfrentan hoy, pero no es exhaustiva. Si encuentras errores, por favor añade una nota a la página de discusión o haz el cambio tú mismo. Estarás ayudando a miles de otros que usan esta guía.

Por favor considera [unirte a nosotros](https://owasp.org/membership/) como miembro individual o corporativo para que podamos continuar produciendo materiales como esta guía de pruebas y todos los otros grandes proyectos en OWASP.

Gracias a todos los contribuyentes pasados y futuros de esta guía, tu trabajo ayudará a hacer las aplicaciones en todo el mundo más seguras.

Open Worldwide Application Security Project y OWASP son marcas registradas de la Fundación OWASP, Inc.
