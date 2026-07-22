# Pruebas de Inyección CSV

|ID          |
|------------|
|WSTG-INPV-21|

## Resumen

La Inyección CSV (también conocida como Inyección de Fórmulas) ocurre cuando una aplicación embebe entrada no confiable, controlada por el usuario, en exportaciones CSV (u otras compatibles con hojas de cálculo) y el archivo resultante se abre en un programa de hoja de cálculo (por ejemplo, Microsoft Excel, LibreOffice Calc). Las aplicaciones de hoja de cálculo podrían interpretar ciertos valores de celda como fórmulas, lo cual puede llevar a problemas de seguridad tales como engaño al usuario (flujos de trabajo estilo phishing), manipulación de la salida de la hoja de cálculo, o exfiltración de datos. En algunos entornos, la inyección de fórmulas puede escalarse a mayor impacto vía "gadgets" de hoja de cálculo y características heredadas (por ejemplo, comportamientos DDE / Dynamic Data Exchange), potencialmente alcanzando ejecución de comandos en la estación de trabajo que abre el archivo — típicamente dependiente de la configuración del cliente y/o interacción del usuario.

Una característica clave de este problema es que la vulnerabilidad a menudo se manifiesta solo cuando el archivo exportado es abierto por un usuario (por ejemplo, un administrador, finanzas o soporte) en una aplicación de hoja de cálculo.

## Objetivos de Prueba

- Identificar características de exportación CSV/hoja de cálculo que incluyan entrada no confiable.
- Verificar si valores controlados por el atacante se interpretan como fórmulas cuando la exportación se abre en aplicaciones comunes de hoja de cálculo.
- Verificar si la inyección de separador/comilla puede mover un prefijo peligroso al inicio de una celda.
- Validar si las mitigaciones permanecen efectivas en Microsoft Excel después de guardar y reabrir el CSV.
- Evaluar el impacto práctico basado en quién abre la exportación y cómo se usa.

## Cómo Probar

### Prefijos Disparadores de Fórmulas

Las celdas que comienzan con los siguientes caracteres podrían ser interpretadas como fórmulas por software de hoja de cálculo:

- Igual (`=`)
- Más (`+`)
- Menos (`-`)
- At (`@`)
- Tab (`0x09`)
- Retorno de carro (`0x0D`)
- Salto de línea (`0x0A`)
- Variantes de ancho completo (doble byte) tales como `＝`, `＋`, `－`, `＠` (dependiendo del comportamiento de locale/aplicación)

> Importante (comportamiento de Excel): Microsoft Excel podría remover comillas o caracteres de escape de celdas CSV cuando un archivo se guarda y se reabre. Como resultado, algunas mitigaciones comúnmente sugeridas pueden fallar después de guardar/reabrir y fórmulas previamente escapadas podrían volverse activas nuevamente.

También notar que no es suficiente asegurar que la entrada no confiable *general* no comience con un carácter peligroso. Los atacantes podrían inyectar separadores y comillas para comenzar una nueva celda, colocando el carácter peligroso al principio de una celda.

### Identificar Funcionalidad de Exportación CSV y Fuentes de Datos

Localizar características que generen contenido CSV/TSV o "exportar a hoja de cálculo":

- Reportes (usuarios, transacciones, logs de auditoría, tickets)
- Dashboards de administración exportando listas
- Archivos adjuntos de correo electrónico generados por la aplicación
- Exportaciones programadas / integraciones

Identificar fuentes de datos no confiables que pueden terminar en la exportación:

- Perfiles de usuario (nombre, email, empresa)
- Campos de texto libre (comentarios, asuntos de ticket, notas)
- Datos externos importados/integrados (webhooks, sincronización CRM, feeds de partners)

Documentar qué roles pueden disparar la exportación y qué roles probablemente la abran.

### Colocar Valores Tipo Fórmula Benignos y Detectables en Campos Candidatos

Usar payloads inofensivos para detectar evaluación de fórmulas (evitar payloads que ejecuten comandos o realicen acceso de red incontrolado).
Probar valores que comiencen con cada carácter disparador de fórmulas:

- `=1+1`
- `+1+1`
- `-1+1`
- `@SUM(1,1)`
- `=HYPERLINK("http://example.invalid/leak?test=1", "Click Me")`

Notas:

- El caso `HYPERLINK()` es útil para demostrar impacto realista (flujos de engaño/phishing, o potencial exposición de metadatos cuando un enlace se hace clic/abre). Usar un endpoint controlado durante las pruebas (por ejemplo, un listener local o un host de prueba interno).
- No usar infraestructura "de atacante real" externa en validación.

También probar variantes de caracteres de control y Unicode (donde el manejo de entrada lo permita):

- Un valor que comience con un carácter de tabulación seguido de `=1+1` (TAB + `=1+1`)
- Variantes de prefijo de ancho completo (por ejemplo, `＝1+1`)

### Probar Escenarios de "Cell Breakout" con Separador y Comilla

Como CSV está basado en celdas, probar si se puede inyectar contenido que comience una nueva celda y luego comience con un carácter peligroso. Esto depende de:

- Separador de campo (comúnmente `,` o `;`)
- Reglas de comillado y escapado
- Generación y codificación CSV del lado de la aplicación

Ejemplo de patrones de prueba *benignos* (ajustar separador al formato de exportación real):

- Un valor que contiene una comilla y separador destinado a crear una nueva celda, luego `=1+1`
- Un valor que contiene un separador directamente (si no es citado por el exportador), luego `=1+1`

El objetivo es ver si el CSV resultante contiene alguna celda cuyo primer carácter sea uno de los prefijos disparadores de fórmulas. Verificar esto inspeccionando la salida CSV cruda en un editor de texto.

### Exportar y Verificar en Aplicaciones de Hoja de Cálculo

- Exportar/descargar el CSV.
- Abrirlo en al menos una aplicación de hoja de cálculo usada en el entorno objetivo (Excel y/o LibreOffice, etc.).
- Confirmar si la celda se interpreta como una fórmula:
    - La hoja de cálculo muestra el resultado computado (por ejemplo, `2`) en lugar de la cadena literal (por ejemplo, `=1+1`), y/o
    - La barra de fórmulas muestra una fórmula en lugar de texto plano.

Registrar:

- Nombre/versión de la aplicación (por ejemplo, Microsoft Excel)
- Cómo se abrió el archivo (doble clic vs asistente de importación)
- Si las configuraciones de locale influyeron en el análisis

### Prueba de Regresión Guardar/Reabrir en Excel (Fiabilidad de Mitigación)

Si la aplicación afirma escapar/citar valores:

- Abrir el CSV exportado en Microsoft Excel.
- Guardar el archivo (por ejemplo, como CSV).
- Cerrar y reabrirlo.
- Re-verificar si valores previamente "neutralizados" se volvieron fórmulas activas.

Este paso es crítico porque Excel podría normalizar/remover ciertos comportamientos de escapado/comillado después de guardar/reabrir.

### Evaluar Impacto

Evaluar el riesgo práctico basado en:

- Quién abre el archivo (roles privilegiados vs usuarios finales)
- Si las exportaciones se comparten externamente
- Si los valores de la hoja de cálculo se confían para decisiones (por ejemplo, totales, flags, estados)
- Si las fórmulas podrían engañar a usuarios o disparar comportamientos de hoja de cálculo en el entorno

Proporcionar una ruta de reproducción clara:

1. Dónde se ingresa la entrada
2. Cómo se genera la exportación
3. Qué programa la interpreta como fórmula
4. Evidencia (captura de pantalla o descripción) de que ocurre evaluación

### Validación de Mayor Impacto (Opcional, Estrictamente Controlada)

Algunos casos del mundo real muestran inyección de fórmulas escalando más allá de simple cálculo/engaño (por ejemplo, vía características heredadas de hoja de cálculo tales como DDE). Este comportamiento es altamente dependiente del entorno.

## Remediación

No hay una estrategia universal de saneamiento CSV segura para todas las aplicaciones de hoja de cálculo y consumidores posteriores. Para CSVs destinados a visualización humana en software de hoja de cálculo, aplicar un enfoque de defensa en profundidad:

- Asegurar que ninguna celda comience con caracteres disparadores de fórmulas (`=`, `+`, `-`, `@`) y variantes de control/Unicode relevantes.
- Considerar saneamiento por campo (comúnmente sugerido):
    - Envolver cada celda en comillas dobles
    - Prependar cada celda con una comilla simple
    - Escapar comillas dobles duplicándolas
  Nota: Esto podría no permanecer confiable en Microsoft Excel después de guardar y reabrir.

- Mitigación resistente a Excel (comportamiento observado):
    - Si una celda comienza con `=`, `+`, `-` o `@`, prefixar el contenido con un carácter de tabulación (`0x09`) dentro de un campo citado (por ejemplo, `"\t=1+1"`).
  Compromiso: El tab se vuelve parte de los datos subyacentes y podría afectar importaciones programáticas posteriores.

Siempre validar la mitigación elegida contra las aplicaciones de hoja de cálculo y flujos de trabajo en alcance.

## Herramientas

- Proxy de intercepción (por ejemplo, Burp Suite, ZAP) para inyectar valores controlados y reproducir el flujo de exportación exacto.
- Software de hoja de cálculo para validación en un entorno aislado (por ejemplo, Microsoft Excel, LibreOffice Calc).
- Un listener HTTP seguro (local o host de prueba interno) para observar clics/solicitudes al validar el comportamiento de `HYPERLINK()`.
- Un editor de texto para inspeccionar el CSV crudo y confirmar si alguna celda comienza con un prefijo disparador de fórmulas.

## Referencias

- [OWASP CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- [CWE-1236: Improper Neutralization of Formula Elements in a CSV File](https://cwe.mitre.org/data/definitions/1236.html)
- [HackerOne Report #72785: CSV Injection](https://hackerone.com/reports/72785)
- [HackerOne Report #118582: CSV Injection](https://hackerone.com/reports/118582)
- [HackerOne Report #1748961: CSV Injection](https://hackerone.com/reports/1748961)
- [HackerOne Report #244292: CSV Injection](https://hackerone.com/reports/244292)
