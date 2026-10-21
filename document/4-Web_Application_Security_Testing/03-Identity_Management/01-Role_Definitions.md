# Definiciones de roles

|ID          |
|------------|
|WSTG-IDNT-01|

## Resumen

Las aplicaciones ofrecen varios tipos de funcionalidades y servicios, que requieren permisos de acceso basados en las necesidades del usuario. Ese usuario podría ser:

- Un administrador, que gestiona las funcionalidades de la aplicación.
- Un auditor, que revisa las transacciones de la aplicación y proporciona un informe detallado.
- Un ingeniero de soporte, que ayuda a los clientes a depurar y solucionar problemas en sus cuentas.
- Un cliente, que interactúa con la aplicación y se beneficia de sus servicios.

Para gestionar estos usos y cualquier otro caso de uso de la aplicación, se configuran definiciones de roles (más comúnmente conocidas como [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control)). En función de estos roles, el usuario puede realizar la tarea requerida.

## Objetivos de la prueba

- Identificar y documentar los roles utilizados por la aplicación.
- Intentar cambiar, modificar o acceder a otro rol.
- Revisar la granularidad de los roles y las necesidades que justifican los permisos otorgados.

## Cómo probar

### Identificación de roles

El evaluador debe comenzar por identificar los roles de la aplicación que se van a probar mediante cualquiera de los siguientes métodos:

- Documentación de la aplicación.
- Orientación de los desarrolladores o administradores de la aplicación.
- Comentarios de la aplicación.
- Probar posibles roles mediante fuzzing:
    - Variable de cookie (*p. ej.*, `role=admin`, `isAdmin=True`)
    - Variable de cuenta (*p. ej.*, `Role: manager`)
    - Directorios o archivos ocultos (*p. ej.*, `/admin`, `/mod`, `/backups`)
    - Cambiar a usuarios conocidos (*p. ej.*, `admin`, `backups`, etc.)

### Cambiar a roles disponibles

Después de identificar posibles vectores de ataque, el evaluador debe probar y validar si puede acceder a los roles disponibles.

> Nota: Algunas aplicaciones definen los roles del usuario al crearlo, mediante verificaciones y políticas rigurosas, o se aseguran de que el rol del usuario esté debidamente protegido mediante una firma creada por el backend. El hecho de que existan roles no significa que haya una vulnerabilidad.

### Revisar los permisos de los roles

Después de obtener acceso a los roles del sistema, el evaluador debe comprender los permisos asignados a cada rol.

Por ejemplo, en general, un ingeniero de soporte no debería poder realizar funciones administrativas, gestionar las copias de seguridad ni llevar a cabo transacciones en nombre de un usuario.

Los usuarios tampoco deberían poder realizar acciones que aumenten sus propios privilegios (como asignarse a sí mismos un rol con más privilegios), salvo en circunstancias muy específicas.

Un administrador no debería tener poderes absolutos en el sistema. Las funciones administrativas sensibles deberían aplicar el principio de doble control (maker-checker) o usar MFA para garantizar que el administrador está llevando a cabo la transacción. Un ejemplo claro de ello fue el [incidente de Twitter de 2020](https://blog.x.com/en_us/topics/company/2020/an-update-on-our-security-incident).

## Herramientas

Las pruebas mencionadas anteriormente se pueden realizar sin utilizar ninguna herramienta, salvo la que se use para acceder al sistema.

Para facilitar las pruebas y documentarlas mejor, se pueden usar:

- [Extensión Autorize de Burp](https://github.com/Quitten/Autorize)
- [Complemento de pruebas de control de acceso de ZAP](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)

## Referencias

- [Ingeniería de roles y estándares RBAC](https://csrc.nist.gov/projects/role-based-access-control#rbac-standard)