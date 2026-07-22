# Metodologías de Pruebas de Penetración

## Resumen

- [Guías de Pruebas OWASP](#guías-de-pruebas-owasp)
    - Guía de Pruebas de Seguridad Web (WSTG)
    - Guía de Pruebas de Seguridad Móvil (MSTG)
    - Metodología de Pruebas de Seguridad de Firmware
- [Estándar de Ejecución de Pruebas de Penetración](#estándar-de-ejecución-de-pruebas-de-penetración)
- [Guía de Pruebas de Penetración PCI](#guía-de-pruebas-de-penetración-pci)
    - [Orientación de Pruebas de Penetración PCI DSS](#orientación-de-pruebas-de-penetración-pci-dss)
    - [Requisitos de Pruebas de Penetración PCI DSS](#requisitos-de-pruebas-de-penetración-pci-dss)
- [Marco de Pruebas de Penetración](#marco-de-pruebas-de-penetración)
- [Guía Técnica para Pruebas y Evaluación de Seguridad de la Información](#guía-técnica-para-pruebas-y-evaluación-de-seguridad-de-la-información)
- [Manual de Metodología de Pruebas de Seguridad de Código Abierto](#manual-de-metodología-de-pruebas-de-seguridad-de-código-abierto)
- [Referencias](#referencias)

## Guías de Pruebas OWASP

En términos de ejecución técnica de pruebas de seguridad, las guías de pruebas OWASP son altamente recomendadas. Dependiendo de los tipos de aplicaciones, las guías de pruebas se listan a continuación para servicios web/nube, aplicación móvil (Android/iOS), o firmware IoT respectivamente.

- [Guía de Pruebas de Seguridad Web OWASP](https://owasp.org/www-project-web-security-testing-guide/)
- [Guía de Pruebas de Seguridad Móvil OWASP](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Metodología de Pruebas de Seguridad de Firmware OWASP](https://github.com/scriptingxss/owasp-fstm)

## Estándar de Ejecución de Pruebas de Penetración

El Estándar de Ejecución de Pruebas de Penetración (PTES) define las pruebas de penetración como 7 fases. Particularmente, las Directrices Técnicas PTES dan sugerencias prácticas sobre procedimientos de pruebas, y recomendación para herramientas de pruebas de seguridad.

- Interacciones Pre-Compromiso
- Recopilación de Inteligencia
- Modelado de Amenazas
- Análisis de Vulnerabilidades
- Explotación
- Post Explotación
- Reporte

[Directrices Técnicas PTES](http://www.pentest-standard.org/index.php/PTES_Technical_Guidelines)

## Guía de Pruebas de Penetración PCI

El Estándar de Seguridad de Datos de la Industria de Tarjetas de Pago (PCI DSS) Requisito 11.3 define las pruebas de penetración. PCI también define Orientación de Pruebas de Penetración.

### Orientación de Pruebas de Penetración PCI DSS

La directriz de pruebas de penetración PCI DSS proporciona orientación sobre lo siguiente:

- Componentes de Pruebas de Penetración
- Calificaciones de un Probador de Penetración
- Metodologías de Pruebas de Penetración
- Directrices de Reporte de Pruebas de Penetración

### Requisitos de Pruebas de Penetración PCI DSS

Los requisitos PCI DSS se refieren al Estándar de Seguridad de Datos de la Industria de Tarjetas de Pago (PCI DSS) Requisito 11.3

- Basado en enfoques aceptados por la industria
- Cobertura para CDE y sistemas críticos
- Incluye pruebas externas e internas
- Prueba para validar reducción de alcance
- Pruebas de capa de aplicación
- Pruebas de capa de red para red y OS

[Orientación de Pruebas de Penetración PCI DSS](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)

## Marco de Pruebas de Penetración

El Marco de Pruebas de Penetración (PTF) proporciona una guía completa práctica de pruebas de penetración. También lista usos de las herramientas de pruebas de seguridad en cada categoría de pruebas. El área mayor de pruebas de penetración incluye:

- Huella de Red (Reconocimiento)
- Descubrimiento y Sondaje
- Enumeración
- Cracking de contraseñas
- Evaluación de Vulnerabilidades
- Auditoría AS/400
- Pruebas Específicas de Bluetooth
- Pruebas Específicas de Cisco
- Pruebas Específicas de Citrix
- Backbone de Red
- Pruebas Específicas de Servidor
- Seguridad VoIP
- Penetración Inalámbrica
- Seguridad Física
- Reporte Final - plantilla

[Marco de Pruebas de Penetración](https://web.archive.org/web/20130925044437/http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)

## Guía Técnica para Pruebas y Evaluación de Seguridad de la Información

La Guía Técnica para Pruebas y Evaluación de Seguridad de la Información (NIST 800-115) fue publicada por NIST, incluye algunas técnicas de evaluación listadas a continuación.

- Técnicas de Revisión
- Técnicas de Identificación y Análisis de Objetivos
- Técnicas de Validación de Vulnerabilidades de Objetivos
- Planificación de Evaluación de Seguridad
- Ejecución de Evaluación de Seguridad
- Actividades Post-Pruebas

El NIST 800-115 puede accederse [aquí](https://csrc.nist.gov/publications/detail/sp/800-115/final)

## Manual de Metodología de Pruebas de Seguridad de Código Abierto

El Manual de Metodología de Pruebas de Seguridad de Código Abierto (OSSTMM) es una metodología para probar la seguridad operacional de ubicaciones físicas, flujo de trabajo, pruebas de seguridad humana, pruebas de seguridad física, pruebas de seguridad inalámbrica, pruebas de seguridad de telecomunicaciones, pruebas de seguridad de redes de datos y cumplimiento. OSSTMM puede ser referencia de apoyo de ISO 27001 en lugar de una guía de penetración de aplicación técnica o práctica.

OSSTMM incluye las siguientes secciones clave:

- Análisis de Seguridad
- Métricas de Seguridad Operacional
- Análisis de Confianza
- Flujo de Trabajo
- Pruebas de Seguridad Humana
- Pruebas de Seguridad Física
- Pruebas de Seguridad Inalámbrica
- Pruebas de Seguridad de Telecomunicaciones
- Pruebas de Seguridad de Redes de Datos
- Regulaciones de Cumplimiento
- Reporte con el STAR (Informe de Auditoría de Pruebas de Seguridad)

[Manual de Metodología de Pruebas de Seguridad de Código Abierto](https://www.isecom.org/OSSTMM.3.pdf)

## Referencias

- [Estándar de Seguridad de Datos PCI - Orientación de Pruebas de Penetración](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)
- [Estándar PTES](http://www.pentest-standard.org/index.php/Main_Page)
- [Manual de Metodología de Pruebas de Seguridad de Código Abierto (OSSTMM)](https://www.isecom.org/research.html#content5-9d)
- [Guía Técnica para Pruebas y Evaluación de Seguridad de la Información NIST SP 800-115](https://csrc.nist.gov/publications/detail/sp/800-115/final)
- [Orientación de Pruebas de Seguridad HIPAA](https://www.hhs.gov/hipaa/for-professionals/security/guidance/cybersecurity/index.html)
- [Marco de Pruebas de Penetración 0.59 (Archivado)](https://web.archive.org/web/20130925044437/http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)
- [Guía de Pruebas de Seguridad Móvil OWASP](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Kali Linux](https://www.kali.org/)
- [Suplemento de Información: Requisito 11.3 Pruebas de Penetración](https://www.pcisecuritystandards.org/pdfs/infosupp_11_3_penetration_testing.pdf)