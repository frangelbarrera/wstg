# Metodologías de pruebas de penetración

## Resumen

- [Guías de pruebas OWASP](#guías-de-pruebas-owasp)
    - Guía de pruebas de seguridad web (WSTG)
    - Guía de pruebas de seguridad móvil (MSTG)
    - Metodología de pruebas de seguridad de firmware
- [Estándar de ejecución de pruebas de penetración](#estándar-de-ejecución-de-pruebas-de-penetración)
- [Guía de pruebas de penetración PCI](#guía-de-pruebas-de-penetración-pci)
    - [Orientación sobre pruebas de penetración PCI DSS](#orientación-sobre-pruebas-de-penetración-pci-dss)
    - [Requisitos de pruebas de penetración PCI DSS](#requisitos-de-pruebas-de-penetración-pci-dss)
- [Marco de pruebas de penetración](#marco-de-pruebas-de-penetración)
- [Guía técnica para pruebas y evaluación de la seguridad de la información](#guía-técnica-para-pruebas-y-evaluación-de-la-seguridad-de-la-información)
- [Manual de metodología de pruebas de seguridad de código abierto](#manual-de-metodología-de-pruebas-de-seguridad-de-código-abierto)
- [Referencias](#referencias)

## Guías de pruebas OWASP

En cuanto a la ejecución técnica de pruebas de seguridad, se recomiendan ampliamente las guías de pruebas de OWASP. Según el tipo de aplicación, a continuación se enumeran las guías para servicios web y en la nube, aplicaciones móviles (Android/iOS) o firmware de IoT, respectivamente.

- [Guía de pruebas de seguridad web de OWASP](https://owasp.org/www-project-web-security-testing-guide/)
- [Guía de pruebas de seguridad móvil de OWASP](https://mas.owasp.org/)
- [Metodología de pruebas de seguridad de firmware de OWASP](https://github.com/scriptingxss/owasp-fstm)

## Estándar de ejecución de pruebas de penetración

El Estándar de ejecución de pruebas de penetración (PTES) define las pruebas de penetración en 7 fases. En particular, las Directrices técnicas de PTES ofrecen sugerencias prácticas sobre los procedimientos de prueba y recomendaciones de herramientas para pruebas de seguridad.

- Interacciones previas al compromiso
- Recopilación de inteligencia
- Modelado de amenazas
- Análisis de vulnerabilidades
- Explotación
- Explotación posterior
- Elaboración de informes

[Directrices técnicas de PTES](http://www.pentest-standard.org/index.php/PTES_Technical_Guidelines)

## Guía de pruebas de penetración PCI

El requisito 11.3 del Estándar de Seguridad de Datos de la Industria de Tarjetas de Pago (PCI DSS) define las pruebas de penetración. PCI también establece directrices para las pruebas de penetración.

### Orientación sobre pruebas de penetración PCI DSS

La directriz de pruebas de penetración de PCI DSS proporciona orientación sobre lo siguiente:

- Componentes de las pruebas de penetración
- Cualificaciones de un profesional de pruebas de penetración
- Metodologías de pruebas de penetración
- Directrices para elaborar informes de pruebas de penetración

### Requisitos de pruebas de penetración PCI DSS

El requisito de PCI DSS hace referencia al requisito 11.3 del Estándar de Seguridad de Datos de la Industria de Tarjetas de Pago (PCI DSS).

- Basarse en enfoques aceptados por la industria
- Cubrir el CDE y los sistemas críticos
- Incluir pruebas externas e internas
- Realizar pruebas para validar la reducción del alcance
- Realizar pruebas de la capa de aplicación
- Realizar pruebas de la capa de red para la red y el sistema operativo

[Orientación sobre pruebas de penetración PCI DSS](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)

## Marco de pruebas de penetración

El Marco de pruebas de penetración (PTF) ofrece una guía práctica y completa sobre pruebas de penetración. También enumera los usos de las herramientas de pruebas de seguridad en cada categoría de pruebas. Las principales áreas de las pruebas de penetración incluyen:

- Reconocimiento de la red
- Descubrimiento y sondeo
- Enumeración
- Descifrado de contraseñas
- Evaluación de vulnerabilidades
- Auditoría de AS/400
- Pruebas específicas de Bluetooth
- Pruebas específicas de Cisco
- Pruebas específicas de Citrix
- Infraestructura troncal de red
- Pruebas específicas de servidores
- Seguridad de VoIP
- Pruebas de penetración inalámbrica
- Seguridad física
- Informe final: plantilla

[Marco de pruebas de penetración](https://web.archive.org/web/20130925044437/http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)

## Guía técnica para pruebas y evaluación de la seguridad de la información

La Guía técnica para pruebas y evaluación de la seguridad de la información (NIST 800-115), publicada por el NIST, incluye algunas técnicas de evaluación que se enumeran a continuación.

- Técnicas de revisión
- Técnicas de identificación y análisis de objetivos
- Técnicas de validación de vulnerabilidades de objetivos
- Planificación de la evaluación de seguridad
- Ejecución de la evaluación de seguridad
- Actividades posteriores a las pruebas

Para más detalles, consulte [NIST 800-115](https://csrc.nist.gov/publications/detail/sp/800-115/final).

## Manual de metodología de pruebas de seguridad de código abierto

El Manual de metodología de pruebas de seguridad de código abierto (OSSTMM) es una metodología para evaluar la seguridad operativa de ubicaciones físicas, flujos de trabajo, seguridad humana, seguridad física, seguridad inalámbrica, seguridad de las telecomunicaciones, seguridad de las redes de datos y cumplimiento. OSSTMM puede servir como referencia complementaria de ISO 27001, en lugar de como guía práctica o técnica para pruebas de penetración de aplicaciones.

OSSTMM incluye las siguientes secciones principales:

- Análisis de seguridad
- Métricas de seguridad operativa
- Análisis de confianza
- Flujo de trabajo
- Pruebas de seguridad humana
- Pruebas de seguridad física
- Pruebas de seguridad inalámbrica
- Pruebas de seguridad de las telecomunicaciones
- Pruebas de seguridad de redes de datos
- Normativa de cumplimiento
- Elaboración de informes con STAR (informe de auditoría de pruebas de seguridad)

[Manual de metodología de pruebas de seguridad de código abierto](https://www.isecom.org/OSSTMM.3.pdf)

## Referencias

- [Estándar de seguridad de datos PCI: orientación sobre pruebas de penetración](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)
- [Estándar PTES](http://www.pentest-standard.org/index.php/Main_Page)
- [Manual de metodología de pruebas de seguridad de código abierto (OSSTMM)](https://www.isecom.org/research.html#content5-9d)
- [Guía técnica para pruebas y evaluación de la seguridad de la información NIST SP 800-115](https://csrc.nist.gov/publications/detail/sp/800-115/final)
- [Orientación sobre pruebas de seguridad de HIPAA](https://www.hhs.gov/hipaa/for-professionals/security/guidance/cybersecurity/index.html)
- [Marco de pruebas de penetración 0.59 (archivado)](https://web.archive.org/web/20130925044437/http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)
- [Guía de pruebas de seguridad móvil de OWASP](https://mas.owasp.org/)
- [Kali Linux](https://www.kali.org/)
- [Suplemento informativo: requisito 11.3, pruebas de penetración](https://www.pcisecuritystandards.org/pdfs/infosupp_11_3_penetration_testing.pdf)