# Pruebas de métodos de autenticación débiles

|ID          |
|------------|
|WSTG-ATHN-07|

## Resumen

El mecanismo de autenticación más prevalente y más fácilmente administrado es una contraseña estática. La contraseña representa las llaves del reino, pero a menudo es subvertida por los usuarios en nombre de la usabilidad. En cada uno de los recientes hacks de alto perfil que han revelado credenciales de usuario, se lamenta que las contraseñas más comunes siguen siendo: `123456`, `password` y `qwerty`.

Además, las aplicaciones pueden utilizar credenciales alternativas que se tratan igual que una contraseña, pero son considerablemente más débiles, como fechas de nacimiento, números de seguridad social, PINs o preguntas de seguridad. En algunos escenarios, estas credenciales más fácilmente adivinables pueden actuar como el único valor proporcionado por el usuario para la autenticación.

## Objetivos de la prueba

- Determinar la resistencia de la aplicación contra la adivinación de contraseñas por fuerza bruta utilizando diccionarios de contraseñas disponibles evaluando la longitud, complejidad, reutilización y requisitos de envejecimiento de las contraseñas.

## Cómo probar

1. ¿Qué caracteres están permitidos y prohibidos para usar dentro de una contraseña? ¿Se requiere que el usuario use caracteres de diferentes conjuntos de caracteres como letras minúsculas y mayúsculas, dígitos y símbolos especiales?
2. ¿Con qué frecuencia puede un usuario cambiar su contraseña? ¿Con qué rapidez puede un usuario cambiar su contraseña después de un cambio anterior? Los usuarios pueden eludir los requisitos de historial de contraseñas cambiando su contraseña 5 veces en una fila para que después del último cambio de contraseña hayan configurado su contraseña inicial nuevamente.
3. ¿Cuándo debe un usuario cambiar su contraseña?
    - Tanto [NIST](https://pages.nist.gov/800-63-3/sp800-63b.html#memsecretver) como [NCSC](https://www.ncsc.gov.uk/collection/passwords/updating-your-approach#PasswordGuidance:UpdatingYourApproach-Don'tenforceregularpasswordexpiry) recomiendan **contra** forzar la expiración regular de contraseñas, aunque puede ser requerido por estándares como PCI DSS.
4. ¿Con qué frecuencia puede un usuario reutilizar una contraseña? ¿La aplicación mantiene un historial de las 8 contraseñas anteriores utilizadas por el usuario?
5. ¿Qué tan diferente debe ser la siguiente contraseña de la última contraseña?
6. ¿Se impide al usuario usar su nombre de usuario u otra información de cuenta (como nombre o apellido) en la contraseña?
7. ¿Cuáles son las longitudes mínimas y máximas de contraseña que se pueden establecer, y son apropiadas para la sensibilidad de la cuenta y la aplicación?
8. ¿Es posible establecer contraseñas comunes como `Password1` o `123456`?
9. ¿La credencial elegida para el usuario por la aplicación, como un número de seguridad social o una fecha de nacimiento? ¿La credencial que se utiliza en lugar de una contraseña estándar es fácilmente obtenible, predecible o susceptible a ataques de fuerza bruta?

## Remediation

Para mitigar el riesgo de que contraseñas fácilmente adivinables faciliten el acceso no autorizado, hay dos soluciones: introducir controles de autenticación adicionales (es decir, autenticación de dos factores) o introducir una política de contraseñas fuertes. La más simple y barata de estas es la introducción de una política de contraseñas fuertes que garantice la longitud, complejidad, reutilización y envejecimiento de las contraseñas; aunque idealmente ambas deberían implementarse.

## Referencias

- [Ataques de fuerza bruta](https://owasp.org/www-community/attacks/Brute_force_attack)