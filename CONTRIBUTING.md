# Contribuyendo a la Guía de Pruebas

¡Gracias por considerar contribuir a la Guía de Pruebas de Seguridad Web (WSTG)!

Aquí hay algunas formas en que puedes hacer una contribución útil. La [Guía de Código Abierto para por qué y cómo contribuir](https://opensource.guide/how-to-contribute/) también es un buen recurso. Necesitarás una [cuenta de GitHub](https://help.github.com/en/github/getting-started-with-github/signing-up-for-a-new-github-account) para ayudar.

- [Conviértete en Autor](#conviértete-en-autor)
- [Conviértete en Revisor o Editor](#conviértete-en-revisor-o-editor)
  - [Revisión Técnica](#revisión-técnica)
  - [Revisión Editorial](#revisión-editorial)
- [Cómo Abrir un Issue](#cómo-abrir-un-issue)
- [Cómo Enviar una Solicitud de Extracción](#cómo-enviar-una-solicitud-de-extracción)
- [Cómo Configurar Tu Entorno de Contribuidor](#cómo-configurar-tu-entorno-de-contribuidor)
- [Contribuyendo con Codespaces](#contribuyendo-con-codespaces)

## Conviértete en Autor

Este proyecto no sería posible sin las contribuciones de escritores en la comunidad de seguridad! Nuestros autores ayudan a mantener el WSTG relevante y útil para todos.

Ya sea que estés enviando una nueva sección o agregando información a una existente, por favor sigue el [ejemplo de plantilla](template/999-Foo_Testing/1-Testing_for_a_Cat_in_a_Box.md). Las [secciones de plantilla se explican aquí](template/999-Foo_Testing/2-Template_Explanation.md).

Al enviar tu [solicitud de extracción](#cómo-enviar-una-solicitud-de-extracción), los autores deberían vincular contribuciones a un issue:

1. Abre un [issue de Agregar Nuevo Contenido](https://github.com/OWASP/wstg/issues/new?assignees=&labels=New&template=new-content.md&title=), o elige un [issue de nuevo contenido no asignado](https://github.com/OWASP/wstg/issues?q=is%3Aopen+is%3Aissue+label%3ANew+no%3Aassignee) y pide ser asignado a él.
2. Crea y cambia a una nueva rama local con el nombre `new-<número de issue>`. Por ejemplo, `git checkout -b new-164`.

## Conviértete en Revisor o Editor

¡Mantener el proyecto actualizado y luciendo genial es un esfuerzo grupal! El WSTG es un documento constantemente actualizado y se beneficia de tu revisión técnica o editorial.

Al enviar tu [solicitud de extracción](#cómo-enviar-una-solicitud-de-extracción), revisores y editores deberían vincular contribuciones a un issue:

1. Elige un [issue abierto con la etiqueta `help wanted`](https://github.com/OWASP/wstg/labels/help%20wanted) para trabajar, o [abre un issue](https://github.com/OWASP/wstg/issues/new/choose) tú mismo. Publica un comentario en el issue y solicita ser asignado a él.
2. Crea y cambia a una nueva rama local con el nombre `fix-<número de issue>`. Por ejemplo, `git checkout -b fix-88`.

### Revisión Técnica

Si tienes experiencia en cualquier tema cubierto por el WSTG, tu revisión técnica es alentada. Por favor asegura que los artículos:

- Sigan los [materiales de plantilla de artículo](template)
- Sigan la [guía de estilo](style_guide.md)
- Describan con precisión vulnerabilidades y pruebas
- Tengan enlaces inline apropiados y actualizados a recursos
- Proporcionen información completa y relevante adecuada para una audiencia con experiencia técnica básica

### Revisión Editorial

¡Gramáticos, únanse! El WSTG da la bienvenida a tus mejoras en las áreas de gramática, formato, elección de palabras y brevedad. Todos los cambios deberían adherirse a la [guía de estilo](style_guide.md).

Por favor no dudes en hacer tantos cambios como veas fit, especialmente si notas que el contenido existente no coincide con los [materiales de plantilla de artículo](template).

### Traducción

Debido a desafíos con sincronizar imágenes y contenido removido, el WSTG ya no está abordando esfuerzos de traducción entrante directamente.

En este momento sugerimos que inicies otro repositorio en el que abordar traducciones de un idioma específico. Una vez que hayas producido un PDF para una versión dada de la guía estaremos felices de adjuntarlo al release apropiado. Simplemente [abre un issue](https://github.com/OWASP/wstg/issues/new) aquí pidiendo que lo hagamos.

También estamos dispuestos a listar tu repositorio de traducción, solo [házlo saber](https://github.com/OWASP/wstg/issues/new) dónde está.

## Cómo Abrir un Issue

[Crea un issue](https://github.com/OWASP/wstg/issues/new/choose) usando la plantilla apropiada.

Elige un título corto, descriptivo. Explica brevemente qué crees que necesita cambiar. Entre otras cosas, tus sugerencias pueden incluir errores de gramática o ortografía, o abordar contenido insuficiente o desactualizado.

## Cómo Enviar una Solicitud de Extracción

Aquí están los pasos para crear y enviar una Solicitud de Extracción (PR) que podamos revisar y fusionar rápidamente.

1. [Configura tu entorno](#cómo-configurar-tu-entorno-de-contribuidor) para forkear el proyecto e instalar un linter de Markdown.
2. Asocia tu contribución con un [issue](https://github.com/OWASP/wstg/issues). Para cambiar contenido existente, lee [Conviértete en Revisor o Editor](#conviértete-en-revisor-o-editor). Para hacer adiciones, lee [Conviértete en Autor](#conviértete-en-autor).
3. Haz tus modificaciones. Asegúrate de seguir nuestra [guía de estilo](style_guide.md).
4. Cuando estés listo para enviar tu trabajo, empuja tus cambios a tu fork. Asegura que tu fork esté [sincronizado con `master`](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/syncing-a-fork).
5. Puedes enviar una [PR borrador](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests#draft-pull-requests) o una [PR regular](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork). Si tu trabajo no está listo para revisión y fusión, elige una PR borrador. Cuando tus cambios estén listos para ser revisados, puedes convertir a una PR regular. Ver [cómo cambiar el stage de una PR](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/changing-the-stage-of-a-pull-request) para más.

Puedes querer [permitir ediciones de maintainers](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/allowing-changes-to-a-pull-request-branch-created-from-a-fork) para que podamos ayudar con pequeños cambios como corregir typos.

Una vez que hayas enviado tu PR lista-para-revisión, la revisaremos. Podemos comentar para pedir aclaración o cambios, así que por favor revisa en los próximos días.

Para aumentar las chances de que tu PR sea fusionada, por favor asegura que:

1. Has seguido las directrices arriba para asociar tu trabajo con un issue.
2. Tu trabajo está linted en Markdown.
3. Tu escritura sigue los [materiales de plantilla de artículo](template) y [guía de estilo](style_guide.md).
4. Tus snippets de código son correctos, bien probados, y comentados donde sea necesario para comprensión.

Una vez que la PR esté completa, la fusionaremos! En ese punto, puedes querer agregarte a [la lista del proyecto de autores, revisores o editores](document/1-Frontispiece/README.md).

## Cómo Configurar Tu Entorno de Contribuidor

1. [Crea una cuenta en GitHub](https://help.github.com/en/github/getting-started-with-github/signing-up-for-a-new-github-account).
2. Instala [Visual Studio Code](https://code.visualstudio.com/) y este [plugin de linter de Markdown](https://github.com/DavidAnson/vscode-markdownlint#install). Usamos este linter para ayudar a mantener el contenido del proyecto consistente y bonito.

    1. Desde el ícono de engranaje/menu selecciona "Settings".
    2. Selecciona la pestaña "Workspace".
    3. Expande "Extensions", y encuentra "markdownlint".
    4. Justo debajo de "Markdownlint: config" haz clic en el enlace "Edit in settings.json".
    5. Agrega lo siguiente:

    ```json
    "markdownlint.config": {
      "extends": ".github/configs/.markdownlint.json"
    }
    ```

3. Forkea y clona tu propia copia del repositorio. Aquí están instrucciones completas para [forkear y sincronizar con GitHub](https://help.github.com/en/github/getting-started-with-github/fork-a-repo).

## Contribuyendo con Codespaces

Hemos incluido configuraciones para GitHub Codespaces para que puedas usar un IDE alojado en la nube para contribuir a este repositorio! Nuestra configuración incluye extensiones de Visual Studio Code y configuraciones de `markdownlint` que ayudan a mantener el trabajo consistente en todos nuestros increíbles contribuidores.

Codespaces está actualmente en beta limitada. Para aprender más, ver [About Codespaces](https://docs.github.com/en/github/developing-online-with-codespaces/about-codespaces).

Si tienes acceso a la beta, comienza [creando un codespace](https://docs.github.com/en/github/developing-online-with-codespaces/creating-a-codespace) para este repositorio.