# Autenticación más débil en canales alternativos

|ID          |
|------------|
|WSTG-ATHN-10|

## Resumen

Aunque los mecanismos de autenticación principales no presenten vulnerabilidades, es posible que existan vulnerabilidades en canales legítimos alternativos de autenticación para las mismas cuentas de usuario. Se deben realizar pruebas para identificar los canales alternativos y, sujeto al alcance de las pruebas, identificar vulnerabilidades.

Los canales alternativos de interacción con el usuario podrían utilizarse para eludir el canal principal o exponer información que luego pueda utilizarse para facilitar un ataque contra este. Algunos de estos canales pueden ser aplicaciones web independientes que utilizan nombres de host o rutas diferentes. Por ejemplo:

- Sitio web estándar
- Sitio web optimizado para dispositivos móviles o dispositivos específicos
- Sitio web optimizado para accesibilidad
- Sitios web alternativos para distintos países e idiomas
- Sitios web paralelos que utilizan las mismas cuentas de usuario (por ejemplo, otro sitio web que ofrece funcionalidades diferentes de la misma organización o un sitio web asociado que comparte las cuentas de usuario)
- Versiones de desarrollo, prueba, UAT y preproducción del sitio web estándar

También podrían ser otros tipos de aplicaciones o procesos empresariales:

- Aplicación para dispositivos móviles
- Aplicación de escritorio
- Operadores de centros de llamadas
- Sistemas de respuesta de voz interactiva o de menú telefónico

Tenga en cuenta que esta prueba se centra en los canales alternativos; algunas alternativas de autenticación pueden presentarse como contenido diferente proporcionado a través del mismo sitio web y casi con toda seguridad estarán dentro del alcance de las pruebas. No se tratan más aquí y deberían haberse identificado durante la recopilación de información y las pruebas de autenticación principal. Por ejemplo:

- Enriquecimiento progresivo y degradación elegante que modifican la funcionalidad
- Uso del sitio sin cookies
- Uso del sitio sin JavaScript
- Uso del sitio sin complementos como Flash y Java

Aunque el alcance de la prueba no permita probar los canales alternativos, se debe documentar su existencia. Estos pueden reducir el grado de garantía de los mecanismos de autenticación y ser un indicio de la necesidad de realizar pruebas adicionales.

## Ejemplo

El sitio web principal es `https://www.example.com` y las funciones de autenticación siempre se realizan en páginas que utilizan TLS: `https://www.example.com/myaccount/`.

Sin embargo, existe un sitio web independiente optimizado para dispositivos móviles que no utiliza TLS y tiene un mecanismo más débil de recuperación de contraseñas: `https://m.example.com/myaccount/`.

## Objetivos de la prueba

- Identificar canales de autenticación alternativos.
- Evaluar las medidas de seguridad utilizadas y determinar si existen formas de eludir la autenticación en los canales alternativos.

## Cómo probar

### Comprender el mecanismo principal

Probar exhaustivamente las funciones de autenticación principales del sitio web. Esto debería permitir identificar cómo se emiten, crean o modifican las cuentas, y cómo se recuperan, restablecen o cambian las contraseñas. Además, se deben conocer los mecanismos de autenticación para privilegios elevados y las medidas de protección de la autenticación. Esta información previa es necesaria para poder compararla con la de los canales alternativos.

### Identificar otros canales

Se pueden encontrar otros canales mediante los siguientes métodos:

- Leer el contenido del sitio, especialmente la página de inicio, las páginas de contacto, las páginas de ayuda, los artículos de soporte y las preguntas frecuentes, los términos y condiciones, los avisos de privacidad, el archivo robots.txt y cualquier archivo sitemap.xml.
- Buscar en los registros del proxy HTTP, capturados durante las actividades anteriores de recopilación de información y pruebas, cadenas como "mobile", "Android", "blackberry", "ipad", "iphone", "mobile app", "e-reader", "wireless", "auth", "sso", "single sign on" en las rutas de las URL y el contenido del cuerpo.
- Utilizar motores de búsqueda para encontrar sitios web distintos de la misma organización, o que utilicen el mismo nombre de dominio, y que tengan contenido similar en la página de inicio o mecanismos de autenticación.

Para cada canal posible, confirmar si se comparten las cuentas de usuario o si el canal proporciona acceso a la misma funcionalidad o a una similar.

### Enumerar las funciones de autenticación

Para cada canal alternativo que comparta cuentas de usuario o funcionalidad, identificar si están disponibles todas las funciones de autenticación del canal principal y si existe alguna adicional. Puede ser útil crear una tabla como la siguiente:

  | Principal | Móvil  |  Centro de llamadas | Sitio web asociado |
  |---------|---------|--------------|-----------------|
  | Registrarse| Sí     |     -        |       -         |
  | Iniciar sesión  | Sí     |    Sí       |    Sí(SSO)     |
  | Cerrar sesión |   -     |     -        |       -         |
  |Restablecer contraseña |   Sí  |   Sí   |       -         |
  | -       | Cambiar contraseña |   -  |       -         |

En este ejemplo, el canal móvil ofrece una función adicional, «cambiar contraseña», pero no ofrece la opción de «cerrar sesión». También es posible realizar un número limitado de tareas llamando al centro de llamadas. Los centros de llamadas pueden ser interesantes porque sus verificaciones de identidad podrían ser más débiles que las del sitio web, lo que permitiría utilizar este canal para facilitar un ataque contra la cuenta de un usuario.

Al enumerar estas funciones, conviene tomar nota de cómo se gestiona la sesión, por si existe alguna superposición entre los canales (por ejemplo, cookies con alcance al mismo dominio padre o sesiones simultáneas permitidas entre canales, pero no dentro de un mismo canal).

### Revisar y probar

Los canales alternativos deben mencionarse en el informe de pruebas, aunque se marquen como «solo informativos» o «fuera del alcance». En algunos casos, el alcance de la prueba puede incluir el canal alternativo (por ejemplo, porque es simplemente otra ruta en el nombre de host objetivo), o puede ampliarse tras conversar con los responsables de todos los canales. Si las pruebas están permitidas y autorizadas, se deben realizar todas las demás pruebas de autenticación de esta guía y comparar los resultados con los del canal principal.

## Casos de prueba relacionados

Se deben utilizar los casos de prueba de todas las demás pruebas de autenticación.

## Remediación

Asegúrese de aplicar una política de autenticación coherente en todos los canales para que sean igual de seguros.