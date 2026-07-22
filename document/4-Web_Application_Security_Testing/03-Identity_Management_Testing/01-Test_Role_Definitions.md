# Prueba de Definiciones de Roles

|ID          |
|------------|
|WSTG-IDNT-01|

## Resumen

Las aplicaciones tienen varios tipos de funcionalidades y servicios, y requieren permisos de acceso basados en las necesidades del usuario. Ese usuario podría ser:

- un administrador, donde gestionan las funcionalidades de la aplicación.
- un auditor, donde revisan las transacciones de la aplicación y proporcionan un informe detallado.
- un ingeniero de soporte, donde ayudan a los clientes a depurar y solucionar problemas en sus cuentas.
- un cliente, donde interactúan con la aplicación y se benefician de sus servicios.

Para manejar estos usos y cualquier otro caso de uso para esa aplicación, se configuran definiciones de roles (más comúnmente conocidas como [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control)). Basado en estos roles, el usuario es capaz de realizar la tarea requerida.

## Objetivos de la Prueba

- Identificar y documentar los roles utilizados por la aplicación.
- Intentar cambiar, modificar o acceder a otro rol.
- Revisar la granularidad de los roles y las necesidades detrás de los permisos otorgados.

## Cómo Probar

### Identificación de Roles

El probador debe comenzar identificando los roles de la aplicación que se están probando a través de cualquiera de los siguientes métodos:

- Documentación de la aplicación.
- Orientación de los desarrolladores o administradores de la aplicación.
- Comentarios de la aplicación.
- Fuzz posibles roles:
    - variable de cookie (*e.g.* `role=admin`, `isAdmin=True`)
    - variable de cuenta (*e.g.* `Role: manager`)
    - directorios o archivos ocultos (*e.g.* `/admin`, `/mod`, `/backups`)
    - cambiar a usuarios conocidos (*e.g.* `admin`, `backups`, etc.)

### Cambiar a Roles Disponibles

Después de identificar posibles vectores de ataque, el probador necesita probar y validar que pueden acceder a los roles disponibles.

> Algunas aplicaciones definen los roles del usuario en la creación, a través de verificaciones rigurosas y políticas, o asegurando que el rol del usuario esté protegido adecuadamente a través de una firma creada por el backend. Encontrar que existen roles no significa que sean una vulnerabilidad.

### Revisar Permisos de Roles

Después de obtener acceso a los roles en el sistema, el probador debe entender los permisos proporcionados a cada rol.

Un ingeniero de soporte no debería poder realizar funcionalidades administrativas, gestionar las copias de seguridad o realizar cualquier transacción en lugar de un usuario.

Un administrador no debería tener poderes completos en el sistema. Las funcionalidades administrativas sensibles deberían aprovechar un principio de maker-checker, o usar MFA para asegurar que el administrador esté realizando la transacción. Un ejemplo claro de esto fue el [incidente de Twitter en 2020](https://blog.twitter.com/en_us/topics/company/2020/an-update-on-our-security-incident.html).

## Herramientas

Las pruebas mencionadas anteriormente se pueden realizar sin el uso de ninguna herramienta, excepto la que se usa para acceder al sistema.

Para hacer las cosas más fáciles y documentadas, se puede usar:

- [Extensión Autorize de Burp](https://github.com/Quitten/Autorize)
- [Complemento de Pruebas de Control de Acceso de ZAP](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)

## Referencias

- [Role Engineering for Enterprise Security Management, E Coyne & J Davis, 2007](https://www.bookdepository.co.uk/Role-Engineering-for-Enterprise-Security-Management-Edward-Coyne/9781596932180)
- [Role engineering and RBAC standards](https://csrc.nist.gov/projects/role-based-access-control#rbac-standard)