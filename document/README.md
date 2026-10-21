# Contenido

## 0. [Prefacio](0-Foreword/README.md)

## 1. [Acerca de](1-About/)

## 2. [Introducción](2-Introduction/)

### 2.1 [Propósito y alcance](2-Introduction/README.md#Purpose-and-Scope)

### 2.2 [Por qué son importantes las pruebas de seguridad de aplicaciones web](2-Introduction/README.md#Why-Web-Application-Security-Testing-Matters)

### 2.3 [Quién debería usar esta guía](2-Introduction/README.md#Who-Should-Use-This-Guide)

### 2.4 [Principios fundamentales de las pruebas de seguridad eficaces](2-Introduction/README.md#Core-Principles-of-Effective-Security-Testing)

### 2.5 [Organización de la guía](2-Introduction/README.md#How-the-Guide-Is-Organized)

### 2.6 [Cómo hacer referencia a los escenarios del WSTG](2-Introduction/README.md#How-to-Reference-WSTG-Scenarios)

### 2.7 [Comentarios y sugerencias](2-Introduction/README.md#Feedback-and-Comments)

### 2.8 [Proyectos relacionados de OWASP](2-Introduction/README.md#Related-OWASP-Projects)

## 3. [El marco de pruebas de OWASP](3-The_OWASP_Testing_Framework/)

### 3.1 [El marco de pruebas de seguridad web](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md)

### 3.2 [Fase 1: antes de que comience el desarrollo](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-1-Before-Development-Begins)

### 3.3 [Fase 2: durante la definición y el diseño](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-2-During-Definition-and-Design)

### 3.4 [Fase 3: durante el desarrollo](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-3-During-Development)

### 3.5 [Fase 4: durante el despliegue](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-4-During-Deployment)

### 3.6 [Fase 5: durante el mantenimiento y las operaciones](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#Phase-5-During-Maintenance-and-Operations)

### 3.7 [Flujo de trabajo típico de pruebas del SDLC](3-The_OWASP_Testing_Framework/0-The_Web_Security_Testing_Framework.md#A-Typical-SDLC-Testing-Workflow)

### 3.8 [Metodologías de pruebas de penetración](3-The_OWASP_Testing_Framework/1-Penetration_Testing_Methodologies.md)

## 4. [Pruebas de seguridad de aplicaciones web](4-Web_Application_Security_Testing/)

### 4.0 [Introducción y objetivos](4-Web_Application_Security_Testing/00-Introduction_and_Objectives/README.md)

### 4.1 [Recopilación de información](4-Web_Application_Security_Testing/01-Information_Gathering/README.md)

#### 4.1.1 [Realizar reconocimiento con motores de búsqueda para detectar fugas de información](4-Web_Application_Security_Testing/01-Information_Gathering/01-Conduct_Search_Engine_Reconnaissance_for_Information_Leakage.md)

#### 4.1.2 [Identificar el servidor web](4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server.md)

#### 4.1.3 [Revisar los metarchivos del servidor web para detectar fugas de información](4-Web_Application_Security_Testing/01-Information_Gathering/03-Review_Webserver_Metafiles_for_Information_Leakage.md)

#### 4.1.4 [Identificar la superficie de ataque](4-Web_Application_Security_Testing/01-Information_Gathering/04-Attack_Surface_Identification.md)

#### 4.1.5 [Revisar el contenido de las páginas web para detectar fugas de información](4-Web_Application_Security_Testing/01-Information_Gathering/05-Review_Web_Page_Content_for_Information_Leakage.md)

#### 4.1.6 [Identificar los puntos de entrada de la aplicación](4-Web_Application_Security_Testing/01-Information_Gathering/06-Identify_Application_Entry_Points.md)

#### 4.1.7 [Trazar las rutas de ejecución de la aplicación](4-Web_Application_Security_Testing/01-Information_Gathering/07-Map_Execution_Paths_Through_Application.md)

#### 4.1.8 [Identificar el framework de la aplicación web](4-Web_Application_Security_Testing/01-Information_Gathering/08-Fingerprint_Web_Application_Framework.md)

#### 4.1.9 [Identificar la aplicación web](4-Web_Application_Security_Testing/01-Information_Gathering/09-Fingerprint_Web_Application.md)

#### 4.1.10 [Trazar la arquitectura de la aplicación](4-Web_Application_Security_Testing/01-Information_Gathering/10-Map_Application_Architecture.md)

### 4.2 [Gestión de la configuración y el despliegue](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/README.md)

#### 4.2.1 [Configuración de la infraestructura de red](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/01-Network_Infrastructure_Configuration.md)

#### 4.2.2 [Configuración de la plataforma de aplicaciones](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/02-Application_Platform_Configuration.md)

#### 4.2.3 [Gestión de extensiones de archivo con información sensible](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/03-File_Extensions_Handling_for_Sensitive_Information.md)

#### 4.2.4 [Revisar copias de seguridad antiguas y archivos sin referencias para detectar información sensible](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/04-Review_Old_Backup_and_Unreferenced_Files_for_Sensitive_Information.md)

#### 4.2.5 [Enumerar las interfaces de administración de la infraestructura y de las aplicaciones](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/05-Enumerate_Infrastructure_and_Application_Admin_Interfaces.md)

#### 4.2.6 [Métodos HTTP](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/06-HTTP_Methods.md)

#### 4.2.7 [HTTP Strict Transport Security](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/07-HTTP_Strict_Transport_Security.md)

#### 4.2.8 [Política entre dominios de RIA](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/08-RIA_Cross_Domain_Policy.md)

#### 4.2.9 [Permisos de archivos](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/09-File_Permissions.md)

#### 4.2.10 [Secuestro de subdominios](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/10-Subdomain_Takeover.md)

#### 4.2.11 [Almacenamiento en la nube](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/11-Cloud_Storage.md)

#### 4.2.12 [Content Security Policy](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/12-Content_Security_Policy.md)

#### 4.2.13 [Confusión de rutas](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/13-Path_Confusion.md)

#### 4.2.14 [Otras configuraciones incorrectas de encabezados de seguridad HTTP](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/14-Other_HTTP_Security_Header_Misconfigurations.md)

#### 4.2.15 [Elusión de seguridad mediante indicadores de funcionalidades](4-Web_Application_Security_Testing/02-Configuration_and_Deployment_Management/15-Feature_Flag_Security_Bypass.md)

### 4.3 [Gestión de identidades](4-Web_Application_Security_Testing/03-Identity_Management/README.md)

#### 4.3.1 [Definiciones de roles](4-Web_Application_Security_Testing/03-Identity_Management/01-Role_Definitions.md)

#### 4.3.2 [Proceso de registro de usuarios](4-Web_Application_Security_Testing/03-Identity_Management/02-User_Registration_Process.md)

#### 4.3.3 [Proceso de aprovisionamiento de cuentas](4-Web_Application_Security_Testing/03-Identity_Management/03-Account_Provisioning_Process.md)

#### 4.3.4 [Enumeración de cuentas y cuentas de usuario fáciles de adivinar](4-Web_Application_Security_Testing/03-Identity_Management/04-Account_Enumeration_and_Guessable_User_Account.md)

#### 4.3.5 [Política de nombres de usuario débil o no aplicada](4-Web_Application_Security_Testing/03-Identity_Management/05-Weak_or_Unenforced_Username_Policy.md)

### 4.4 [Autenticación](4-Web_Application_Security_Testing/04-Authentication/README.md)

#### 4.4.1 [Transmisión de credenciales por un canal cifrado](4-Web_Application_Security_Testing/04-Authentication/01-Credentials_Transported_over_an_Encrypted_Channel.md)

#### 4.4.2 [Credenciales predeterminadas](4-Web_Application_Security_Testing/04-Authentication/02-Default_Credentials.md)

#### 4.4.3 [Mecanismo de bloqueo débil](4-Web_Application_Security_Testing/04-Authentication/03-Weak_Lock_Out_Mechanism.md)

#### 4.4.4 [Elusión del esquema de autenticación](4-Web_Application_Security_Testing/04-Authentication/04-Bypassing_Authentication_Schema.md)

#### 4.4.5 [Funcionalidad vulnerable para recordar contraseñas](4-Web_Application_Security_Testing/04-Authentication/05-Vulnerable_Remember_Password.md)

#### 4.4.6 [Debilidades de la caché del navegador](4-Web_Application_Security_Testing/04-Authentication/06-Browser_Cache_Weaknesses.md)

#### 4.4.7 [Métodos de autenticación débiles](4-Web_Application_Security_Testing/04-Authentication/07-Weak_Authentication_Methods.md)

#### 4.4.8 [Respuesta débil a la pregunta de seguridad](4-Web_Application_Security_Testing/04-Authentication/08-Weak_Security_Question_Answer.md)

#### 4.4.9 [Funciones débiles para cambiar o restablecer contraseñas](4-Web_Application_Security_Testing/04-Authentication/09-Weak_Password_Change_or_Reset_Functionalities.md)

#### 4.4.10 [Autenticación más débil en un canal alternativo](4-Web_Application_Security_Testing/04-Authentication/10-Weaker_Authentication_in_Alternative_Channel.md)

#### 4.4.11 [Autenticación multifactor](4-Web_Application_Security_Testing/04-Authentication/11-Multi-Factor_Authentication.md)

### 4.5 [Autorización](4-Web_Application_Security_Testing/05-Authorization/README.md)

#### 4.5.1 [Inclusión de archivos y recorrido de directorios](4-Web_Application_Security_Testing/05-Authorization/01-Directory_Traversal_File_Include.md)

#### 4.5.2 [Elusión del esquema de autorización](4-Web_Application_Security_Testing/05-Authorization/02-Bypassing_Authorization_Schema.md)

#### 4.5.3 [Escalamiento de privilegios](4-Web_Application_Security_Testing/05-Authorization/03-Privilege_Escalation.md)

#### 4.5.4 [Referencias directas inseguras a objetos](4-Web_Application_Security_Testing/05-Authorization/04-Insecure_Direct_Object_References.md)

#### 4.5.5 [Debilidades de OAuth](4-Web_Application_Security_Testing/05-Authorization/05-OAuth_Weaknesses.md)

##### 4.5.5.1 [Debilidades del servidor de autorización de OAuth](4-Web_Application_Security_Testing/05-Authorization/05.1-OAuth_Authorization_Server_Weaknesses.md)

##### 4.5.5.2 [Debilidades del cliente de OAuth](4-Web_Application_Security_Testing/05-Authorization/05.2-OAuth_Client_Weaknesses.md)

### 4.6 [Gestión de sesiones](4-Web_Application_Security_Testing/06-Session_Management/README.md)

#### 4.6.1 [Esquema de gestión de sesiones](4-Web_Application_Security_Testing/06-Session_Management/01-Session_Management_Schema.md)

#### 4.6.2 [Atributos de las cookies](4-Web_Application_Security_Testing/06-Session_Management/02-Cookies_Attributes.md)

#### 4.6.3 [Fijación de sesión](4-Web_Application_Security_Testing/06-Session_Management/03-Session_Fixation.md)

#### 4.6.4 [Variables de sesión expuestas](4-Web_Application_Security_Testing/06-Session_Management/04-Exposed_Session_Variables.md)

#### 4.6.5 [Falsificación de solicitudes entre sitios](4-Web_Application_Security_Testing/06-Session_Management/05-Cross_Site_Request_Forgery.md)

#### 4.6.6 [Funcionalidad de cierre de sesión](4-Web_Application_Security_Testing/06-Session_Management/06-Logout_Functionality.md)

#### 4.6.7 [Tiempo de espera de la sesión](4-Web_Application_Security_Testing/06-Session_Management/07-Session_Timeout.md)

#### 4.6.8 [Confusión de sesiones](4-Web_Application_Security_Testing/06-Session_Management/08-Session_Puzzling.md)

#### 4.6.9 [Secuestro de sesión](4-Web_Application_Security_Testing/06-Session_Management/09-Session_Hijacking.md)

#### 4.6.10 [JSON Web Tokens](4-Web_Application_Security_Testing/06-Session_Management/10-JSON_Web_Tokens.md)

#### 4.6.11 [Sesiones concurrentes](4-Web_Application_Security_Testing/06-Session_Management/11-Concurrent_Sessions.md)

### 4.7 [Inyección](4-Web_Application_Security_Testing/07-Injection/README.md)

#### 4.7.1 [Cross-Site Scripting reflejado](4-Web_Application_Security_Testing/07-Injection/01-Reflected_Cross_Site_Scripting.md)

#### 4.7.2 [Cross-Site Scripting almacenado](4-Web_Application_Security_Testing/07-Injection/02-Stored_Cross_Site_Scripting.md)

#### 4.7.3 [Manipulación de verbos HTTP](4-Web_Application_Security_Testing/07-Injection/03-HTTP_Verb_Tampering.md)

#### 4.7.4 [Contaminación de parámetros HTTP](4-Web_Application_Security_Testing/07-Injection/04-HTTP_Parameter_Pollution.md)

#### 4.7.5 [Inyección SQL](4-Web_Application_Security_Testing/07-Injection/05-SQL_Injection.md)

##### 4.7.5.1 [Oracle](4-Web_Application_Security_Testing/07-Injection/05.1-Oracle.md)

##### 4.7.5.2 [MySQL](4-Web_Application_Security_Testing/07-Injection/05.2-MySQL.md)

##### 4.7.5.3 [SQL Server](4-Web_Application_Security_Testing/07-Injection/05.3-SQL_Server.md)

##### 4.7.5.4 [PostgreSQL](4-Web_Application_Security_Testing/07-Injection/05.4-PostgreSQL.md)

##### 4.7.5.5 [MS Access](4-Web_Application_Security_Testing/07-Injection/05.5-MS_Access.md)

##### 4.7.5.6 [Inyección NoSQL](4-Web_Application_Security_Testing/07-Injection/05.6-NoSQL_Injection.md)

##### 4.7.5.7 [Inyección ORM](4-Web_Application_Security_Testing/07-Injection/05.7-ORM_Injection.md)

##### 4.7.5.8 [Lado del cliente](4-Web_Application_Security_Testing/07-Injection/05.8-Client-side.md)

#### 4.7.6 [Inyección LDAP](4-Web_Application_Security_Testing/07-Injection/06-LDAP_Injection.md)

#### 4.7.7 [Inyección XML](4-Web_Application_Security_Testing/07-Injection/07-XML_Injection.md)

#### 4.7.8 [Inyección SSI](4-Web_Application_Security_Testing/07-Injection/08-SSI_Injection.md)

#### 4.7.9 [Inyección XPath](4-Web_Application_Security_Testing/07-Injection/09-XPath_Injection.md)

#### 4.7.10 [Inyección IMAP y SMTP](4-Web_Application_Security_Testing/07-Injection/10-IMAP_SMTP_Injection.md)

#### 4.7.11 [Inyección de código](4-Web_Application_Security_Testing/07-Injection/11-Code_Injection.md)

##### 4.7.11.1 [Inclusión de archivos](4-Web_Application_Security_Testing/07-Injection/11.1-File_Inclusion.md)

#### 4.7.12 [Inyección de comandos](4-Web_Application_Security_Testing/07-Injection/12-Command_Injection.md)

#### 4.7.13 [Inyección de cadenas de formato](4-Web_Application_Security_Testing/07-Injection/13-Format_String_Injection.md)

#### 4.7.14 [Vulnerabilidad incubada](4-Web_Application_Security_Testing/07-Injection/14-Incubated_Vulnerability.md)

#### 4.7.15 [División de respuestas HTTP](4-Web_Application_Security_Testing/07-Injection/15-HTTP_Response_Splitting.md)

#### 4.7.16 [Contrabando de solicitudes HTTP](4-Web_Application_Security_Testing/07-Injection/16-HTTP_Request_Smuggling.md)

#### 4.7.17 [Inyección en el encabezado Host](4-Web_Application_Security_Testing/07-Injection/17-Host_Header_Injection.md)

#### 4.7.18 [Inyección de plantillas del lado del servidor](4-Web_Application_Security_Testing/07-Injection/18-Server-side_Template_Injection.md)

#### 4.7.19 [Falsificación de solicitudes del lado del servidor](4-Web_Application_Security_Testing/07-Injection/19-Server-Side_Request_Forgery.md)

#### 4.7.20 [Asignación masiva](4-Web_Application_Security_Testing/07-Injection/20-Mass_Assignment.md)

#### 4.7.21 [Inyección CSV](4-Web_Application_Security_Testing/07-Injection/21-CSV_Injection.md)

#### 4.7.22 [Contaminación de prototipos](4-Web_Application_Security_Testing/07-Injection/22-Prototype_Pollution.md)

#### 4.7.23 [Deserialización insegura](4-Web_Application_Security_Testing/07-Injection/23-Insecure_Deserialization.md)

### 4.8 [Gestión de errores](4-Web_Application_Security_Testing/08-Error_Handling/README.md)

#### 4.8.1 [Gestión inadecuada de errores](4-Web_Application_Security_Testing/08-Error_Handling/01-Improper_Error_Handling.md)

#### 4.8.2 [Trazas de pila](4-Web_Application_Security_Testing/08-Error_Handling/02-Stack_Traces.md)

### 4.9 [Criptografía débil](4-Web_Application_Security_Testing/09-Weak_Cryptography/README.md)

#### 4.9.1 [Seguridad débil de la capa de transporte](4-Web_Application_Security_Testing/09-Weak_Cryptography/01-Weak_Transport_Layer_Security.md)

#### 4.9.2 [Oráculo de relleno](4-Web_Application_Security_Testing/09-Weak_Cryptography/02-Padding_Oracle.md)

#### 4.9.3 [Información sensible enviada por canales no cifrados](4-Web_Application_Security_Testing/09-Weak_Cryptography/03-Sensitive_Information_Sent_via_Unencrypted_Channels.md)

#### 4.9.4 [Primitivas criptográficas débiles](4-Web_Application_Security_Testing/09-Weak_Cryptography/04-Weak_Cryptographic_Primitives.md)

### 4.10 [Lógica de negocio](4-Web_Application_Security_Testing/10-Business_Logic/README.md)

#### 4.10.0 [Introducción a la lógica de negocio](4-Web_Application_Security_Testing/10-Business_Logic/00-Introduction_to_Business_Logic.md)

#### 4.10.1 [Validación de datos de la lógica de negocio](4-Web_Application_Security_Testing/10-Business_Logic/01-Business_Logic_Data_Validation.md)

#### 4.10.2 [Capacidad para falsificar solicitudes](4-Web_Application_Security_Testing/10-Business_Logic/02-Ability_to_Forge_Requests.md)

#### 4.10.3 [Comprobaciones de integridad](4-Web_Application_Security_Testing/10-Business_Logic/03-Integrity_Checks.md)

#### 4.10.4 [Temporización de procesos](4-Web_Application_Security_Testing/10-Business_Logic/04-Process_Timing.md)

#### 4.10.5 [Límites al número de veces que se puede usar una función](4-Web_Application_Security_Testing/10-Business_Logic/05-Number_of_Times_a_Function_Can_Be_Used_Limits.md)

#### 4.10.6 [Elusión de flujos de trabajo](4-Web_Application_Security_Testing/10-Business_Logic/06-Circumvention_of_Work_Flows.md)

#### 4.10.7 [Defensas contra el uso indebido de la aplicación](4-Web_Application_Security_Testing/10-Business_Logic/07-Defenses_Against_Application_Misuse.md)

#### 4.10.8 [Carga de tipos de archivo inesperados](4-Web_Application_Security_Testing/10-Business_Logic/08-Upload_of_Unexpected_File_Types.md)

#### 4.10.9 [Carga de archivos maliciosos](4-Web_Application_Security_Testing/10-Business_Logic/09-Upload_of_Malicious_Files.md)

#### 4.10.10 [Funcionalidad de pago](4-Web_Application_Security_Testing/10-Business_Logic/10-Payment_Functionality.md)

### 4.11 [Lado del cliente](4-Web_Application_Security_Testing/11-Client-side/README.md)

#### 4.11.1 [Cross-Site Scripting basado en DOM](4-Web_Application_Security_Testing/11-Client-side/01-DOM-based_Cross_Site_Scripting.md)

##### 4.11.1.1 [Cross-Site Scripting basado en DOM propio](4-Web_Application_Security_Testing/11-Client-side/01.1-Self_DOM_Based_Cross_Site_Scripting.md)

#### 4.11.2 [Ejecución de JavaScript](4-Web_Application_Security_Testing/11-Client-side/02-JavaScript_Execution.md)

#### 4.11.3 [Inyección HTML](4-Web_Application_Security_Testing/11-Client-side/03-HTML_Injection.md)

#### 4.11.4 [Redirección de URL del lado del cliente](4-Web_Application_Security_Testing/11-Client-side/04-Client-side_URL_Redirect.md)

#### 4.11.5 [Inyección CSS](4-Web_Application_Security_Testing/11-Client-side/05-CSS_Injection.md)

#### 4.11.6 [Manipulación de recursos del lado del cliente](4-Web_Application_Security_Testing/11-Client-side/06-Client-side_Resource_Manipulation.md)

#### 4.11.7 [Intercambio de recursos entre orígenes](4-Web_Application_Security_Testing/11-Client-side/07-Cross_Origin_Resource_Sharing.md)

#### 4.11.8 [Cross-Site Flashing](4-Web_Application_Security_Testing/11-Client-side/08-Cross_Site_Flashing.md)

#### 4.11.9 [Clickjacking](4-Web_Application_Security_Testing/11-Client-side/09-Clickjacking.md)

#### 4.11.10 [WebSockets](4-Web_Application_Security_Testing/11-Client-side/10-WebSockets.md)

#### 4.11.11 [Mensajería web](4-Web_Application_Security_Testing/11-Client-side/11-Web_Messaging.md)

#### 4.11.12 [Almacenamiento del navegador](4-Web_Application_Security_Testing/11-Client-side/12-Browser_Storage.md)

#### 4.11.13 [Inclusión de scripts entre sitios](4-Web_Application_Security_Testing/11-Client-side/13-Cross_Site_Script_Inclusion.md)

#### 4.11.14 [Reverse Tabnabbing](4-Web_Application_Security_Testing/11-Client-side/14-Reverse_Tabnabbing.md)

#### 4.11.15 [Inyección de plantillas del lado del cliente](4-Web_Application_Security_Testing/11-Client-side/15-Client-Side_Template_Injection.md)

### 4.12 [Pruebas de API](4-Web_Application_Security_Testing/12-API_Testing/README.md)

#### 4.12.0 [Descripción general de las pruebas de API](4-Web_Application_Security_Testing/12-API_Testing/00-API_Testing_Overview.md)

#### 4.12.1 [Reconocimiento de API](4-Web_Application_Security_Testing/12-API_Testing/01-API_Reconnaissance.md)

#### 4.12.2 [Autorización deficiente a nivel de objeto en API](4-Web_Application_Security_Testing/12-API_Testing/02-API_Broken_Object_Level_Authorization.md)

#### 4.12.3 [Exposición excesiva de datos](4-Web_Application_Security_Testing/12-API_Testing/03-Excessive_Data_Exposure.md)

#### 4.12.4 [Autorización deficiente a nivel de función en API](4-Web_Application_Security_Testing/12-API_Testing/04-API_Broken_Function_Level_Authorization.md)

#### 4.12.99 [GraphQL](4-Web_Application_Security_Testing/12-API_Testing/99-GraphQL.md)

### 4.13 [Pruebas de WebAssembly](4-Web_Application_Security_Testing/13-WebAssembly_Testing/README.md)

#### 4.13.0 [Descripción general de las pruebas de WebAssembly](4-Web_Application_Security_Testing/13-WebAssembly_Testing/00-WebAssembly_Testing_Overview.md)

#### 4.13.1 [Reconocimiento de WebAssembly](4-Web_Application_Security_Testing/13-WebAssembly_Testing/01-WebAssembly_Reconnaissance.md)

## 5. [Informes](5-Reporting/README.md)

### 5.1 [Estructura de los informes](5-Reporting/01-Reporting_Structure.md)

### 5.2 [Esquemas de nomenclatura](5-Reporting/02-Naming_Schemes.md)

## Apéndice A. [Historial](6-Appendix/A-History.md)

## Apéndice B. [Recursos de herramientas de pruebas](6-Appendix/B-Testing_Tools_Resource.md)

## Apéndice C. [Lecturas recomendadas](6-Appendix/C-Suggested_Reading.md)

## Apéndice D. [Fuzzing](6-Appendix/D-Fuzzing.md)

## Apéndice E. [Inyección codificada](6-Appendix/E-Encoded_Injection.md)

## Apéndice F. [Aprovechar las herramientas de desarrollo](6-Appendix/F-Leveraging_Dev_Tools.md)