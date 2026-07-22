# Probar Permisos de Archivo

|ID          |
|------------|
|WSTG-CONF-09|

## Resumen

Cuando a un recurso se le da una configuración de permisos que proporciona acceso a un rango más amplio de actores que el requerido, podría llevar a la exposición de información sensible, o la modificación de ese recurso por partes no intencionadas. Esto es especialmente peligroso cuando el recurso está relacionado con configuración de programa, ejecución, o datos de usuario sensibles.

Un ejemplo claro sería un archivo ejecutable que puede ser ejecutado por usuarios no autorizados. Para otro ejemplo, considera información de cuenta o un valor de token usado para acceder a una API. Estos son cada vez más vistos en servicios web modernos y microservicios, y pueden ser almacenados en un archivo de configuración que tiene permisos legibles por el mundo por defecto tras instalación. Tales datos sensibles podrían ser expuestos ya sea por actores internos maliciosos dentro del sistema host o por atacantes remotos. Los últimos pueden haber comprometido el servicio a través de otras vulnerabilidades, mientras ganan solo privilegios de usuario normal.

## Objetivos de Prueba

- Revisar e identificar cualquier permiso de archivo rogue.

## Cómo Probar

En Linux, usar el comando `ls` para verificar los permisos de archivo. Alternativamente, `namei` también puede ser usado para listar recursivamente permisos de archivo.

`$ namei -l /PathToCheck/`

Los archivos y directorios que requieren prueba de permisos de archivo pueden incluir, pero no están limitados a, lo siguiente:

- Archivos/directorio web
- Archivos/directorio de configuración
- Archivos sensibles (datos encriptados, contraseña, clave)/directorio
- Archivos de log (logs de seguridad, logs de operación, logs de admin)/directorio
- Ejecutables (scripts, EXE, JAR, class, PHP, ASP)/directorio
- Archivos/directorio de base de datos
- Archivos/directorio temp
- Archivos/directorio de upload

## Remediación

Establecer los permisos de los archivos y directorios apropiadamente para que usuarios no autorizados no puedan acceder a recursos críticos.

## Herramientas

- [Windows AccessEnum](https://technet.microsoft.com/en-us/sysinternals/accessenum)
- [Windows AccessChk](https://technet.microsoft.com/en-us/sysinternals/accesschk)
- [Linux namei](https://linux.die.net/man/1/namei)

## Referencias

- [CWE-732: Asignación Incorrecta de Permisos para Recurso Crítico](https://cwe.mitre.org/data/definitions/732.html)