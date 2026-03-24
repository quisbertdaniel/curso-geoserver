# GeoServer — Guía completa: Editar capa (las 5 pestañas)

> Referencia rápida para **toda** la ventana de **Editar capa** al publicar un shapefile en GeoServer.
> Ejemplo: capa `DEPARTAMENTOS:departamentos_geo`

---

## 1. Pestaña: Datos

Define qué es la capa, de dónde viene, su sistema de coordenadas y su extensión geográfica.

### Información básica del recurso

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Nombre** | Identificador interno de la capa en GeoServer. Aparece en las URLs de WMS/WFS. | Dejar `departamentos_geo` o renombrar a algo limpio como `departamentos` | ⚠️ Revisar |
| **Habilitado** | Si está apagado, la capa existe pero no responde peticiones. | Debe estar activado (☑) | 🔴 Obligatorio |
| **Anunciado** | Si aparece en el GetCapabilities (catálogo público WMS/WFS). | Activar si quieres que clientes la descubran | 🔴 Obligatorio |
| **Título** | Nombre legible que ven los usuarios en visores y clientes GIS. | Ej: `Departamentos de Bolivia` | 🔴 Obligatorio |
| **Resumen** | Descripción corta que aparece en GetCapabilities. | Ej: "Límites departamentales de Bolivia" | ⚠️ Recomendado |

### Metadatos y descubrimiento

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Palabras clave** | Tags para búsqueda en catálogos CSW/OGC. Mejoran la visibilidad de tu capa en infraestructuras de datos espaciales. | Ej: Bolivia, departamentos, límites, administrativo | ⚠️ Recomendado |
| **Vocabulario** | Permite asociar las palabras clave a un vocabulario controlado (tesauro) registrado en GeoServer, como GEMET o INSPIRE. Si no tienes un tesauro configurado, se dejan como palabras clave libres. | Dejar en blanco para uso general. Solo seleccionar un vocabulario si publicas en una IDE formal que lo requiera | ⏭️ Ignorar |
| **i18n** | Internacionalización: título y resumen en otros idiomas para servicios multilingües. | No necesitas esto para uso local | ⏭️ Ignorar |
| **Vínculos a metadatos** | URLs a fichas de metadatos externos (ISO 19115, FGDC, TC211). Solo aparecen en capabilities WMS 1.1.1 si son de tipo FGDC o TC211. | Solo si publicas en un catálogo formal (IDE, GeoBolivia) | 🔵 Opcional |
| **Enlaces de datos** | URLs para descargar los datos en otros formatos (shapefile, GeoJSON, etc.). | Opcional, para ofrecer descarga directa fuera de WFS | 🔵 Opcional |

### Sistema de referencia de coordenadas

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **SRS nativo** | El sistema de coordenadas original del shapefile. GeoServer lo lee del archivo `.prj`. | Verificar que sea `EPSG:4326` (WGS 84) | 🔴 Verificar |
| **SRS declarado** | El SRS que GeoServer anuncia en los servicios WMS/WFS. | Debe coincidir con el nativo: `EPSG:4326` | 🔴 Verificar |
| **Gestión de SRC** | Cómo GeoServer resuelve conflictos entre el SRS nativo y el declarado. Opciones: **Nativo** (usa el del .prj), **Forzar declarado** (ignora el .prj y usa el declarado), **Reproyectar** (transforma al vuelo). | Usar `Forzar declarado` si el .prj no tiene código EPSG. Si coinciden, dejar en "Nativo" | 🔴 Configurar |

### Encuadres (Bounding Boxes)

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Encuadre nativo** | Extensión geográfica (Min X, Min Y, Máx X, Máx Y) en coordenadas del SRS nativo. Define el área que cubre la capa. | Clic en **"Calcular desde los datos"** | 🔴 Calcular |
| **Encuadre Lat/Lon** | Extensión en WGS84 (EPSG:4326). Es lo que aparece en GetCapabilities y lo usan los clientes para centrar el mapa. | Clic en **"Calcular desde el encuadre nativo"** | 🔴 Calcular |

### Geometrías y atributos

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Geometrías curvas** | Soporte para arcos circulares en geometrías. La tolerancia de linealización controla cuántos segmentos rectos aproximan una curva. | No aplica para shapefiles (solo usan polígonos rectos) | ⏭️ Ignorar |
| **Detalles del Feature Type** | Lista de atributos del shapefile con su tipo de dato. En este caso: `the_geom` (MultiPolygon), `ogc_fid` (Integer), `id` (Integer), `c_ut` (String), `departamen` (String). | Solo lectura. Verificar que `departamen` tenga los nombres correctos | ⚠️ Verificar |
| **Filtro CQL** | Permite restringir qué features se publican usando sintaxis CQL. Ej: `departamen = 'La Paz'` publicaría solo ese departamento. | Dejar vacío para publicar todos los departamentos | ⏭️ Ignorar |

### Resumen pestaña Datos: 6 pasos

1. Poner un **Título** legible: "Departamentos de Bolivia"
2. Verificar que **Habilitado** y **Anunciado** estén activados
3. Confirmar que ambos SRS digan `EPSG:4326`
4. Clic en **"Calcular desde los datos"** (encuadre nativo)
5. Clic en **"Calcular desde el encuadre nativo"** (encuadre Lat/Lon)
6. **Guardar**

---

## 2. Pestaña: Publicación

Controla cómo se comporta la capa en los servicios WMS, WFS y KML. Es donde defines estilos, límites y comportamiento de caché HTTP.

### Configuración de HTTP

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Cabeceras de respuesta de caché** | Activa las cabeceras HTTP `Cache-Control` y `Expires` en las respuestas del servidor. Permite que navegadores y proxies almacenen las imágenes de mapa sin volver a pedirlas. | Activar si tu capa no cambia frecuentemente (datos estáticos como departamentos) | ⚠️ Recomendado |
| **Tiempo de caché (segundos)** | Cuántos segundos el navegador/proxy puede reusar la respuesta sin consultar al servidor. Ej: 3600 = 1 hora, 86400 = 1 día. | Para datos estáticos: `86400` (1 día). Para datos que cambian: `3600` (1 hora) o menos | ⚠️ Recomendado |

### Root Layer in Capabilities

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Configuración global de WMS** | Usa la configuración global del servidor para decidir si esta capa aparece como raíz en GetCapabilities. | Dejar en **"Configuración global de WMS"** (opción por defecto) | ⏭️ Ignorar |
| **Sí / No** | Fuerza que esta capa sea o no sea la capa raíz del árbol WMS. Solo útil si tienes una estructura jerárquica personalizada. | No tocar salvo que organices capas en grupos jerárquicos | ⏭️ Ignorar |

### Services Settings — Opciones de Capa

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Selectively disable services for layer** | Permite desactivar servicios específicos (WMS, WFS, WCS) solo para esta capa. Ej: publicar como mapa (WMS) pero no permitir descarga (WFS). | Dejar desactivado para que funcione en todos los servicios. Activar solo si quieres restringir acceso a WFS por ejemplo | 🔵 Opcional |

### Configuración WMS — Opciones de Capa

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Interrogable** | Permite hacer GetFeatureInfo (clic en el mapa para ver atributos del feature). Si se desactiva, la capa se ve pero no se puede consultar. | **Activar** (☑). Es lo que permite que al hacer clic en un departamento se vean sus datos | 🔴 Obligatorio |
| **Opaco** | Indica si la capa tiene fondo opaco (sin transparencia). Afecta cómo se superpone con otras capas en el mapa. | **Desactivar** para capas vectoriales. Así los polígonos de departamentos se pueden superponer sobre un mapa base | 🔴 Configurar |
| **Estilo por defecto** | El estilo SLD que se aplica cuando un cliente no especifica uno. Define colores, bordes, etiquetas de los polígonos. | Seleccionar un estilo apropiado. Ej: `polygon` (genérico) o crear uno personalizado como `departamentos_style` | 🔴 Obligatorio |
| **Estilos adicionales** | Estilos alternativos que el cliente puede elegir vía parámetro `STYLES` en la petición WMS. | Mover estilos de "Disponibles" a "Seleccionados" si quieres ofrecer variantes (ej: uno con etiquetas, otro sin) | 🔵 Opcional |
| **Buffer de renderizado por defecto** | Píxeles extra que GeoServer dibuja fuera del tile visible. Evita que etiquetas o símbolos se corten en los bordes de las teselas. | Dejar en `0` para polígonos simples. Subir a `50-100` si usas etiquetas que se cortan en los bordes | ⚠️ Revisar |
| **Ruta WMS por defecto** | Ruta virtual para organizar capas en el árbol del GetCapabilities. Ej: `/Bolivia/Administrativo`. | Útil si tienes muchas capas y quieres organizarlas jerárquicamente. Para pocas capas, dejar vacío | ⏭️ Ignorar |
| **Método de interpolación predeterminado** | Algoritmo para reescalar imágenes raster (Nearest, Bilinear, Bicubic). | No aplica para shapefiles vectoriales. Ignorar | ⏭️ Ignorar |

### URLs autoritativas para esta capa WMS

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **URLs autoritativas** | URLs de otros servidores WMS que publican esta misma capa. Permite a clientes saber que hay fuentes alternativas del mismo dato. | Solo si participas en una IDE con servidores espejo. Caso normal: dejar vacío | ⏭️ Ignorar |

### Identificadores de la capa

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Identificadores de capa** | Códigos de referencia de la capa en catálogos externos (ej: un código ISO o un UUID de catálogo). | Solo si registras la capa en un catálogo formal. Caso normal: dejar vacío | ⏭️ Ignorar |

### Atribución de WMS

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Texto de atribución** | Crédito del autor/fuente que aparece en GetCapabilities. Los visores pueden mostrarlo como pie de mapa. | Ej: `Instituto Geográfico Militar - Bolivia` o `GeoBolivia` | ⚠️ Recomendado |
| **Vínculo a la información de atribución** | URL a la página web de la fuente original de los datos. | Ej: `https://geo.gob.bo` | 🔵 Opcional |
| **URL del logo** | URL de una imagen con el logo de la fuente de datos. | Ej: URL del logo de GeoBolivia si los datos vienen de ahí | 🔵 Opcional |
| **Content type del logo** | Tipo MIME de la imagen del logo. | `image/png` o `image/jpeg` según corresponda | 🔵 Opcional |
| **Altura/Anchura de la imagen para el logo** | Dimensiones en píxeles del logo. | Usar **"Auto detectar tamaño y tipo de imagen"** para que GeoServer lo calcule solo | 🔵 Opcional |

### Configuración del formato KML

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Atributo de "regionado" por defecto** | Qué atributo usa GeoServer para decidir qué features mostrar primero en Google Earth cuando hay muchos datos. | Dejar por defecto. Solo relevante si publicas para Google Earth | ⏭️ Ignorar |
| **Método de "regionado" por defecto** | Algoritmo de simplificación para KML: `native-sorting`, `best_guess`, `random`, etc. | Dejar por defecto | ⏭️ Ignorar |
| **Features por tesela de "regionado"** | Cuántos features se incluyen por región/tile en KML. | Dejar por defecto (64) | ⏭️ Ignorar |

### Configuración de WFS — Features

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Límite de número de features por consulta** | Máximo de features que devuelve una petición GetFeature. Protege al servidor de consultas que pidan millones de registros. `0` = sin límite (usa el global). | Para departamentos (solo 9 registros) no es necesario. Para capas grandes: `5000` o `10000` | 🔵 Opcional |
| **Máximo número de decimales** | Precisión de las coordenadas en las respuestas GML/GeoJSON. Más decimales = más precisión pero archivos más pesados. | `6` decimales es suficiente (~11 cm de precisión). Para departamentos `4` bastaría | ⚠️ Recomendado |
| **Right-pad decimals with zeros** | Rellena con ceros a la derecha hasta alcanzar el máximo de decimales. Ej: `65.2000` en vez de `65.2`. | Desactivar. Solo útil si necesitas formato fijo para integración con sistemas externos | ⏭️ Ignorar |
| **Forced decimal notation** | Fuerza notación decimal en vez de científica. Ej: `0.000123` en vez de `1.23E-4`. | Activar. Evita problemas de compatibilidad con algunos clientes GIS | ⚠️ Recomendado |
| **Activate complex to simple features conversion** | Convierte features complejos (con atributos anidados) a features simples (tabla plana). | No aplica para shapefiles. Ignorar | ⏭️ Ignorar |

### Omitir NumberMatched

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Omitir el conteo del atributo numberMatched** | En WFS 2.0, omite contar el total de features que coinciden con el filtro. Mejora rendimiento en capas enormes porque evita un COUNT antes de la consulta. | Desactivar (dejar el conteo). Solo activar para capas con millones de registros donde el conteo sea lento | ⏭️ Ignorar |

### Códigos SRS extra y coordenadas

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Códigos SRS extra para WFS** | Sistemas de referencia adicionales que GeoServer reproyecta al vuelo para esta capa en WFS. Ej: agregar `EPSG:32719` (UTM 19S) además del nativo 4326. | Agregar SRS que tus usuarios necesiten. Para Bolivia: `EPSG:32719`, `EPSG:32720` (UTM zonas 19S y 20S) | 🔵 Opcional |
| **Anular la lista SRS amplia de WFS** | Reemplaza la lista global de SRS con una lista personalizada solo para esta capa. | Dejar desactivado | ⏭️ Ignorar |
| **Encode coordinates measures** | Incluye valores M (medidas) en las coordenadas si existen en la geometría. | Desactivar. Los shapefiles de departamentos no tienen coordenadas M | ⏭️ Ignorar |

---

## 3. Pestaña: Dimensiones

Permite publicar la capa con dimensiones adicionales (tiempo, elevación o personalizadas) para filtrar datos en las peticiones WMS/WFS. Ej: mostrar datos de un mes específico.

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Hora — Habilitado** | Activa la dimensión temporal. Requiere un atributo de tipo Fecha en el shapefile. Permite peticiones como `&TIME=2024-01` para ver datos de ese período. | Aparece deshabilitado: "No se ha encontrado ningún atributo de tipo Fecha". Tu shapefile no tiene campo de fecha, así que no aplica | ⏭️ No aplica |
| **Elevación — Habilitado** | Activa la dimensión de elevación. Requiere un atributo numérico que represente altitud/profundidad. Permite peticiones como `&ELEVATION=3000`. | Tu shapefile de departamentos no tiene datos de elevación. Dejar desactivado | ⏭️ Ignorar |
| **Custom Vector Dimensions** | Permite crear dimensiones personalizadas basadas en cualquier atributo. Ej: dimensión "año" o "categoría" para filtrar vía parámetro en la URL. | No necesario para departamentos. Útil si publicas datos multitemporales o con categorías filtrables | ⏭️ Ignorar |

> **¿Cuándo sí usar dimensiones?** Si publicas datos de precipitación mensual, temperatura por año, o cualquier dato que varíe en el tiempo. En ese caso necesitarías un campo fecha en tu shapefile o base de datos.

---

## 4. Pestaña: Cacheado de Teselas

Configura GeoWebCache (GWC) integrado en GeoServer. Genera y almacena teselas (tiles) pre-renderizadas para servir mapas más rápido sin redibujar cada vez.

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Enable tile caching for this layer** | Activa el caché de teselas. Las peticiones WMS se almacenan como imágenes pre-generadas. Mejora dramáticamente el rendimiento en capas que no cambian. | **Activar** (☑). Departamentos es una capa estática, ideal para cachear | 🔴 Recomendado |
| **BlobStore** | Dónde se almacenan físicamente las teselas. Por defecto usa el almacenamiento local del servidor. | Dejar por defecto a menos que tengas configurado almacenamiento externo (S3, etc.) | ⏭️ Ignorar |
| **Meta tiling factors** | Cuántas teselas se renderizan juntas en una sola imagen grande antes de cortarlas. Ej: `4x4` genera una imagen de 4×4 teselas y la divide. Reduce artefactos en bordes y etiquetas duplicadas. | `4 x 4` es un buen valor por defecto | ⚠️ Recomendado |
| **Gutter size (pixels)** | Píxeles extra alrededor de cada meta-tesela para evitar que etiquetas y símbolos se corten. Similar al buffer de renderizado pero para el caché. | `50` píxeles si usas etiquetas. `0` si son solo polígonos sin texto | 🔵 Opcional |
| **Cache image formats** | Formatos de imagen que se cachean. Cada formato consume espacio en disco. | Activar solo los que necesites: `image/png` (calidad, transparencia) y/o `image/jpeg` (más liviano, sin transparencia). Para departamentos: **image/png** | 🔴 Configurar |
| **Expirar cache de servidor (segundos)** | Cuánto tiempo el servidor mantiene las teselas antes de regenerarlas. `0` = usa configuración global. | `0` para datos estáticos (se regeneran solo manualmente). Para datos dinámicos: `86400` (1 día) | ⏭️ Dejar en 0 |
| **Expirar cache del cliente (segundos)** | Cuánto tiempo el navegador del usuario mantiene las teselas sin volver a pedirlas. `0` = usa configuración global. | `0` para usar la config global. O `86400` para 1 día | ⏭️ Dejar en 0 |
| **Skip caching on dimension warnings** | No cachea teselas si hay advertencias relacionadas con dimensiones (tiempo/elevación). Evita cachear datos incompletos. | No aplica si no usas dimensiones. Dejar desactivado | ⏭️ Ignorar |

> **Consejo práctico:** Después de activar el caché, ve a **Tile Layers** en el menú principal de GeoServer y haz clic en **"Seed/Truncate"** para pre-generar todas las teselas. Así el primer usuario que vea el mapa no tendrá que esperar.

---

## 5. Pestaña: Seguridad

Controla quién puede leer (ver) y escribir (editar vía WFS-T) esta capa específica. Sobreescribe las reglas globales de seguridad del servidor.

| Parámetro | Qué es | Qué configurar | Acción |
|-----------|--------|-----------------|--------|
| **Permitir el acceso a cualquier rol** | Si está activado, cualquier usuario (autenticado o anónimo) puede acceder a la capa. Es la opción por defecto. | **Activar** si la capa es pública (caso normal para departamentos) | 🔴 Configurar |
| **ROLE_ANONYMOUS — Lectura** | Permite que usuarios no autenticados (público general) vean la capa vía WMS/WFS. | Activar para capas públicas | 🔴 Si es pública |
| **ROLE_ANONYMOUS — Escritura** | Permite que usuarios no autenticados modifiquen features vía WFS-T. | **Nunca activar.** Cualquiera podría editar tus departamentos | 🔴 No activar |
| **ROLE_AUTHENTICATED — Lectura** | Permite que usuarios con sesión iniciada vean la capa. | Activar | 🔴 Activar |
| **ROLE_AUTHENTICATED — Escritura** | Permite que usuarios autenticados editen vía WFS-T. | Solo si necesitas edición colaborativa controlada | 🔵 Opcional |
| **GROUP_ADMIN — Lectura/Escritura** | Acceso para el grupo de administradores. | Lectura: sí. Escritura: según necesidad | 🔵 Opcional |
| **ADMIN — Lectura/Escritura** | Acceso total para el rol de administrador del servidor. | Ambos activados por defecto | ⏭️ Dejar así |

> **Configuración típica para una capa pública:**
> - Dejar marcado "Permitir el acceso a cualquier rol"
> - Lectura: todos los roles
> - Escritura: solo ADMIN
>
> **Configuración para una capa restringida (ej: datos sensibles):**
> - Desmarcar "Permitir el acceso a cualquier rol"
> - Lectura: solo ROLE_AUTHENTICATED y ADMIN
> - Escritura: solo ADMIN

---

## Resumen general: qué configurar en cada pestaña

| Pestaña | Prioridad | Acciones clave |
|---------|-----------|----------------|
| **1. Datos** | 🔴 Obligatoria | Título, SRS, calcular encuadres |
| **2. Publicación** | 🔴 Obligatoria | Activar Interrogable, asignar estilo por defecto, configurar decimales WFS |
| **3. Dimensiones** | ⏭️ No aplica | Tu shapefile no tiene campos de fecha ni elevación |
| **4. Cacheado de Teselas** | ⚠️ Recomendada | Activar caché, seleccionar image/png, meta tiling 4×4 |
| **5. Seguridad** | 🔴 Obligatoria | Verificar que lectura sea pública y escritura solo admin |

---

*Documento de referencia para clases de GeoServer — ARTECLAB*
