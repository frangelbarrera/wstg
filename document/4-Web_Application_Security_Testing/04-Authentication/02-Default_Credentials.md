# Credenciales predeterminadas

|ID          |
|------------|
|WSTG-ATHN-02|

## Resumen

Muchas aplicaciones web y dispositivos de hardware tienen contraseñas predeterminadas para la cuenta administrativa integrada. Aunque en algunos casos estas pueden generarse aleatoriamente, a menudo son estáticas, lo que significa que un atacante puede adivinarlas u obtenerlas fácilmente.

Además, al crear nuevos usuarios en las aplicaciones, es posible que se les asignen contraseñas predefinidas. Estas pueden generarse automáticamente mediante la aplicación o crearse manualmente por el personal. En ambos casos, si no se generan de manera segura, un atacante podría adivinarlas.

## Objetivos de la prueba

- Determinar si la aplicación tiene cuentas de usuario con contraseñas predeterminadas.
- Revisar si las nuevas cuentas de usuario se crean con contraseñas débiles o predecibles.

## Cómo probar

### Credenciales predeterminadas del proveedor

El primer paso para identificar contraseñas predeterminadas es identificar el software que se utiliza. Esto se explica en detalle en la sección [Recopilación de información](../01-Information_Gathering/README.md) de la guía.

Una vez identificado el software, intenta averiguar si utiliza contraseñas predeterminadas y, de ser así, cuáles son. Esto debería incluir:

- Buscar "[SOFTWARE] default password".
- Revisar el manual o la documentación del proveedor.
- Consultar bases de datos de contraseñas predeterminadas comunes, como [CIRT.net](https://cirt.net/passwords), [SecLists Default Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials) o [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv).
- Inspeccionar el código fuente de la aplicación (si está disponible).
- Instalar la aplicación en una máquina virtual e inspeccionarla.
- Inspeccionar el hardware físico en busca de etiquetas adhesivas (frecuentes en dispositivos de red).

Si no se encuentra una contraseña predeterminada, prueba opciones comunes como:

- "admin", "password", "12345" u otras [contraseñas predeterminadas comunes](https://github.com/nixawk/fuzzdb/blob/master/bruteforce/passwds/default_devices_users%2Bpasswords.txt).
- Una contraseña vacía.
- El número de serie o la dirección MAC del dispositivo.

Si se desconoce el nombre de usuario, hay varias opciones para enumerar usuarios, que se explican en la guía [Enumeración de cuentas](../03-Identity_Management/04-Account_Enumeration_and_Guessable_User_Account.md). Como alternativa, prueba opciones comunes como "admin", "root" o "system".

### Contraseñas predeterminadas de la organización

Cuando el personal de una organización crea manualmente contraseñas para cuentas nuevas, puede hacerlo de manera predecible. Algunos ejemplos frecuentes son:

- Una contraseña común, como "Password1".
- Detalles específicos de la organización, como el nombre o la dirección de la organización.
- Contraseñas que siguen un patrón sencillo, como "Monday123" si la cuenta se crea un lunes.

A menudo es difícil identificar este tipo de contraseñas desde una perspectiva de caja negra, a menos que se puedan adivinar o descubrir mediante fuerza bruta. Sin embargo, son fáciles de identificar al realizar pruebas de caja gris o de caja blanca.

### Contraseñas predeterminadas generadas por la aplicación

Si la aplicación genera automáticamente contraseñas para nuevas cuentas de usuario, estas también pueden ser predecibles. Para probarlas, crea varias cuentas en la aplicación con datos similares y al mismo tiempo, y compara las contraseñas que se les asignan.

Las contraseñas pueden basarse en:

- Una cadena estática compartida entre las cuentas.
- Una parte hash u ofuscada de los datos de la cuenta, como `md5($username)`.
- Un algoritmo basado en el tiempo.
- Un generador de números pseudoaleatorios (PRNG) débil.

Este tipo de problema suele ser difícil de identificar desde una perspectiva de caja negra.

## Herramientas

- [Burp Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Nikto 2](https://cirt.net/Nikto2)
- [Nuclei](https://github.com/projectdiscovery/nuclei)
    - [Inicio de sesión predeterminado - Nuclei Templates](https://github.com/projectdiscovery/nuclei-templates/tree/6b26c63d8f63b2a812a478f14c4c098b485d54b4/http/default-logins)

## Referencias

- [CIRT](https://cirt.net/passwords)
- [SecLists Default Passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials)
- [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv)