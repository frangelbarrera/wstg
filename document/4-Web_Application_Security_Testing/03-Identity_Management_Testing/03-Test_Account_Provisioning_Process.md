# Probar Proceso de Provisioning de Cuenta

|ID          |
|------------|
|WSTG-IDNT-03|

## Resumen

El provisioning de cuentas presenta una oportunidad para un atacante crear una cuenta válida sin aplicación del proceso apropiado de identificación y autorización.

## Objetivos de Prueba

- Verificar cuáles cuentas pueden provision otras cuentas y de qué tipo.

## Cómo Probar

Determinar cuáles roles son capaces de provision usuarios y qué tipo de cuentas pueden provision.

- ¿Hay alguna verificación, vetting y autorización de peticiones de provisioning?
- ¿Hay alguna verificación, vetting y autorización de peticiones de de-provisioning?
- ¿Puede un administrador provision otros administradores o solo usuarios?
- ¿Puede un administrador o otro usuario provision cuentas con privilegios mayores que los suyos?
- ¿Puede un administrador o usuario de-provisionarse a sí mismos?
- ¿Cómo son gestionados los archivos o recursos owned por el usuario de-provisioned? ¿Son borrados? ¿Es transferido el acceso?

### Ejemplo

En WordPress, solo el nombre y dirección email de un usuario son requeridos para provision el usuario, como mostrado abajo:

![Agregar Usuario de WordPress](images/Wordpress_useradd.png)\
 *Figura 4.3.3-1: Agregar Usuario de WordPress*

De-provisioning de usuarios requiere que el administrador seleccione los usuarios a ser de-provisioned, seleccione Delete del menú dropdown (circulado) y entonces aplicando esta acción. El administrador es entonces presentado con un dialog box preguntando qué hacer con los posts del usuario (borrarlos o transferirlos).

![Auth y Usuarios de WordPress](images/Wordpress_authandusers.png)\
 *Figura 4.3.3-2: Auth y Usuarios de WordPress*

## Herramientas

Mientras el enfoque más thorough y preciso para completar esta prueba es conducirla manualmente, herramientas proxy HTTP podrían ser también útiles.