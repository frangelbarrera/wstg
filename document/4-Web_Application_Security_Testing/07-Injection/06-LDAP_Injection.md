# Inyección LDAP

|ID          |
|------------|
|WSTG-INJT-06|

## Resumen

El Protocolo Ligero de Acceso a Directorios (LDAP) se utiliza para almacenar información sobre usuarios, hosts y muchos otros objetos. La [inyección LDAP](https://community.owasp.org/attacks/LDAP_Injection) es un ataque del lado del servidor que podría permitir divulgar, modificar o insertar información sensible sobre usuarios y hosts representados en una estructura LDAP. Esto se logra manipulando parámetros de entrada que posteriormente se pasan a funciones internas de búsqueda, adición y modificación.

Una aplicación web podría utilizar LDAP para permitir que los usuarios se autentiquen o busquen información de otros usuarios dentro de una estructura corporativa. El objetivo de los ataques de inyección LDAP es inyectar metacaracteres de filtros de búsqueda LDAP en una consulta que ejecutará la aplicación.

[Rfc2254](https://www.ietf.org/rfc/rfc2254.txt) define una gramática para construir un filtro de búsqueda en LDAPv3 y amplía [Rfc1960](https://www.ietf.org/rfc/rfc1960.txt) (LDAPv2).

Un filtro de búsqueda LDAP se construye en notación polaca, también conocida como [notación prefija polaca](https://en.wikipedia.org/wiki/Polish_notation).

Esto significa que una condición de pseudocódigo en un filtro de búsqueda como esta:

`find("cn=John & userPassword=mypass")`

se representará como:

`find("(&(cn=John)(userPassword=mypass))")`

Las condiciones booleanas y las agrupaciones en un filtro de búsqueda LDAP pueden aplicarse mediante los siguientes metacaracteres:

| Metacarácter |  Significado              |
|----------|-----------------------|
| &        |  AND booleano          |
| \|       |  OR booleano           |
| !        |  NOT booleano          |
| =        |  Igual a               |
| ~=       |  Aproximadamente igual |
| >=       |  Mayor que             |
| <=       |  Menor que             |
| *        |  Cualquier carácter    |
| ()       |  Paréntesis de agrupación |

En el RFC relacionado se pueden encontrar ejemplos más completos sobre cómo construir un filtro de búsqueda.

La explotación exitosa de una vulnerabilidad de inyección LDAP podría permitir al probador:

- Acceder a contenido no autorizado
- Evadir las restricciones de la aplicación
- Recopilar información no autorizada
- Agregar o modificar objetos dentro de la estructura de árbol LDAP

## Objetivos de la prueba

- Identificar puntos de inyección LDAP.
- Evaluar la gravedad de la inyección.

## Cómo probar

### Ejemplo 1: Filtros de búsqueda

Supongamos que tenemos una aplicación web que utiliza un filtro de búsqueda como el siguiente:

`searchfilter="(cn="+user+")"`

que se instancia mediante una solicitud HTTP como esta:

`https://www.example.com/ldapsearch?user=John`

Si el valor `John` se reemplaza por un `*` al enviar la solicitud:

`https://www.example.com/ldapsearch?user=*`

el filtro se verá así:

`searchfilter="(cn=*)"`

que coincide con todos los objetos cuyo atributo 'cn' es igual a cualquier valor.

Si la aplicación es vulnerable a la inyección LDAP, mostrará algunos o todos los atributos del usuario, según el flujo de ejecución de la aplicación y los permisos del usuario LDAP conectado.

El probador podría utilizar un enfoque de prueba y error, insertando `(`, `|`, `&`, `*` y los demás caracteres en el parámetro para comprobar si se producen errores en la aplicación.

### Ejemplo 2: Inicio de sesión

Si una aplicación web utiliza LDAP para comprobar las credenciales del usuario durante el proceso de inicio de sesión y es vulnerable a la inyección LDAP, es posible eludir la comprobación de autenticación inyectando una consulta LDAP que siempre sea verdadera (de manera similar a la inyección SQL y XPATH).

Supongamos que una aplicación web utiliza un filtro para buscar el par de usuario y contraseña LDAP.

`searchlogin= "(&(uid="+user+")(userPassword={MD5}"+base64(pack("H*",md5(pass)))+"))";`

Utilizando los siguientes valores:

```txt
user=*)(uid=*))(|(uid=*
pass=password
```

el filtro de búsqueda resultará así:

`searchlogin="(&(uid=*)(uid=*))(|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))";`

lo cual es correcto y siempre verdadero. De esta manera, el probador obtendrá una sesión iniciada como el primer usuario del árbol LDAP.

## Herramientas

- [Softerra LDAP Browser](https://www.ldapadministrator.com)

## Referencias

- [Hoja de referencia para la prevención de la inyección LDAP](https://cheatsheetseries.owasp.org/cheatsheets/LDAP_Injection_Prevention_Cheat_Sheet.html)

### Documentos técnicos

- [Sacha Faust: Inyección LDAP: ¿Son vulnerables tus aplicaciones?](https://networkdls.com/File/Entity/article-archive/ldap-injection.pdf)
- [Documento de IBM: Entender LDAP](https://www.redbooks.ibm.com/redbooks/pdfs/sg244986.pdf)
- [RFC 1960: Representación de cadena de los filtros de búsqueda LDAP](https://www.ietf.org/rfc/rfc1960.txt)
- [Inyección LDAP](https://www.blackhat.com/presentations/bh-europe-08/Alonso-Parada/Whitepaper/bh-eu-08-alonso-parada-WP.pdf)