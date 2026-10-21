# Métodos de autenticación débiles

|ID          |
|------------|
|WSTG-ATHN-07|

## Resumen

El mecanismo de autenticación más común y fácil de administrar es una contraseña estática. La contraseña representa las llaves del reino, pero los usuarios suelen debilitarla en nombre de la facilidad de uso. En cada uno de los recientes ataques de alto perfil que han revelado credenciales de usuarios, se lamenta que las contraseñas más comunes sigan siendo: `123456`, `password` y `qwerty`.

Además, las aplicaciones pueden utilizar credenciales alternativas que se tratan igual que una contraseña, pero son considerablemente más débiles, como fechas de nacimiento, números de seguridad social, PIN o preguntas de seguridad. En algunos escenarios, estas credenciales más fáciles de adivinar pueden ser el único valor que proporciona el usuario para autenticarse.

## Objetivos de la prueba

- Determinar la resistencia de la aplicación a los intentos de adivinar contraseñas por fuerza bruta mediante diccionarios de contraseñas disponibles, evaluando los requisitos de longitud, complejidad, reutilización y caducidad de las contraseñas.

## Cómo probar

1. ¿Qué caracteres están permitidos y prohibidos en las contraseñas? ¿Se exige que el usuario utilice caracteres de diferentes conjuntos, como letras minúsculas y mayúsculas, dígitos y símbolos especiales?
2. ¿Con qué frecuencia puede un usuario cambiar su contraseña? ¿Cuánto tiempo debe transcurrir desde un cambio anterior para que el usuario pueda volver a cambiarla? Los usuarios pueden eludir los requisitos del historial de contraseñas cambiándola 5 veces seguidas para volver a establecer la contraseña inicial después del último cambio.
3. ¿Cuándo debe un usuario cambiar su contraseña?
    - Tanto [NIST](https://pages.nist.gov/800-63-4/sp800-63b.html) como [NCSC](https://www.ncsc.gov.uk/collection/passwords/updating-your-approach#PasswordGuidance:UpdatingYourApproach-Don'tenforceregularpasswordexpiry) recomiendan **no** forzar la caducidad periódica de las contraseñas, aunque esto puede ser un requisito de estándares como PCI DSS.
4. ¿Con qué frecuencia puede un usuario reutilizar una contraseña? ¿La aplicación mantiene un historial de las 8 contraseñas anteriores que utilizó el usuario?
5. ¿Cuánto debe diferir la siguiente contraseña de la última?
6. ¿Se impide que el usuario incluya su nombre de usuario u otra información de la cuenta (como el nombre o los apellidos) en la contraseña?
7. ¿Cuáles son las longitudes mínima y máxima que se pueden establecer para las contraseñas, y son apropiadas para la sensibilidad de la cuenta y de la aplicación?
8. ¿Es posible establecer contraseñas comunes, como `Password1` o `123456`?
9. ¿La aplicación elige la credencial del usuario, como un número de seguridad social o una fecha de nacimiento? ¿La credencial que se utiliza en lugar de una contraseña estándar se puede obtener fácilmente, es predecible o es susceptible a ataques de fuerza bruta?

## Remediación

Para mitigar el riesgo de que las contraseñas fáciles de adivinar faciliten el acceso no autorizado, hay dos soluciones: introducir controles de autenticación adicionales (es decir, autenticación de dos factores) o implementar una política de contraseñas seguras. La solución más sencilla y económica es implementar una política de contraseñas seguras que garantice la longitud, la complejidad, la reutilización y la caducidad de las contraseñas; aunque, idealmente, deberían implementarse ambas.

## Referencias

- [Ataques de fuerza bruta](https://owasp.org/www-community/attacks/Brute_force_attack)