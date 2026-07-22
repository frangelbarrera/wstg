# Pruebas de credenciales predeterminadas

|ID          |
|------------|
|WSTG-ATHN-02|

## Resumen

Muchas aplicaciones web y dispositivos de hardware tienen contraseñas predeterminadas para la cuenta administrativa integrada. Aunque en algunos casos estas pueden generarse aleatoriamente, a menudo son estáticas, lo que significa que pueden ser fácilmente adivinadas u obtenidas por un atacante.

Además, cuando se crean nuevos usuarios en las aplicaciones, estos pueden tener contraseñas predefinidas. Estas podrían generarse automáticamente por la aplicación o crearse manualmente por el personal. En ambos casos, si no se generan de manera segura, las contraseñas pueden ser posibles de adivinar por un atacante.

## Objetivos de la prueba

- Determinar si la aplicación tiene alguna cuenta de usuario con contraseñas predeterminadas.
- Revisar si las nuevas cuentas de usuario se crean con contraseñas débiles o predecibles.

## Cómo probar

### Pruebas de credenciales predeterminadas del proveedor

El primer paso para identificar contraseñas predeterminadas es identificar el software que se está utilizando. Esto se cubre en detalle en la sección [Recopilación de información](../01-Information_Gathering/README.md) de la guía.

Una vez identificado el software, intenta encontrar si utiliza contraseñas predeterminadas, y si es así, cuáles son. Esto debería incluir:

- Buscar "[SOFTWARE] default password".
- Revisar el manual o la documentación del proveedor.
- Verificar bases de datos comunes de contraseñas predeterminadas, como [CIRT.net](https://cirt.net/passwords), [SecLists Default Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials) o [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv).
- Inspeccionar el código fuente de la aplicación (si está disponible).
- Instalar la aplicación en una máquina virtual e inspeccionarla.
- Inspeccionar el hardware físico en busca de pegatinas (a menudo presentes en dispositivos de red).

Si no se puede encontrar una contraseña predeterminada, intenta opciones comunes como:

- "admin", "password", "12345", u otras [contraseñas predeterminadas comunes](https://github.com/nixawk/fuzzdb/blob/master/bruteforce/passwds/default_devices_users%2Bpasswords.txt).
- Una contraseña vacía o en blanco.
- El número de serie o la dirección MAC del dispositivo.

Si el nombre de usuario es desconocido, hay varias opciones para enumerar usuarios, discutidas en la guía [Pruebas de enumeración de cuentas](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md). Alternativamente, intenta opciones comunes como "admin", "root" o "system".

### Pruebas de contraseñas predeterminadas de la organización

Cuando el personal dentro de una organización crea manualmente contraseñas para nuevas cuentas, puede hacerlo de manera predecible. Esto a menudo puede ser:

- Una sola contraseña común como "Password1".
- Detalles específicos de la organización, como el nombre de la organización o la dirección.
- Contraseñas que siguen un patrón simple, como "Monday123" si la cuenta se crea un lunes.

Estos tipos de contraseñas a menudo son difíciles de identificar desde una perspectiva de caja negra, a menos que puedan adivinarse o forzarse con éxito. Sin embargo, son fáciles de identificar al realizar pruebas de caja gris o caja blanca.

### Pruebas de contraseñas predeterminadas generadas por la aplicación

Si la aplicación genera automáticamente contraseñas para nuevas cuentas de usuario, estas también pueden ser predecibles. Para probar estas, crea múltiples cuentas en la aplicación con detalles similares al mismo tiempo, y compara las contraseñas que se les dan.

Las contraseñas pueden basarse en:

- Una sola cadena estática compartida entre cuentas.
- Una parte hashada u ofuscada de los detalles de la cuenta, como `md5($username)`.
- Un algoritmo basado en tiempo.
- Un generador de números pseudoaleatorios débil (PRNG).

Este tipo de problema a menudo es difícil de identificar desde una perspectiva de caja negra.

## Herramientas

- [Burp Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Nikto 2](https://www.cirt.net/nikto2)
- [Nuclei](https://github.com/projectdiscovery/nuclei)
    - [Default Login - Nuclei Templates](https://github.com/projectdiscovery/nuclei-templates/tree/6b26c63d8f63b2a812a478f14c4c098b485d54b4/http/default-logins)

## Referencias

- [CIRT](https://cirt.net/passwords)
- [SecLists Default Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials)
- [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv)