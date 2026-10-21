# Descripción general de la recopilación de información

Solo se puede probar lo que se puede encontrar. Antes de evaluar la autenticación, la autorización, la lógica de negocio o cualquier otra área de una aplicación, el evaluador debe identificar primero qué está realmente dentro del alcance: las aplicaciones, los hosts, los endpoints y las tecnologías que conforman la superficie de ataque. Los activos que nunca se descubren nunca se prueban, por exhaustiva que sea el resto de la evaluación.

Por lo tanto, la recopilación de información y el descubrimiento son la base de cualquier evaluación de seguridad de aplicaciones web. Combinan el reconocimiento pasivo (que no interactúa directamente con el objetivo) con la enumeración activa (que sí lo hace) para obtener una imagen lo más completa posible de lo que está expuesto, cómo se expone y qué tecnologías intervienen.

Las evaluaciones actuales suelen combinar la enumeración pasiva de subdominios (`subfinder`, Amass passive), el sondeo de hosts activos (`httpx`), la Transparencia de Certificados, la recopilación de URL de archivos de archivo (`gau`, `waybackurls`, `waymore`), el análisis de JavaScript (`jsluice`, LinkFinder) y el descubrimiento de contenido dirigido (`ffuf`, `feroxbuster`, Kiterunner). Consulta los escenarios INFO individuales y el capítulo de [Reconocimiento de API](../12-API_Testing/01-API_Reconnaissance.md) para ver ejemplos concretos.

Mantente siempre dentro del alcance definido para el encargo.