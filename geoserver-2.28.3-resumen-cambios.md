# GeoServer 2.28.3 — Resumen de cambios (20 marzo 2026)

> Versión de mantenimiento. No trae funciones nuevas grandes, pero sí correcciones importantes y mejoras menores.
> Se lanza junto con GeoTools 34.3 y GeoWebCache 1.28.3.

---

## Mejoras

| Código | Cambio | Qué significa en la práctica |
|--------|--------|------------------------------|
| GEOS-11886 | Archivos `.properties` ordenados alfabéticamente | Los archivos de configuración internos ahora están ordenados de la A a la Z. Esto no cambia funcionalidad, pero facilita encontrar parámetros cuando editas archivos de configuración manualmente y reduce conflictos al usar Git para versionar configuraciones. |
| GEOS-12033 | Configurar autoridades CRS y transformaciones personalizadas | Ahora puedes registrar tus propios códigos de sistema de coordenadas (más allá de los EPSG estándar) y definir transformaciones personalizadas entre ellos. Útil si trabajas con sistemas de coordenadas locales de Bolivia que no están en el catálogo EPSG oficial. |
| GEOS-12037 | Soporte de Metatiling en MapBox Vector Tiles | Los vector tiles (formato .mvt de Mapbox) ahora se pueden generar con metatiling, es decir, GeoServer genera una tesela grande y la corta en partes. Reduce artefactos en bordes y mejora el rendimiento al servir vector tiles. Antes esto solo funcionaba para teselas de imagen (PNG/JPEG). |

---

## Correcciones de errores

| Código | Error corregido | Qué pasaba antes |
|--------|----------------|-------------------|
| GEOS-11964 | Error en operaciones masivas de metadatos | Al intentar editar metadatos en lote (muchas capas a la vez), GeoServer mostraba un error de Wicket (el framework de la interfaz web) y la operación fallaba. Ahora funciona correctamente. |
| GEOS-12038 | Problema con filtros y ModificationProxy | Cuando usabas filtros CQL o ECQL en ciertas configuraciones, GeoServer no los traducía correctamente al lenguaje nativo de la base de datos. Esto podía hacer que consultas WFS fueran más lentas porque se filtraban en memoria en vez de en la base de datos. |
| GEOS-12047 | Problemas con bloqueos (locks) anidados | En edición transaccional (WFS-T), si un usuario bloqueaba un feature para editarlo y otro proceso intentaba un bloqueo dentro del mismo, el servidor podía quedarse trabado o no respetar el tiempo de expiración del bloqueo. |
| GEOS-12055 | Caché de seguridad no se limpiaba al recargar | Al cambiar configuración de seguridad y hacer reload, los cambios no se aplicaban inmediatamente porque el caché de servicios no se limpiaba. Tenías que reiniciar GeoServer completo para que los nuevos permisos tomaran efecto. Ahora el reload funciona correctamente. |
| GEOS-12060 | REST API PUT no permitía vaciar campos | Al actualizar una capa o almacén vía REST API con PUT, no podías dejar un campo vacío (por ejemplo, quitar un resumen). El servidor ignoraba el valor nulo y mantenía el anterior. |

---

## Tareas de mantenimiento

| Código | Cambio | Qué significa |
|--------|--------|---------------|
| GEOS-12027 | Eliminación de dependencia `restlet.fileupload` | Se quitó una librería que ya no se usaba. Reduce el tamaño del servidor y superficie de posibles vulnerabilidades. No afecta funcionalidad. |
| GEOS-12028 | Actualización de librería Gson | Se actualizó la librería de Google para manejo de JSON. Incluye correcciones de seguridad y rendimiento. |
| GEOS-12029 | Actualización de librería Protobuf | Se actualizó la librería de Google Protocol Buffers (usada internamente para serialización de datos). Correcciones de seguridad. |
| GEOS-12049 | Eliminación del caché en memoria de GWC | Se quitó el soporte de caché InMemory de GeoWebCache. Si lo usabas, ahora debes usar caché en disco o un BlobStore externo. La mayoría de instalaciones usan disco, así que probablemente no te afecta. |

---

## Módulos de la comunidad (experimentales)

Estos no vienen incluidos por defecto. Son código fuente que puedes compilar si te interesa probarlos.

| Código | Módulo | Para qué sirve |
|--------|--------|-----------------|
| GEOS-11509 | OGC API 3D GeoVolumes | Nuevo módulo para servir datos geoespaciales en 3D siguiendo el estándar OGC API. Permite publicar modelos 3D de edificios, terrenos, etc. Etapa experimental. |
| GEOS-12002 | hz-cluster: error en popup de inicio | Corrección para el módulo de clustering con Hazelcast. El popup de bienvenida fallaba en configuraciones de múltiples nodos. |
| GEOS-12030 | Conflicto entre Features Templating y GeoFence | Si usabas plantillas de features (para personalizar respuestas GetFeatureInfo) junto con GeoFence (control de acceso avanzado), los tags XML entraban en conflicto. |
| GEOS-12044 | STAC: mejor manejo de errores | El endpoint de búsqueda STAC (catálogo espacio-temporal para imágenes satelitales) ahora reporta nombres de colección inválidos como errores de parámetro en vez de errores internos del servidor (500). |
| GEOS-12061 | Formato PNG-WIND para datos de viento | Nuevo módulo que permite exportar datasets de viento en un formato PNG especial donde los píxeles codifican dirección y velocidad del viento. Útil para visualización de datos meteorológicos en visores web. |

---

## ¿Te afecta esta actualización?

Para tu caso con ARTECLAB y los cursos de GeoServer, los cambios más relevantes son:

- **GEOS-12033** (CRS personalizados): útil si trabajas con sistemas de coordenadas bolivianos no estándar
- **GEOS-12037** (Metatiling en vector tiles): relevante si estás sirviendo vector tiles para tus apps Flutter con mapas
- **GEOS-12055** (fix de seguridad/reload): importante si cambias permisos de capas sin reiniciar el servidor
- **GEOS-12049** (eliminación caché en memoria): verificar que tu instalación no use InMemory cache antes de actualizar

El resto son correcciones menores que no requieren acción de tu parte.

---

*Documento de referencia para clases de GeoServer — ARTECLAB*
