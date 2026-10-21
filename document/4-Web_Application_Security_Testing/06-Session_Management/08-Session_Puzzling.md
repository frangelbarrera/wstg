# Confusión de sesión

|ID          |
|------------|
|WSTG-SESS-08|

## Resumen

La sobrecarga de variables de sesión (también conocida como confusión de sesión) es una vulnerabilidad a nivel de aplicación que puede permitir a un atacante realizar diversas acciones maliciosas, entre ellas:

- Eludir mecanismos eficaces de aplicación de la autenticación e suplantar a usuarios legítimos.
- Elevar los privilegios de una cuenta de usuario maliciosa en un entorno que, de otro modo, se consideraría infalible.
- Saltarse etapas de validación en procesos de varias fases, incluso si el proceso incluye todas las restricciones recomendadas habitualmente a nivel de código.
- Manipular valores del lado del servidor mediante métodos indirectos que no pueden predecirse ni detectarse.
- Ejecutar ataques tradicionales en ubicaciones que antes eran inaccesibles o incluso se consideraban seguras.

Esta vulnerabilidad se produce cuando una aplicación utiliza la misma variable de sesión para más de un propósito. Un atacante puede acceder a páginas en un orden no previsto por los desarrolladores, de modo que la variable de sesión se establece en un contexto y luego se utiliza en otro.

Por ejemplo, un atacante podría utilizar la sobrecarga de variables de sesión para eludir los mecanismos de aplicación de la autenticación de aplicaciones que aplican la autenticación validando la existencia de variables de sesión que contienen valores relacionados con la identidad, que suelen almacenarse en la sesión después de un proceso de autenticación satisfactorio. Esto significa que el atacante primero accede a una ubicación de la aplicación que establece el contexto de sesión y luego accede a ubicaciones privilegiadas que examinan dicho contexto.

Por ejemplo, un vector de ataque de elusión de la autenticación podría ejecutarse accediendo a un punto de entrada de acceso público (por ejemplo, una página de recuperación de contraseña) que rellena la sesión con una variable de sesión idéntica, basada en valores fijos o en datos introducidos por el usuario.

## Objetivos de las pruebas

- Identificar todas las variables de sesión.
- Romper el flujo lógico de generación de sesiones.

## Cómo probar

### Pruebas de caja negra

Esta vulnerabilidad puede detectarse y explotarse enumerando todas las variables de sesión utilizadas por la aplicación y los contextos en los que son válidas. En particular, esto es posible accediendo a una secuencia de puntos de entrada y luego examinando los puntos de salida. En las pruebas de caja negra, este procedimiento es difícil y requiere algo de suerte, ya que cada secuencia diferente podría dar lugar a un resultado distinto.

#### Ejemplos

Un ejemplo muy sencillo podría ser la funcionalidad de restablecimiento de contraseña, cuyo punto de entrada podría solicitar al usuario que proporcione algún dato de identificación, como el nombre de usuario o la dirección de correo electrónico. Esta página podría rellenar la sesión con esos datos de identificación, que se reciben directamente del lado del cliente o se obtienen mediante consultas o cálculos basados en los datos recibidos. En este punto, puede haber páginas de la aplicación que muestren datos privados basándose en este objeto de sesión. De esta manera, el atacante podría eludir el proceso de autenticación.

### Pruebas de caja gris

La forma más eficaz de detectar estas vulnerabilidades es mediante una revisión del código fuente.

## Remediación

Las variables de sesión deben utilizarse únicamente para un propósito único y coherente.

## Referencias

- [Session Puzzles](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/puzzlemall/Session%20Puzzles%20-%20Indirect%20Application%20Attack%20Vectors%20-%20May%202011%20-%20Whitepaper.pdf)
- [Session Puzzling and Session Race Conditions](https://sectooladdict.blogspot.com/2011/09/session-puzzling-and-session-race.html)