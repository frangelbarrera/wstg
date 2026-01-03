# Pruebas de autenticación más débil en canales alternativos

|ID          |
|------------|
|WSTG-ATHN-10|

## Resumen

Incluso si los mecanismos de autenticación principales no incluyen vulnerabilidades, puede que existan vulnerabilidades en canales de autenticación legítimos alternativos para las mismas cuentas de usuario. Se deben realizar pruebas para identificar canales alternativos y, sujeto al alcance de la prueba, identificar vulnerabilidades.

Los canales de interacción alternativos del usuario podrían utilizarse para eludir el canal principal o exponer información que luego se puede usar para ayudar en un ataque contra el canal principal. Algunos de estos canales pueden ser aplicaciones web separadas que utilizan nombres de host o rutas diferentes. Por ejemplo:

- Sitio web estándar
- Sitio web optimizado para móvil o dispositivo específico
- Sitio web optimizado para accesibilidad
- Sitios web alternativos de países e idiomas
- Sitios web paralelos que utilizan las mismas cuentas de usuario (por ejemplo, otro sitio web que ofrece funcionalidad diferente de la misma organización, un sitio web de socio con el que se comparten cuentas de usuario)
- Versiones de desarrollo, prueba, UAT y staging del sitio web estándar

Pero también podrían ser otros tipos de aplicación o procesos empresariales:

- Aplicación de dispositivo móvil
- Aplicación de escritorio
- Operadores de centro de llamadas
- Sistemas de respuesta de voz interactiva o árbol telefónico

Tenga en cuenta que el enfoque de esta prueba está en canales alternativos; algunas alternativas de autenticación podrían aparecer como contenido diferente entregado a través del mismo sitio web y casi seguramente estarían en el alcance de las pruebas. Estos no se discuten más aquí y deberían haber sido identificados durante la recopilación de información y las pruebas de autenticación principales. Por ejemplo:

- Enriquecimiento progresivo y degradación elegante que cambian la funcionalidad
- Uso del sitio sin cookies
- Uso del sitio sin JavaScript
- Uso del sitio sin complementos como para Flash y Java

Incluso si el alcance de la prueba no permite probar los canales alternativos, su existencia debería documentarse. Estos pueden socavar el grado de confianza en los mecanismos de autenticación y pueden ser un precursor para pruebas adicionales.

## Ejemplo

El sitio web principal es `https://www.example.com` y las funciones de autenticación siempre tienen lugar en páginas que utilizan TLS `https://www.example.com/myaccount/`.

Sin embargo, existe un sitio web optimizado para móvil separado que no utiliza TLS en absoluto y tiene un mecanismo de recuperación de contraseña más débil `https://m.example.com/myaccount/`.

## Objetivos de la prueba

- Identificar canales de autenticación alternativos.
- Evaluar las medidas de seguridad utilizadas y si existen bypasses en los canales alternativos.

## Cómo probar

### Entender el mecanismo principal

Probar completamente las funciones de autenticación principales del sitio web. Esto debería identificar cómo se emiten, crean o cambian las cuentas y cómo se recuperan, restablecen o cambian las contraseñas. Además, se debe conocer cualquier autenticación de privilegios elevados y medidas de protección de autenticación. Estos precursores son necesarios para poder comparar con cualquier canal alternativo.

### Identificar otros canales

Otros canales se pueden encontrar utilizando los siguientes métodos:

- Leer el contenido del sitio, especialmente la página de inicio, contáctenos, páginas de ayuda, artículos de soporte y preguntas frecuentes, T&C, avisos de privacidad, el archivo robots.txt y cualquier archivo sitemap.xml.
- Buscar en los registros del proxy HTTP, grabados durante la recopilación de información y pruebas anteriores, cadenas como "mobile", "android", "blackberry", "ipad", "iphone", "mobile app", "e-reader", "wireless", "auth", "sso", "single sign on" en rutas URL y contenido del cuerpo.
- Usar motores de búsqueda para encontrar diferentes sitios web de la misma organización, o utilizando el mismo nombre de dominio, que tengan contenido de página de inicio similar o que también tengan mecanismos de autenticación.

Para cada canal posible, confirmar si las cuentas de usuario se comparten entre estos, o proporcionan acceso a la misma o similar funcionalidad.

### Enumerar funcionalidad de autenticación

Para cada canal alternativo donde las cuentas de usuario o funcionalidad se comparten, identificar si todas las funciones de autenticación del canal principal están disponibles, y si existe algo extra. Puede ser útil crear una cuadrícula como la siguiente:

  | Principal | Móvil  |  Centro de llamadas | Sitio web de socio |
  |-----------|--------|---------------------|-------------------|
  | Registrarse| Sí     |     -               |       -           |
  | Iniciar sesión  | Sí     |    Sí              |    Sí(SSO)        |
  | Cerrar sesión |   -     |     -               |       -           |
  | Restablecer contraseña |   Sí  |   Sí              |       -           |
  | -       | Cambiar contraseña |   - |       -           |

En este ejemplo, móvil tiene una función extra "cambiar contraseña" pero no ofrece "cerrar sesión". Un número limitado de tareas también son posibles llamando al centro de llamadas. Los centros de llamadas pueden ser interesantes, porque sus comprobaciones de confirmación de identidad podrían ser más débiles que las del sitio web, permitiendo que este canal se use para ayudar en un ataque contra la cuenta de un usuario.

Mientras se enumeran estos, vale la pena tomar nota de cómo se lleva a cabo la gestión de sesiones, en caso de que haya superposición en cualquier canal (por ejemplo, cookies con alcance al mismo nombre de dominio padre, sesiones concurrentes permitidas en canales, pero no en el mismo canal).

### Revisar y probar

Los canales alternativos deberían mencionarse en el informe de pruebas, incluso si se marcan como "solo información" o "fuera de alcance". En algunos casos, el alcance de la prueba podría incluir el canal alternativo (por ejemplo, porque es solo otra ruta en el nombre de host objetivo), o podría agregarse al alcance después de discutir con los propietarios de todos los canales. Si se permite y autoriza la prueba, todas las otras pruebas de autenticación en esta guía deberían entonces realizarse y compararse con el canal principal.

## Casos de prueba relacionados

Los casos de prueba para todas las otras pruebas de autenticación deberían utilizarse.

## Remediación

Asegurar que se aplique una política de autenticación consistente en todos los canales para que sean igualmente seguros.