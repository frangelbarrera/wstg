# Permisos de archivos

|ID          |
|------------|
|WSTG-CONF-09|

## Resumen

Cuando se asigna a un recurso una configuración de permisos que proporciona acceso a un conjunto de actores más amplio del necesario, esto podría provocar la exposición de información sensible o la modificación de ese recurso por partes no autorizadas. Esto es especialmente peligroso cuando el recurso está relacionado con la configuración o ejecución de programas, o con datos sensibles de usuarios.

Un ejemplo claro sería un archivo ejecutable que pueden ejecutar usuarios no autorizados. Otro ejemplo sería la información de una cuenta o un valor de token utilizado para acceder a una API. Estos elementos son cada vez más comunes en los servicios web modernos y los microservicios, y pueden almacenarse en un archivo de configuración que, de forma predeterminada, tiene permisos de lectura para todos después de la instalación. Estos datos sensibles podrían quedar expuestos por actores internos maliciosos en el sistema host o por atacantes remotos. Estos últimos podrían haber comprometido el servicio mediante otras vulnerabilidades, aunque solo tengan privilegios de usuario normales.

## Objetivos de las pruebas

- Revisar e identificar permisos de archivo no deseados en el sistema de archivos del host, en imágenes y sistemas de archivos de contenedores y en objetos de almacenamiento en la nube.
- Evaluar si los montajes de volúmenes de Kubernetes y la configuración de `securityContext` hacen cumplir el principio de mínimo privilegio para el acceso a archivos y secretos.
- Identificar secretos almacenados con permisos demasiado amplios, independientemente de dónde se encuentren (sistema de archivos, imagen de contenedor, almacén de secretos de orquestación, exportación de un gestor de secretos en la nube o artefactos de CI).

## Cómo realizar las pruebas

### Permisos tradicionales de SO/sistema de archivos

En Linux, use el comando `ls` para comprobar los permisos de los archivos. Como alternativa, también se puede usar `namei` para listar recursivamente los permisos de los archivos.

`$ namei -l /PathToCheck/`

Los archivos y directorios que requieren pruebas de permisos pueden incluir, entre otros, los siguientes:

- Archivos/directorio web
- Archivos/directorio de configuración
- Archivos sensibles (datos cifrados, contraseñas, claves)/directorio
- Archivos de registro (registros de seguridad, operación y administración)/directorio
- Ejecutables (scripts, EXE, JAR, class, PHP, ASP)/directorio
- Archivos/directorio de base de datos
- Archivos temporales/directorio
- Archivos cargados/directorio

### Permisos del sistema de archivos de contenedores

Las imágenes de contenedor suelen incorporar permisos demasiado amplios en archivos que terminan siendo propiedad de `root` o que permiten escritura o lectura a todos dentro del contenedor en ejecución:

- Inspeccione la compilación de la imagen en busca de instrucciones que amplíen los permisos: `docker history --no-trunc <image>` y revise el `Dockerfile` para detectar `COPY`/`ADD` sin un `chmod`/`chown` posterior, o `RUN chmod -R 777 ...` utilizado para evitar errores de permisos.
- Desde un contenedor en ejecución (o mediante `docker exec`), compruebe si hay archivos con permisos de escritura para todos fuera de las rutas temporales previstas: `find / -perm -0002 -type f 2>/dev/null`. En este caso, omita `-xdev`, a diferencia de lo que se hace en un análisis del host, ya que los archivos sensibles suelen estar en un volumen/PVC montado y no en el sistema de archivos raíz propio del contenedor. Los permisos de un volumen montado los establece el host o el orquestador en el momento del montaje y pueden diferir de los permisos especificados por las propias capas de la imagen para esa ruta, o prevalecer sobre ellos. Por eso, ambos deben comprobarse por separado: los permisos de las capas de la imagen mediante `docker history` o la inspección de capas (véase arriba), y los permisos del volumen montado mediante esta ejecución de `find` en el contenedor activo.
- Compruebe si el contenedor se ejecuta como `root` de forma predeterminada (`docker inspect --format '{{.Config.User}}' <image>` vacío o `0` significa que se ejecuta como `root`). Un proceso `root` combinado con una ruta del host montada con permisos de escritura es una vía habitual para la escalada de privilegios.
- Compruebe si hay secretos o configuraciones incorporados en las capas de la imagen con modos permisivos: use `docker history` y extraiga las capas (por ejemplo, mediante `dive` o `docker save` + `tar`) para confirmar que archivos como `/app/.env` o material de claves no tengan permisos `644`/`666` cuando deberían tener permisos `600` y estar restringidos a `root`.

### Montajes de volúmenes de Kubernetes y securityContext

- Revise las especificaciones de Pod/Deployment en busca de `securityContext.fsGroup`, `runAsUser`, `runAsNonRoot` y `readOnlyRootFilesystem`. La ausencia de estos valores suele significar que los volúmenes y el sistema de archivos raíz pueden ser modificados por más procesos o usuarios de los necesarios.
- Compruebe que los volúmenes montados de `Secret` y `ConfigMap` tengan un `defaultMode` explícito (modo de archivo octal). El valor predeterminado de Kubernetes (`0644`) hace que los archivos de secretos sean legibles por el grupo y por todos los usuarios dentro del pod. Confirme que los montajes sensibles establezcan un modo más restrictivo (por ejemplo, `0400` o `0440`) mediante `defaultMode` o `mode` para cada elemento.
- En el caso de los volúmenes `hostPath`, confirme los permisos propios del directorio del host montado y que el UID/GID del contenedor (según `securityContext`) no otorgue en el host un acceso más amplio del previsto.
- Revise `automountServiceAccountToken` y compruebe si el volumen proyectado del token de la cuenta de servicio tiene permisos de lectura o de montaje más amplios de lo que necesita la carga de trabajo.
- `kubectl auth can-i --list --as=system:serviceaccount:<ns>:<sa>` ayuda a confirmar que el acceso efectivo asociado a la identidad de un pod no sea más amplio de lo que indican los permisos de archivos y volúmenes.

#### Lista de comprobación de SecurityContext y refuerzo de contenedores

Los campos anteriores son el subconjunto específico de permisos de una revisión más amplia de `securityContext` y del refuerzo de contenedores. La siguiente lista de comprobación abarca los campos que con mayor frecuencia mantienen su valor predeterminado inseguro y sirve como referencia compacta para las pruebas de contenedores y Kubernetes de esta sección y de [Configuración de la plataforma de aplicaciones](02-Application_Platform_Configuration.md#container-images):

| Campo/configuración | Valor predeterminado inseguro | Qué comprobar |
|---|---|---|
| `runAsNonRoot` / `USER` del contenedor | Sin definir; el contenedor se ejecuta como `root` (UID 0) | Que la especificación del pod establezca `securityContext.runAsNonRoot: true` y `runAsUser` con un UID distinto de cero, o que `docker inspect --format '{{.Config.User}}' <image>` devuelva un valor no vacío y distinto de cero. |
| `readOnlyRootFilesystem` | Sin definir; el sistema de archivos raíz permite escritura | Que la especificación del pod establezca `readOnlyRootFilesystem: true` y que cualquier ruta en la que el proceso realmente necesite escribir se monte como volumen independiente. |
| `allowPrivilegeEscalation` | Sin definir; de forma predeterminada se permite la escalada | Que la especificación del pod establezca `allowPrivilegeEscalation: false`. |
| `privileged` | Sin definir; el contenedor no tiene privilegios de forma predeterminada, pero debe comprobarse si se habilitó explícitamente | Confirme que no se haya establecido `privileged: true`, salvo que sea realmente necesario (algo poco frecuente en las cargas de trabajo de aplicaciones). |
| Capacidades de Linux | El contenedor conserva el conjunto predeterminado completo de capacidades | Que la especificación del pod establezca `capabilities.drop: ["ALL"]` y que solo se vuelvan a añadir mediante `capabilities.add` las capacidades específicas que necesite la carga de trabajo. |
| `seccompProfile` | Sin definir en clústeres antiguos; sin restricciones | Que `securityContext.seccompProfile.type: RuntimeDefault` esté configurado para el pod o el contenedor (o que se use un perfil personalizado más restrictivo). |
| Espacios de nombres del host (`hostNetwork`, `hostPID`, `hostIPC`) | Sin definir; los espacios de nombres están aislados de forma predeterminada, pero debe comprobarse si se habilitaron explícitamente | Confirme que ninguno de estos valores esté establecido en `true`, salvo que la carga de trabajo realmente necesite acceso al host. |
| `defaultMode` del volumen / permisos de las capas de la imagen | Tratados arriba en los apartados sobre permisos del sistema de archivos y de los volúmenes | Véanse [Permisos del sistema de archivos de contenedores](#container-filesystem-permissions) y [Montajes de volúmenes de Kubernetes y securityContext](#kubernetes-volume-mounts-and-securitycontext), más arriba. |

Al igual que con las demás comprobaciones de este documento, considere que una configuración ausente o insegura es un posible hallazgo que debe confirmarse según el comportamiento real de la carga de trabajo. Por ejemplo, conviene señalar una capacidad innecesaria que el proceso en ejecución no utilice realmente, pero demostrar que es explotable —por ejemplo, mediante una salida del contenedor que aproveche una capacidad conservada— constituye un hallazgo más sólido que limitarse a observar la configuración.

### ACL de objetos en la nube e IAM

Las configuraciones incorrectas del almacenamiento en la nube son un análogo directo de los permisos de archivos y deben someterse a pruebas de la misma manera:

- AWS S3: compruebe las ACL de los buckets y objetos, y las políticas de los buckets, para detectar concesiones de acceso públicas o demasiado amplias: `aws s3api get-bucket-acl --bucket <name>`, `aws s3api get-bucket-policy --bucket <name>`. Confirme que la configuración S3 Block Public Access esté habilitada, salvo que se requiera explícitamente lo contrario.
- Azure Blob Storage: compruebe el nivel de acceso público del contenedor (`Private`/`Blob`/`Container`) y los tokens SAS con un alcance excesivo o una caducidad demasiado lejana: `az storage container show-permission`.
- GCP Cloud Storage: compruebe las vinculaciones de IAM de buckets y objetos, así como las ACL heredadas, para detectar `allUsers`/`allAuthenticatedUsers`: `gsutil iam get gs://<bucket>`.
- En todos los casos, distinga entre el acceso de lectura público previsto (por ejemplo, para recursos de un sitio estático) y la exposición no deseada de configuraciones, copias de seguridad o credenciales; se aplica la misma distinción que al revisar los permisos de archivos de un sistema de archivos.

### Secretos almacenados con permisos demasiado amplios

Independientemente de dónde se encuentren los secretos (sistema de archivos, imagen de contenedor, `Secret` de Kubernetes, exportación de un gestor de secretos en la nube o artefacto de CI), verifique lo siguiente:

- Los secretos basados en archivos (`.env`, claves privadas, JSON de cuentas de servicio, kubeconfig) no sean legibles por el grupo ni por todos los usuarios (`600`/`400`) y que solo sean propiedad del usuario o la cuenta de servicio previstos.
- Los secretos no estén incorporados en las capas de imágenes de contenedor ni sean accesibles mediante `docker history` o la inspección de capas, aunque se hayan eliminado en una capa posterior.
- Los secretos incorporados a artefactos o registros de compilación de CI no terminen en cachés de compilación legibles por todos ni en almacenamiento de artefactos con ACL predeterminadas (a menudo permisivas).

## Remediación

- Establezca correctamente los permisos de archivos y directorios para impedir que usuarios no autorizados accedan a recursos críticos y aplique el principio de mínimo privilegio en el host, en las imágenes de contenedor y en la capa de orquestación.
- En los contenedores, evite ejecutarlos como `root` y evite soluciones alternativas con permisos demasiado amplios, como `chmod -R 777`; corrija en su lugar la incompatibilidad subyacente de propiedad/UID.
- En Kubernetes, configure `securityContext` (`runAsNonRoot`, `readOnlyRootFilesystem`, `fsGroup`) y un `defaultMode` restrictivo en los volúmenes de `Secret`/`ConfigMap`.
- En el almacenamiento en la nube, habilite de forma predeterminada el bloqueo del acceso público y conceda acceso mediante roles o políticas de IAM con alcance limitado, en lugar de ACL amplias o tokens SAS de larga duración y amplio alcance.

## Herramientas

- [Windows AccessEnum](https://learn.microsoft.com/en-us/sysinternals/downloads/accessenum)
- [Windows AccessChk](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk)
- [Linux namei](https://linux.die.net/man/1/namei)
- [dive - Explorador de capas y permisos de imágenes Docker](https://github.com/wagoodman/dive)
- [kube-bench - Analizador del CIS Kubernetes Benchmark](https://github.com/aquasecurity/kube-bench)
- [ScoutSuite - Auditoría de seguridad multinube (AWS/Azure/GCP)](https://github.com/nccgroup/ScoutSuite)
- [Prowler - Evaluación de seguridad de AWS/Azure/GCP](https://github.com/prowler-cloud/prowler)

## Referencias

- [CWE-732: Asignación incorrecta de permisos para un recurso crítico](https://cwe.mitre.org/data/definitions/732.html)
- [Kubernetes: Configurar un contexto de seguridad para un Pod o contenedor](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Kubernetes: Secretos - defaultMode](https://kubernetes.io/docs/concepts/configuration/secret/#secret-files-permissions)
- [AWS: Bloquear el acceso público al almacenamiento de S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)