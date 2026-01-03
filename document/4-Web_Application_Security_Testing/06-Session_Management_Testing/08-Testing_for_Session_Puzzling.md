# Pruebas de Confusión de Sesión

|ID          |
|------------|
|WSTG-SESS-08|

## Resumen

La Sobrecarga de Variables de Sesión (también conocida como Confusión de Sesión) es una vulnerabilidad a nivel de aplicación que puede permitir a un atacante realizar una variedad de acciones maliciosas, incluyendo, pero no limitado a:

- Eludir mecanismos eficientes de aplicación de autenticación e impersonar usuarios legítimos.
- Elevar los privilegios de una cuenta de usuario maliciosa, en un entorno que de otro modo se consideraría a prueba de tontos.
- Saltar fases de calificación en procesos multifase, incluso si el proceso incluye todas las restricciones recomendadas a nivel de código.
- Manipular valores del lado del servidor en métodos indirectos que no pueden predecirse o detectarse.
- Ejecutar ataques tradicionales en ubicaciones que anteriormente eran inalcanzables o incluso consideradas seguras.

Esta vulnerabilidad ocurre cuando una aplicación utiliza la misma variable de sesión para más de un propósito. Un atacante puede potencialmente acceder a páginas en un orden no anticipado por los desarrolladores de modo que la variable de sesión se establece en un contexto y luego se utiliza en otro.

Por ejemplo, un atacante podría utilizar la sobrecarga de variables de sesión para eludir mecanismos de aplicación de autenticación de aplicaciones que aplican autenticación validando la existencia de variables de sesión que contienen valores relacionados con la identidad, que generalmente se almacenan en la sesión después de un proceso de autenticación exitoso. Esto significa que un atacante primero accede a una ubicación en la aplicación que establece el contexto de sesión y luego accede a ubicaciones privilegiadas que examinan este contexto.

Por ejemplo: un vector de ataque de elusión de autenticación podría ejecutarse accediendo a un punto de entrada de acceso público (por ejemplo, una página de recuperación de contraseña) que rellena la sesión con una variable de sesión idéntica, basada en valores fijos o en entrada originada por el usuario.

## Objetivos de las Pruebas

- Identificar todas las variables de sesión.
- Romper el flujo lógico de generación de sesión.

## Cómo Probar

### Pruebas de Caja Negra

Esta vulnerabilidad puede detectarse y explotarse enumerando todas las variables de sesión utilizadas por la aplicación y en qué contexto son válidas. En particular, esto es posible accediendo a una secuencia de puntos de entrada y luego examinando puntos de salida. En caso de pruebas de caja negra, este procedimiento es difícil y requiere algo de suerte ya que cada secuencia diferente podría llevar a un resultado diferente.

#### Ejemplos

Un ejemplo muy simple podría ser la funcionalidad de restablecimiento de contraseña que, en el punto de entrada, podría solicitar al usuario proporcionar alguna información de identificación como el nombre de usuario o la dirección de correo electrónico. Esta página podría entonces rellenar la sesión con estos valores de identificación, que se reciben directamente del lado del cliente, o obtenidos de consultas o cálculos basados en la entrada recibida. En este punto puede haber algunas páginas en la aplicación que muestren datos privados basados en este objeto de sesión. De esta manera, el atacante podría eludir el proceso de autenticación.

### Pruebas de Caja Gris

La forma más efectiva de detectar estas vulnerabilidades es mediante una revisión del código fuente.

## Remediación

Las variables de sesión deben utilizarse solo para un propósito único consistente.

## Referencias

- [Session Puzzles](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/puzzlemall/Session%20Puzzles%20-%20Indirect%20Application%20Attack%20Vectors%20-%20May%202011%20-%20Whitepaper.pdf)
- [Session Puzzling and Session Race Conditions](https://sectooladdict.blogspot.com/2011/09/session-puzzling-and-session-race.html)