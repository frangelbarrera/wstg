# Inyección CSV

|ID          |
|------------|
|WSTG-INJT-21|

## Resumen

La inyección CSV (también conocida como inyección de fórmulas) ocurre cuando una aplicación incluye entradas no confiables controladas por el usuario en exportaciones CSV (u otros formatos compatibles con hojas de cálculo) y el archivo resultante se abre en un programa de hojas de cálculo (por ejemplo, Microsoft Excel o LibreOffice Calc). Las aplicaciones de hojas de cálculo pueden interpretar ciertos valores de celda como fórmulas, lo que puede provocar problemas de seguridad como engaño al usuario (flujos de trabajo de tipo phishing), manipulación de los resultados de la hoja de cálculo o exfiltración de datos. En algunos entornos, la inyección de fórmulas puede escalar a un impacto mayor mediante «gadgets» de hojas de cálculo y características heredadas (por ejemplo, comportamientos DDE / Dynamic Data Exchange), y potencialmente llegar a la ejecución de comandos en la estación de trabajo que abre el archivo; esto suele depender de la configuración del cliente o de la interacción del usuario.

Una característica clave de este problema es que la vulnerabilidad a menudo solo se manifiesta cuando un usuario (por ejemplo, un administrador o alguien de finanzas o soporte) abre el archivo exportado en una aplicación de hojas de cálculo.

## Objetivos de las pruebas

- Identificar las funciones de exportación CSV/hojas de cálculo que incluyen entradas no confiables.
- Verificar si los valores controlados por un atacante se interpretan como fórmulas cuando la exportación se abre en aplicaciones de hojas de cálculo de uso común.
- Comprobar si la inyección de separadores y comillas puede mover un prefijo peligroso al inicio de una celda.
- Validar si las mitigaciones siguen siendo eficaces en Microsoft Excel después de guardar y volver a abrir el CSV.
- Evaluar el impacto práctico en función de quién abre la exportación y cómo se utiliza.

## Cómo probar

### Prefijos que activan fórmulas

Las celdas que comienzan con los siguientes caracteres pueden ser interpretadas como fórmulas por el software de hojas de cálculo:

- Igual (`=`)
- Más (`+`)
- Menos (`-`)
- Arroba (`@`)
- Tabulación (`0x09`)
- Retorno de carro (`0x0D`)
- Salto de línea (`0x0A`)
- Variantes de ancho completo (doble byte), como `＝`, `＋`, `－`, `＠` (según la configuración regional y el comportamiento de la aplicación)

> Importante (comportamiento de Excel): Microsoft Excel puede eliminar comillas o caracteres de escape de las celdas CSV cuando se guarda y se vuelve a abrir un archivo. Como resultado, algunas mitigaciones que se suelen recomendar pueden fallar después de guardar y volver a abrir el archivo, y las fórmulas que antes estaban escapadas pueden volver a activarse.

También hay que tener en cuenta que no basta con asegurarse de que la entrada no confiable *en su conjunto* no comience con un carácter peligroso. Los atacantes pueden inyectar separadores y comillas para iniciar una nueva celda y colocar el carácter peligroso al principio de esta.

### Identificar las funciones de exportación CSV y las fuentes de datos

Localizar las funciones que generan contenido CSV/TSV o contenido para «exportar a una hoja de cálculo»:

- Informes (usuarios, transacciones, registros de auditoría, tickets)
- Paneles de administración que exportan listas
- Archivos adjuntos de correo electrónico generados por la aplicación
- Exportaciones programadas / integraciones

Identificar las fuentes de datos no confiables que pueden terminar en la exportación:

- Perfiles de usuario (nombre, correo electrónico, empresa)
- Campos de texto libre (comentarios, asuntos de tickets, notas)
- Datos externos importados o integrados (webhooks, sincronización con CRM, fuentes de socios)

Documentar qué roles pueden iniciar la exportación y qué roles probablemente la abrirán.

### Incluir valores benignos y detectables similares a fórmulas en los campos candidatos

Usar cargas útiles inofensivas para detectar la evaluación de fórmulas (evitar cargas útiles que ejecuten comandos o realicen accesos a la red sin control).
Probar valores que comiencen con cada carácter que activa fórmulas:

- `=1+1`
- `+1+1`
- `-1+1`
- `@SUM(1,1)`
- `=HYPERLINK("http://example.invalid/leak?test=1", "Click Me")`

Notas:

- El caso `HYPERLINK()` sirve para demostrar un impacto realista (flujos de engaño o phishing, o posible exposición de metadatos cuando se hace clic en un enlace o se abre). Durante las pruebas, usar un endpoint controlado (por ejemplo, un listener local o un host de pruebas interno).
- No usar infraestructura externa de un «atacante real» durante la validación.

Probar también variantes con caracteres de control y Unicode (cuando el procesamiento de entradas lo permita):

- Un valor que comience con un carácter de tabulación seguido de `=1+1` (TAB + `=1+1`)
- Variantes de prefijos de ancho completo (por ejemplo, `＝1+1`)

### Probar escenarios de «salida de celda» mediante separadores y comillas

Como CSV está basado en celdas, comprobar si es posible inyectar contenido que inicie una nueva celda y luego comience con un carácter peligroso. Esto depende de:

- El separador de campos (comúnmente `,` o `;`)
- Las reglas de entrecomillado y escape
- La generación y codificación de CSV en la aplicación

Ejemplos de patrones de prueba *benignos* (ajustar el separador al formato real de exportación):

- Un valor que contenga una comilla y un separador destinados a crear una nueva celda, seguido de `=1+1`
- Un valor que contenga directamente un separador (si el exportador no lo entrecomilla), seguido de `=1+1`

El objetivo es comprobar si el CSV resultante contiene alguna celda cuyo primer carácter sea uno de los prefijos que activan fórmulas. Verificarlo inspeccionando el contenido sin procesar del CSV en un editor de texto.

### Exportar y verificar en aplicaciones de hojas de cálculo

- Exportar o descargar el CSV.
- Abrirlo en al menos una aplicación de hojas de cálculo utilizada en el entorno objetivo (Excel o LibreOffice, entre otras).
- Confirmar si la celda se interpreta como una fórmula:
    - La hoja de cálculo muestra el resultado calculado (por ejemplo, `2`) en lugar de la cadena literal (por ejemplo, `=1+1`), o
    - La barra de fórmulas muestra una fórmula en vez de texto sin formato.

Registrar:

- Nombre y versión de la aplicación (por ejemplo, Microsoft Excel)
- Cómo se abrió el archivo (doble clic o asistente de importación)
- Si la configuración regional influyó en el análisis

### Prueba de regresión de guardar y volver a abrir en Excel (fiabilidad de las mitigaciones)

Si la aplicación afirma que escapa o entrecomilla los valores:

- Abrir en Microsoft Excel el CSV exportado.
- Guardar el archivo (por ejemplo, como CSV).
- Cerrar y volver a abrir el archivo.
- Comprobar de nuevo si los valores previamente «neutralizados» se convirtieron en fórmulas activas.

Este paso es fundamental porque Excel puede normalizar o eliminar ciertos comportamientos de escape o entrecomillado después de guardar y volver a abrir el archivo.

### Evaluar el impacto

Evaluar el riesgo práctico en función de:

- Quién abre el archivo (roles con privilegios frente a usuarios finales)
- Si las exportaciones se comparten externamente
- Si se confía en los valores de la hoja de cálculo para tomar decisiones (por ejemplo, totales, indicadores, estados)
- Si las fórmulas podrían engañar a los usuarios o activar comportamientos de la hoja de cálculo en el entorno

Proporcionar una ruta de reproducción clara:

1. Dónde se introduce la entrada
2. Cómo se genera la exportación
3. Qué programa la interpreta como fórmula
4. Evidencia (captura de pantalla o descripción) de que se produce la evaluación

### Validación de impacto mayor (opcional, estrictamente controlada)

Algunos casos reales muestran que la inyección de fórmulas puede escalar más allá del cálculo o el engaño simples (por ejemplo, mediante características heredadas de las hojas de cálculo, como DDE). Este comportamiento depende mucho del entorno.

## Remediación

No existe una estrategia universal de saneamiento de CSV que sea segura para todas las aplicaciones de hojas de cálculo y los consumidores posteriores. Para los archivos CSV destinados a la lectura humana en software de hojas de cálculo, aplicar una estrategia de defensa en profundidad:

- Asegurarse de que ninguna celda comience con caracteres que activen fórmulas (`=`, `+`, `-`, `@`) ni con variantes pertinentes de control o Unicode.
- Considerar el saneamiento por campo (recomendaciones habituales):
    - Encerrar cada celda entre comillas dobles
    - Anteponer una comilla simple a cada celda
    - Escapar las comillas dobles duplicándolas  
  Nota: Esto puede dejar de ser fiable en Microsoft Excel después de guardar y volver a abrir el archivo.

- Mitigación resistente a Excel (comportamiento observado):
    - Si una celda comienza con `=`, `+`, `-` o `@`, anteponer al contenido un carácter de tabulación (`0x09`) dentro de un campo entrecomillado (por ejemplo, `"\t=1+1"`).  
  Desventaja: La tabulación pasa a formar parte de los datos subyacentes y puede afectar a las importaciones programáticas posteriores.

Validar siempre la mitigación elegida con las aplicaciones de hojas de cálculo y los flujos de trabajo incluidos en el alcance.

## Herramientas

- Proxy de interceptación (por ejemplo, Burp Suite, ZAP) para inyectar valores controlados y reproducir el flujo exacto de exportación.
- Software de hojas de cálculo para la validación en un entorno aislado (por ejemplo, Microsoft Excel, LibreOffice Calc).
- Un listener HTTP seguro (local o en un host de pruebas interno) para observar clics o solicitudes al validar el comportamiento de `HYPERLINK()`.
- Un editor de texto para inspeccionar el CSV sin procesar y confirmar si alguna celda comienza con un prefijo que activa fórmulas.

## Referencias

- [Inyección CSV de OWASP](https://owasp.org/www-community/attacks/CSV_Injection)
- [CWE-1236: Neutralización incorrecta de elementos de fórmula en un archivo CSV](https://cwe.mitre.org/data/definitions/1236.html)
- [Informe de HackerOne #72785: Inyección CSV](https://hackerone.com/reports/72785)
- [Informe de HackerOne #118582: Inyección CSV](https://hackerone.com/reports/118582)
- [Informe de HackerOne #1748961: Inyección CSV](https://hackerone.com/reports/1748961)
- [Informe de HackerOne #244292: Inyección CSV](https://hackerone.com/reports/244292)