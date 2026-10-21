# Almacenamiento en la nube

|ID          |
|------------|
|WSTG-CONF-11|

## Resumen

Los servicios de almacenamiento en la nube (AWS S3, Azure Blob Storage, Google Cloud Storage y equivalentes) permiten a las aplicaciones web almacenar y acceder a objetos sin tener que ejecutar sus propios servidores de archivos. Sin embargo, una configuración incorrecta del control de acceso puede provocar la exposición de información sensible, la manipulación de datos o el acceso no autorizado. De forma predeterminada, el almacenamiento de los tres proveedores principales es privado y solo pueden acceder a él las identidades a las que se haya concedido acceso explícitamente, pero un bucket o contenedor, o los objetos individuales que contiene, puede hacerse público —de manera deliberada (y con un alcance demasiado amplio) o por error—, lo que permite que un usuario no autorizado lea, cargue o modifique datos.

Además del acceso público al nivel del bucket, esta categoría de problemas también abarca las configuraciones incorrectas de las políticas ACL/IAM que conceden a principales específicos un acceso más amplio del previsto; las debilidades de las URL firmadas o prefirmadas, que se usan para conceder acceso por tiempo limitado sin proporcionar credenciales completas; y la configuración del control de versiones y del ciclo de vida de los objetos, que puede permitir recuperar datos sensibles antiguos o que supuestamente se eliminaron, o trasladarlos a ubicaciones con controles de acceso menos estrictos.

## Objetivos de prueba

- Evaluar si la configuración del control de acceso de los servicios de almacenamiento (acceso público al nivel del bucket o contenedor, y ACL o políticas IAM al nivel del objeto o principal) está correctamente implementada.
- Evaluar si las URL firmadas o prefirmadas tienen el alcance y la duración limitados adecuados, y si se gestionan correctamente.
- Evaluar si la configuración del control de versiones y del ciclo de vida de los objetos podría exponer datos sensibles antiguos o que supuestamente se eliminaron.

## Cómo probar

Primero, identifica la URL del bucket o contenedor. En las pruebas de caja negra, suele encontrarse mediante referencias en respuestas HTTP (etiquetas `<img>`/`<script>`, paquetes de JavaScript o respuestas de API); en las pruebas de caja gris, puede encontrarse en la consola del proveedor, el código fuente o la documentación. A continuación, ejecuta las mismas tres pruebas en cada proveedor: listar (¿el bucket o contenedor expone su contenido sin autenticación?), obtener (¿se puede leer un objeto específico o cuya ubicación se haya adivinado?) y escribir (¿se puede cargar o sobrescribir un objeto arbitrario?). Prueba las operaciones de listar y obtener aunque todavía no se conozca el nombre del bucket; los nombres comunes o que se pueden adivinar se abordan en [Artefactos de almacenamiento de objetos en la nube](04-Review_Old_Backup_and_Unreferenced_Files_for_Sensitive_Information.md#cloud-object-storage-artifacts), en lugar de duplicarse aquí.

Las tablas siguientes muestran las pruebas de caja negra (`curl`, sin asumir autenticación) y las de caja gris (herramientas de línea de comandos del proveedor cuando se dispone de credenciales). Usa `curl` para probar el acceso sin autenticación; cuando las credenciales estén dentro del alcance, usa las CLI de los proveedores para confirmar directamente los permisos configurados.

### Amazon S3

Formatos de URL: estilo de host virtual (`https://<bucket>.s3.<region>.amazonaws.com/<key>`) o estilo de ruta (`https://s3.<region>.amazonaws.com/<bucket>/<key>`); el endpoint global heredado (sin región) también funciona en algunas regiones.

| Prueba | curl | AWS CLI |
|---|---|---|
| Listar | `curl "https://<bucket>.s3.amazonaws.com/"` | `aws s3 ls s3://<bucket>` |
| Obtener | `curl "https://<bucket>.s3.amazonaws.com/<key>"` | `aws s3 cp s3://<bucket>/<key> -` |
| Escribir | `curl -X PUT -d 'test' "https://<bucket>.s3.amazonaws.com/test.txt"` | `aws s3 cp test.txt s3://<bucket>/test.txt` |

Si la operación de listado sin autenticación tiene éxito, el cuerpo de la respuesta devuelve un XML `<ListBucketResult>`; si la escritura falla, el cuerpo de la respuesta o el error de la CLI muestra `AccessDenied`.

### Azure Blob Storage

Formato de URL: `https://<account-name>.blob.core.windows.net/<container-name>/<blob-name>`.

| Prueba | curl | Azure CLI (caja gris) |
|---|---|---|
| Listar | `curl "https://<account>.blob.core.windows.net/<container>?restype=container&comp=list"` | `az storage blob list --container-name <container>` |
| Obtener | `curl "https://<account>.blob.core.windows.net/<container>/<blob>"` | `az storage blob download --container-name <container> --name <blob>` |
| Escribir | `curl -X PUT -d 'test' -H "x-ms-blob-type: BlockBlob" -H "x-ms-version: <current>" "https://<account>.blob.core.windows.net/<container>/test.txt"` | `az storage blob upload --container-name <container> --name test.txt` |

Una respuesta `200` con un cuerpo XML `<EnumerationResults>` al probar el listado significa que el nivel de acceso público del contenedor es `Container` (lectura pública y listado), no `Private` ni `Blob` (lectura solo de nombres conocidos). `az storage container show-permission` confirma directamente el nivel configurado cuando se dispone de credenciales.

### Google Cloud Storage (GCS)

Formato de URL (endpoint público de estilo XML): `https://storage.googleapis.com/<bucket-name>/<object-name>`.

| Prueba | curl | gsutil (caja gris) |
|---|---|---|
| Listar | `curl "https://storage.googleapis.com/storage/v1/b/<bucket>/o"` | `gsutil ls gs://<bucket>` |
| Obtener | `curl "https://storage.googleapis.com/<bucket>/<object>"` | `gsutil cat gs://<bucket>/<object>` |
| Escribir | `curl -X POST --data-binary @test.txt "https://storage.googleapis.com/upload/storage/v1/b/<bucket>/o?uploadType=media&name=test.txt"` | `gsutil cp test.txt gs://<bucket>/test.txt` |

Una respuesta `200` con una lista de objetos JSON al probar el listado significa que está habilitado el listado público. Esto suele deberse a una vinculación IAM de `allUsers`/`allAuthenticatedUsers` o a una ACL heredada, y no al comportamiento predeterminado de GCS. `gsutil iam get gs://<bucket>` confirma directamente la vinculación cuando se dispone de credenciales.

En Windows, reemplaza las comillas simples (`'`) por comillas dobles (`"`) en los comandos `curl` anteriores.

### Configuraciones incorrectas de ACL y políticas IAM

Además de la opción sencilla entre público y privado, los tres proveedores permiten conceder acceso granular, que puede configurarse fácilmente con un alcance demasiado amplio:

- AWS S3: revisa las políticas de bucket y las ACL para detectar concesiones a `AllUsers`/`AuthenticatedUsers` (el equivalente de S3 a «público») y políticas de bucket que concedan permisos amplios `s3:*` o `s3:GetObject`/`s3:PutObject` a un principal comodín (`"Principal": "*"`) o a otra cuenta de AWS que no debería necesitar acceso. Confirma también que [S3 Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html) esté habilitado en el nivel de cuenta o bucket, salvo que se requiera explícitamente lo contrario: actúa como medida de protección incluso si una política individual está mal configurada. `aws s3api get-bucket-acl` y `aws s3api get-bucket-policy` muestran la configuración efectiva en las pruebas de caja gris.
- Azure Blob Storage: revisa el nivel de acceso público del contenedor (`Private`/`Blob`/`Container`), como se explicó anteriormente, y revisa por separado las políticas de Shared Access Signature (SAS) y las asignaciones de roles RBAC (`Storage Blob Data Reader/Contributor`) con alcance en la cuenta de almacenamiento, para detectar concesiones más amplias de lo necesario.
- GCS: revisa las vinculaciones IAM (`gsutil iam get gs://<bucket>`) y las ACL heredadas por objeto o bucket para detectar vinculaciones con `allUsers` (público) o `allAuthenticatedUsers` (cualquier cuenta de Google, no solo los usuarios de esta aplicación), ya que a menudo se confunde esta última con «todavía privado», cuando no lo es.

En los tres casos, distingue los objetos que son públicos por diseño (por ejemplo, recursos de sitios estáticos o descargas públicas) de los objetos de configuración, copias de seguridad o datos de usuarios que quedaron expuestos involuntariamente: para determinar si un hallazgo es real, importa la finalidad del objeto, no solo que sea accesible.

### Debilidades de las URL firmadas o prefirmadas

Las URL firmadas (URL prefirmadas de S3, tokens SAS de Azure y URL firmadas de GCS) permiten que una aplicación conceda acceso por tiempo limitado a un objeto privado sin compartir credenciales completas. Revisa lo siguiente:

- Vencimiento: confirma que el vencimiento establecido en la URL firmada (`X-Amz-Expires` para S3, `se=` para Azure SAS, `Expires`/`X-Goog-Expires` para GCS) sea tan breve como permita el caso de uso. Si se filtra una URL firmada con un vencimiento de varios días o años, equivale en la práctica a una credencial permanente (por ejemplo, si aparece en el historial del navegador, encabezados `referrer`, registros del proxy o de acceso, o si se comparte o reenvía).
- Alcance: confirma que la firma se limite al objeto y la acción específicos necesarios (por ejemplo, las firmas de GCS/S3 pueden limitarse a un solo objeto y método HTTP; los tokens SAS de Azure pueden abarcar toda una cuenta si se generan como SAS de nivel de cuenta en vez de nivel de servicio, contenedor o blob: comprueba qué nivel se utilizó).
- Filtración y reutilización: comprueba si los proxies intermedios, las CDN o los registros de la aplicación almacenan las URL firmadas en texto sin cifrar, y si se reutiliza la misma URL firmada entre varios usuarios o sesiones en vez de generarla para cada solicitud. Una URL filtrada es válida para cualquiera hasta que venza o se revoque, y la mayoría de los esquemas no permiten revocar anticipadamente una URL firmada individual sin rotar la clave o credencial subyacente.
- Validación de la firma: como comprobación básica de manipulación, intenta modificar la ruta del objeto o los parámetros de consulta de una URL firmada válida (por ejemplo, apuntándola a otro objeto y manteniendo la misma firma) y confirma que se rechace la solicitud en lugar de entregar el objeto.

### Problemas de control de versiones y ciclo de vida

El control de versiones de objetos (que conserva versiones anteriores de un objeto después de que se sobrescriba o elimine) y las políticas de ciclo de vida (reglas que trasladan o vencen objetos con el tiempo) suelen habilitarse por motivos de durabilidad o cumplimiento, pero pueden frustrar la expectativa de que los datos «eliminados» hayan desaparecido realmente:

- Si el control de versiones está habilitado, confirma que un objeto «eliminado» (por ejemplo, uno que contiene datos que deberían borrarse por motivos de cumplimiento o un archivo antiguo con credenciales que se rotaron) no siga pudiéndose recuperar mediante un ID de versión anterior. Para S3: `aws s3api list-object-versions --bucket <bucket>`; para GCS: `gsutil ls -a gs://<bucket>/<object>`; para Azure: instantáneas y versiones de blobs mediante `az storage blob list --include v`.
- Confirma que la configuración del control de acceso se aplique de forma coherente a todas las versiones: una corrección de control de acceso aplicada al bucket o contenedor después de un incidente puede afectar solo a la versión actual y dejar accesibles las versiones anteriores del mismo objeto si el proveedor gestiona por separado el acceso a las versiones.
- Revisa las reglas del ciclo de vida que trasladan objetos a otra clase o nivel de almacenamiento, o a otro bucket o cuenta (por ejemplo, niveles de archivado o replicación entre regiones), y confirma que el destino tenga controles de acceso equivalentes. Una transición del ciclo de vida puede trasladar silenciosamente datos sensibles a una ubicación que no se incluyó en la revisión original de controles de acceso.
- Cuando una aplicación dependa de la opción «eliminar» para borrar datos sensibles (por ejemplo, ante una solicitud de eliminación de datos de un usuario), confirma que esto los elimine realmente —incluidas todas las versiones y las copias replicadas o trasladadas por el ciclo de vida— y no solo que quite la accesibilidad de la versión actual.

## Herramientas

- [AWS CLI](https://aws.amazon.com/cli/)
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/) / [Azure Storage Explorer](https://azure.microsoft.com/en-us/products/storage/storage-explorer)
- [gcloud storage](https://docs.cloud.google.com/sdk/gcloud/reference/storage) (Google Cloud CLI; [gsutil](https://cloud.google.com/storage/docs/gsutil) heredado)
- [S3Scanner](https://github.com/sa7mon/S3Scanner) - detección de buckets y enumeración de permisos
- [cloud_enum](https://github.com/initstring/cloud_enum) - enumeración de almacenamiento en varias nubes (AWS/Azure/GCP)
- [ScoutSuite](https://github.com/nccgroup/ScoutSuite) / [Prowler](https://github.com/prowler-cloud/prowler) - revisión más amplia de la postura de seguridad en la nube, incluidos hallazgos de ACL/IAM de almacenamiento

## Referencias

- [Trabajar con buckets de Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html)
- [AWS: Bloquear el acceso público al almacenamiento S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
- [Azure: Configurar el acceso público anónimo de lectura para contenedores y blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/anonymous-read-access-configure)
- [Azure: Conceder acceso limitado con Shared Access Signatures (SAS)](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)
- [Google Cloud Storage: Control de acceso](https://cloud.google.com/storage/docs/access-control)
- [Google Cloud Storage: URL firmadas](https://docs.cloud.google.com/storage/docs/access-control/signed-urls)
- [flAWS 2 - Aprender seguridad de AWS](http://flaws2.cloud)
- [Tutorial de curl](https://curl.se/docs/manual.html)