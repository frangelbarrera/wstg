# Pruebas de Inyección LDAP

|ID          |
|------------|
|WSTG-INPV-06|

## Resumen

El Protocolo Ligero de Acceso a Directorios (LDAP) se utiliza para almacenar información sobre usuarios, hosts y muchos otros objetos. La [inyección LDAP](https://wiki.owasp.org/index.php/LDAP_injection) es un ataque del lado del servidor, que podría permitir la divulgación, modificación o inserción de información sensible sobre usuarios y hosts representados en una estructura LDAP. Esto se hace manipulando parámetros de entrada que posteriormente se pasan a funciones internas de búsqueda, adición y modificación.

Una aplicación web podría utilizar LDAP para permitir a los usuarios autenticarse o buscar información de otros usuarios dentro de una estructura corporativa. El objetivo de los ataques de inyección LDAP es inyectar metacaracteres de filtros de búsqueda LDAP en una consulta que será ejecutada por la aplicación.

[Rfc2254](https://www.ietf.org/rfc/rfc2254.txt) define una gramática sobre cómo construir un filtro de búsqueda en LDAPv3 y extiende [Rfc1960](https://www.ietf.org/rfc/rfc1960.txt) (LDAPv2).

Un filtro de búsqueda LDAP se construye en notación polaca, también conocida como [notación prefija polaca](https://en.wikipedia.org/wiki/Polish_notation).

Esto significa que una condición de pseudocódigo en un filtro de búsqueda como esta:

`find("cn=John & userPassword=mypass")`

se representará como:

`find("(&(cn=John)(userPassword=mypass))")`

Las condiciones booleanas y agrupaciones de grupo en un filtro de búsqueda LDAP podrían aplicarse utilizando los siguientes metacaracteres:

| Metacar |  Significado              |
|----------|---------------------------|
| &        |  Y booleano               |
| \|       |  O booleano               |
| !        |  NO booleano              |
| =        |  Igual                    |
| ~=       |  Aproximado               |
| >=       |  Mayor que                |
| <=       |  Menor que                |
| *        |  Cualquier carácter       |
| ()       |  Paréntesis de agrupación |

Ejemplos más completos sobre cómo construir un filtro de búsqueda se pueden encontrar en el RFC relacionado.

Una explotación exitosa de una vulnerabilidad de inyección LDAP podría permitir al probador:

- Acceder a contenido no autorizado
- Evadir restricciones de la aplicación
- Recopilar información no autorizada
- Agregar o modificar objetos dentro de la estructura de árbol LDAP

## Objetivos de la Prueba

- Identificar puntos de inyección LDAP.
- Evaluar la gravedad de la inyección.

## Cómo Probar

### Ejemplo 1: Filtros de Búsqueda

Supongamos que tenemos una aplicación web que utiliza un filtro de búsqueda como el siguiente:

`searchfilter="(cn="+user+")"`

que se instancia mediante una solicitud HTTP como esta:

`https://www.example.com/ldapsearch?user=John`

Si el valor `John` se reemplaza con un `*`, enviando la solicitud:

`https://www.example.com/ldapsearch?user=*`

el filtro se verá como:

`searchfilter="(cn=*)"`

que coincide con cada objeto con un atributo 'cn' igual a cualquier cosa.

Si la aplicación es vulnerable a la inyección LDAP, mostrará algunos o todos los atributos del usuario, dependiendo del flujo de ejecución de la aplicación y los permisos del usuario LDAP conectado.

Un probador podría utilizar un enfoque de prueba y error, insertando en el parámetro `(`, `|`, `&`, `*` y los otros caracteres, para verificar la aplicación en busca de errores.

### Ejemplo 2: Inicio de Sesión

Si una aplicación web utiliza LDAP para verificar las credenciales del usuario durante el proceso de inicio de sesión y es vulnerable a la inyección LDAP, es posible eludir la verificación de autenticación inyectando una consulta LDAP siempre verdadera (de manera similar a la inyección SQL y XPATH).

Supongamos que una aplicación web utiliza un filtro para coincidir con el par usuario/contraseña LDAP.

`searchlogin= "(&(uid="+user+")(userPassword={MD5}"+base64(pack("H*",md5(pass)))+"))";`

Utilizando los siguientes valores:

```txt
user=*)(uid=*))(|(uid=*
pass=password
```

el filtro de búsqueda resultará en:

`searchlogin="(&(uid=*)(uid=*))(|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))";`

que es correcto y siempre verdadero. De esta manera, el probador obtendrá el estado de sesión iniciada como el primer usuario en el árbol LDAP.

## Herramientas

- [Softerra LDAP Browser](https://www.ldapadministrator.com)

## Referencias

- [Hoja de Referencia para la Prevención de Inyección LDAP](https://cheatsheetseries.owasp.org/cheatsheets/LDAP_Injection_Prevention_Cheat_Sheet.html)

### Documentos Técnicos

- [Sacha Faust: Inyección LDAP: ¿Son Vulnerables Tus Aplicaciones?](https://www.networkdls.com/articles/ldapinjection.pdf)
- [Documento IBM: Entendiendo LDAP](https://www.redbooks.ibm.com/redbooks/pdfs/sg244986.pdf)
- [RFC 1960: Una Representación de Cadena de Filtros de Búsqueda LDAP](https://www.ietf.org/rfc/rfc1960.txt)
- [Inyección LDAP](https://www.blackhat.com/presentations/bh-europe-08/Alonso-Parada/Whitepaper/bh-eu-08-alonso-parada-WP.pdf)