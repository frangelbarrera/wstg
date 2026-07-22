# Mapear Rutas de Ejecución a Través de la Aplicación

|ID          |
|------------|
|WSTG-INFO-07|

## Resumen

Antes de comenzar las pruebas de seguridad, entender la estructura de la aplicación es primordial. Sin una comprensión exhaustiva del layout de la aplicación, una prueba comprehensiva es improbable.

## Objetivos de Prueba

- Mapear la aplicación objetivo y entender los workflows principales.

## Cómo Probar

En pruebas black-box, es extremadamente difícil probar todo el codebase. Esto no es solo porque el probador no puede ver los code paths a través de la aplicación, sino también porque probar todos los code paths sería extremadamente time-consuming. Una manera de reconciliar esto es documentar los code paths que fueron descubiertos y probados.

Hay varias maneras de abordar las pruebas y medición de code coverage:

- **Path** - probar cada uno de los paths a través de una aplicación que incluye análisis combinatorial y boundary value testing para cada decision path. Mientras este approach ofrece thoroughness, el número de testable paths crece exponencialmente con cada decision branch.
- **Data Flow (or Taint Analysis)** - tests the assignment of variables via external interaction (normally users). Focuses on mapping the flow, transformation and use of data throughout an application.
- **Race** - tests multiple concurrent instances of the application manipulating the same data.

La choice de method y la extent a la que cada method es usado debería ser negotiated con el application owner. Adicionalmente, simpler approaches podrían ser adopted. For example, the tester could ask the application owner about specific functions or code sections that they are particularly concerned about, and discuss how those code segments can be reached.

To demonstrate code coverage to the application owner, the tester can start by documenting all the links discovered from spidering the application (either manually or automatically) in a spreadsheet. The tester can then look more closely at decision points in the application and investigate how many significant code paths are discovered. These should then be documented in the spreadsheet with URLs, prose and screenshot descriptions of the paths discovered.

### Automatic Spidering

An automatic spider is a tool that is used to discover new resources (URLs) on a specific site automatically. It begins with a list of URLs to visit, called the seeds, which depends on how the Spider is started. While there are a lot of Spidering tools, the following example uses the [Zed Attack Proxy (ZAP)](https://github.com/zaproxy/zaproxy):

![Zed Attack Proxy Screen](images/OWASPZAPSP.png)\
*Figure 4.1.7-1: Zed Attack Proxy Screen*

[ZAP](https://github.com/zaproxy/zaproxy) offers various automatic spidering options, which can be leveraged based on the tester's needs:

- [Spider](https://www.zaproxy.org/docs/desktop/start/features/spider/)
- [Ajax Spider](https://www.zaproxy.org/docs/desktop/addons/ajax-spider/)
- [OpenAPI Support](https://www.zaproxy.org/docs/desktop/addons/openapi-support/)

## Herramientas

- [Zed Attack Proxy (ZAP)](https://github.com/zaproxy/zaproxy)
- [List of spreadsheet software](https://en.wikipedia.org/wiki/List_of_spreadsheet_software)
- [Diagramming software](https://en.wikipedia.org/wiki/List_of_concept-_and_mind-mapping_software)

## Referencias

- [Code Coverage](https://en.wikipedia.org/wiki/Code_coverage)