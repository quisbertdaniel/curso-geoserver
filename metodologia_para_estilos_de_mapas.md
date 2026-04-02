# Guía Práctica: Metodologías para Estilos de Mapas en GIS

Un mapa no solo muestra información espacial; su objetivo es contar una historia de forma clara. Para lograrlo, los profesionales en GIS utilizan metodologías que se dividen en dos áreas principales: **Diseño Visual** (las reglas cartográficas) y **Estándares Técnicos** (el código que entienden los servidores y aplicaciones).

---

## 1. Metodología de Diseño Visual (Las Reglas de Oro)

Antes de programar, hay que diseñar pensando en el usuario final. Estas son las tres reglas fundamentales:

* **Jerarquía Visual (Fondo vs. Frente):** * El **mapa base** (calles, terreno) debe usar colores neutros, grises o pasteles. Su trabajo es dar contexto, no distraer.
    * Los **datos principales** (la capa que estás analizando) deben usar colores vivos y saturados para resaltar inmediatamente.
* **Renderizado por Escala (Reglas de Zoom):** * No se dibuja todo al mismo tiempo. A nivel nacional, se ocultan las calles y se muestran ciudades. Al acercar el zoom (nivel de calle), aparecen los detalles. Esto mantiene el mapa limpio y mejora la velocidad de carga.
* **Uso Inteligente del Color:** * No elegir colores al azar. Se deben usar paletas metodológicas según el dato:
        * *Secuenciales:* De claro a oscuro (ej. cantidad de habitantes).
        * *Categóricos:* Colores distintos para categorías diferentes (ej. bosque, agua, ciudad).

## 2. Estándares Técnicos (El "Idioma" de los Estilos)

Para que un servidor o una aplicación móvil pueda dibujar el mapa que diseñaste, debes traducir el estilo a un estándar internacional. Los dos más importantes son:

* **SLD (Styled Layer Descriptor):** * **Qué es:** Es el estándar oficial creado por el OGC (Open Geospatial Consortium). Está escrito en formato XML.
    * **Para qué sirve:** Es el idioma principal que utilizan servidores como **GeoServer**. Le dice al servidor exactamente qué color, grosor de línea o símbolo usar antes de enviar la imagen (WMS) al cliente.
* **Mapbox Style Specification (JSON):** * **Qué es:** Un formato de texto en JSON, mucho más ligero y moderno.
    * **Para qué sirve:** Es el estándar dominante hoy en día para **Teselas Vectoriales (Vector Tiles)**. Es la metodología que se usa cuando el mapa se dibuja directamente en el dispositivo del usuario, ideal para consumo en aplicaciones móviles y páginas web interactivas.

## 3. El Flujo de Trabajo Profesional (Paso a Paso)

En la industria real, no se suele escribir el código de los estilos a mano desde cero. El ciclo metodológico es este:

1.  **Conexión y Diseño Visual:** Conectas tus datos (ej. desde PostGIS) a un software de escritorio como **QGIS**. Allí, diseñas el mapa usando herramientas visuales, probando colores y grosores.
2.  **Exportación Automática:** Una vez que el mapa se ve perfecto en QGIS, usas las herramientas del programa para exportar ese diseño automáticamente a un archivo `.sld` o un `.json`.
3.  **Publicación:** * Subes el archivo `.sld` a tu servidor (GeoServer) para publicarlo vía web.
    * O alojas tu archivo `.json` para que una aplicación móvil lo lea y pinte los vectores en pantalla.

---

## 4. Enlaces de Referencia Esenciales

Aquí tienes las herramientas y documentaciones estándar de la industria para aplicar estas metodologías:

* **Para Metodología de Color:**
    * [ColorBrewer 2.0](https://colorbrewer2.org/): La herramienta estándar a nivel mundial para elegir paletas de colores cartográficas correctas según el tipo de dato.
* **Para Estándares Técnicos:**
    * [Documentación Oficial de SLD (OGC)](https://www.ogc.org/standard/sld/): El sitio oficial para entender cómo funciona la estructura XML del SLD.
    * [Manual de GeoServer - Trabajando con SLD](https://docs.geoserver.org/stable/en/user/styling/sld/index.html): Guía fundamental para aplicar SLD en la práctica dentro del servidor.
    * [Mapbox Style Specification](https://docs.mapbox.com/mapbox-gl-js/style-spec/): Toda la sintaxis JSON necesaria para dar estilo a mapas en aplicaciones modernas.
* **Para Flujo de Trabajo:**
    * [Documentación de QGIS - Creación de Estilos](https://docs.qgis.org/testing/en/docs/user_manual/working_with_vector/vector_properties.html#symbology-properties): Cómo crear el estilo visualmente para luego exportarlo.
