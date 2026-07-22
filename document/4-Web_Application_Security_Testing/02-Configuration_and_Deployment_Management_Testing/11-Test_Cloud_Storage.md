# Probar Almacenamiento en la Nube

|ID          |
|------------|
|WSTG-CONF-11|

## Resumen

Los servicios de almacenamiento en la nube permiten a las aplicaciones web y servicios almacenar y acceder a objetos en el servicio de almacenamiento. La mala configuración del control de acceso, sin embargo, podría llevar a la exposición de información sensible, manipulación de datos, o acceso no autorizado.

Un ejemplo conocido es donde un bucket de Amazon S3 está mal configurado, aunque los otros servicios de almacenamiento en la nube también podrían estar expuestos a riesgos similares. Por defecto, todos los buckets de S3 son privados y pueden ser accedidos solo por usuarios a quienes se les ha otorgado acceso explícitamente. Los usuarios pueden otorgar acceso público no solo al bucket mismo sino también a objetos individuales almacenados dentro de ese bucket. Esto podría llevar a que un usuario no autorizado sea capaz de subir nuevos archivos, modificar o leer archivos almacenados.

## Objetivos de Prueba

- Evaluar que la configuración de control de acceso para los servicios de almacenamiento esté propiamente en su lugar.

## Cómo Probar

Primero, identificar la URL para acceder a los datos en el servicio de almacenamiento, y luego considerar las siguientes pruebas:

- Leer datos no autorizados
- Subir un nuevo archivo arbitrario

Se puede usar curl para las pruebas con los siguientes comandos y ver si las acciones no autorizadas pueden realizarse exitosamente.

Para probar la capacidad de leer un objeto:

```bash
curl -X GET https://<cloud-storage-service>/<object>
```

Para probar la capacidad de subir un archivo:

```bash
curl -X PUT -d 'test' 'https://<cloud-storage-service>/test.txt'
```

En el comando anterior, se recomienda reemplazar las comillas simples (') con comillas dobles (") cuando se ejecute el comando en una máquina Windows.

### Pruebas de Mala Configuración de Amazon S3 Bucket

Las URLs de buckets de Amazon S3 siguen uno de dos formatos, ya sea estilo virtual host o estilo path.

- Acceso Estilo Virtual Hosted

```text
https://bucket-name.s3.Region.amazonaws.com/key-name
```

En el siguiente ejemplo, `my-bucket` es el nombre del bucket, `us-west-2` es la región, y `puppy.png` es el key-name:

```text
https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png
```

- Acceso Estilo Path

```text
https://s3.Region.amazonaws.com/bucket-name/key-name
```

Como arriba, en el siguiente ejemplo, `my-bucket` es el nombre del bucket, `us-west-2` es la región, y `puppy.png` es el key-name:

```text
https://s3.us-west-2.amazonaws.com/my-bucket/puppy.png
```

Para algunas regiones, se puede usar el endpoint global heredado que no especifica un endpoint específico de región. Su formato es también ya sea estilo virtual hosted o estilo path.

- Acceso Estilo Virtual Hosted

```text
https://bucket-name.s3.amazonaws.com
```

- Acceso Estilo Path

```text
https://s3.amazonaws.com/bucket-name
```

#### Identificar URL del Bucket

Para pruebas de caja negra, las URLs de S3 pueden encontrarse en los mensajes HTTP. El siguiente ejemplo muestra una URL de bucket enviada en la etiqueta `img` en una respuesta HTTP.

```html
...
<img src="https://my-bucket.s3.us-west-2.amazonaws.com/puppy.png">
...
```

Para pruebas de caja gris, se pueden obtener URLs de bucket desde la interfaz web de Amazon, documentos, código fuente, y cualquier otra fuente disponible.

#### Pruebas con AWS-CLI

Además de probar con curl, también se puede probar con la herramienta de línea de comandos de AWS. En este caso se usa el esquema de URI `s3://`.

##### Listar

El siguiente comando lista todos los objetos del bucket cuando está configurado público:

```bash
aws s3 ls s3://<bucket-name>
```

##### Subir

El siguiente es el comando para subir un archivo:

```bash
aws s3 cp arbitrary-file s3://bucket-name/path-to-save
```

Este ejemplo muestra el resultado cuando la subida ha sido exitosa.

```bash
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload: ./test.txt to s3://bucket-name/test.txt
```

Este ejemplo muestra el resultado cuando la subida ha fallado.

```bash
$ aws s3 cp test.txt s3://bucket-name/test.txt
upload failed: ./test2.txt to s3://bucket-name/test2.txt An error occurred (AccessDenied) when calling the PutObject operation: Access Denied
```

##### Remover

El siguiente es el comando para remover un objeto:

```bash
aws s3 rm s3://bucket-name/object-to-remove
```

## Herramientas

- [AWS CLI](https://aws.amazon.com/cli/)

## Referencias

- [Working with Amazon S3 Buckets](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html)
- [flAWS 2 - Learn AWS Security](http://flaws2.cloud)
- [curl Tutorial](https://curl.se/docs/manual.html)
