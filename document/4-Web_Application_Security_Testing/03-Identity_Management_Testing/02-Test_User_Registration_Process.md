# Probar Proceso de Registro de Usuario

|ID          |
|------------|
|WSTG-IDNT-02|

## Resumen

Algunos sitios web ofrecen un proceso de registro de usuario que automatiza (o semi-automatiza) el provisioning de acceso al sistema a usuarios. Los requisitos de identidad para acceso varían de identificación positiva a ninguno en absoluto, dependiendo de los requisitos de seguridad del sistema. Muchas aplicaciones públicas automatizan completamente el proceso de registro y provisioning porque el tamaño de la base de usuario hace imposible gestionarlo manualmente. Sin embargo, muchas aplicaciones corporativas provisionarán usuarios manualmente, así que este caso de prueba puede no aplicar.

## Objetivos de Prueba

- Verificar que los requisitos de identidad para registro de usuario están alineados con requisitos de negocio y seguridad.
- Validar el proceso de registro.

## Cómo Probar

Verificar que los requisitos de identidad para registro de usuario están alineados con requisitos de negocio y seguridad:

1. ¿Puede cualquiera registrarse para acceso?
2. ¿Son los registros vetados por un humano prior a provisioning, o son automáticamente granted si los criterios son met?
3. ¿Puede la misma persona o identidad registrarse múltiples veces?
4. ¿Pueden los usuarios registrarse para diferentes roles o permisos?
5. ¿Qué prueba de identidad es requerida para un registro ser exitoso?
6. ¿Son las identidades registradas verificadas?

Validar el proceso de registro:

1. ¿Puede la información de identidad ser fácilmente forged o faked?
2. ¿Puede el exchange de información de identidad ser manipulated durante registro?

### Ejemplo

En el ejemplo de WordPress abajo, el único requisito de identificación es una dirección email que es accesible al registrant.

![Página de Registro de WordPress](images/Wordpress_registration_page.jpg)\
 *Figura 4.3.2-1: Página de Registro de WordPress*

En contraste, en el ejemplo de Google abajo los requisitos de identificación incluyen nombre, fecha de nacimiento, país, número de teléfono móvil, dirección email y respuesta CAPTCHA. Mientras solo dos de estos pueden ser verificados (dirección email y número móvil), los requisitos de identificación son más estrictos que WordPress.

![Página de Registro de Google](images/Google_registration_page.jpg)\
 *Figura 4.3.2-2: Página de Registro de Google*

## Remediation

Implementar requisitos de identificación y verificación que correspondan a los requisitos de seguridad de la información que las credenciales protegen.

## Herramientas

Un proxy HTTP puede ser una herramienta útil para probar este control.

## Referencias

[Diseño de Registro de Usuario](https://mashable.com/2011/06/09/user-registration-design/)