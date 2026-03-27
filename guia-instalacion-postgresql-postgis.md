# PostgreSQL + PostGIS en servidor Linux — Guía resumida
> Instalación, configuración remota e importación de datos espaciales
> Para Debian 13 (Trixie) / Ubuntu 24.04 LTS

---

## ¿Qué vamos a instalar y por qué?

| Herramienta | ¿Qué es? | ¿Para qué la necesitamos? |
|-------------|----------|---------------------------|
| **PostgreSQL 17** | Motor de base de datos relacional. Es como MySQL pero más potente y con mejor soporte para datos complejos. | Almacenar nuestros datos geográficos de forma profesional, con consultas SQL |
| **PostGIS 3** | Extensión de PostgreSQL que le agrega capacidades espaciales (puntos, líneas, polígonos, reproyección, análisis geográfico). | Sin PostGIS, PostgreSQL no entiende geometrías ni puede hacer consultas espaciales |
| **gdal-bin** | Colección de herramientas de línea de comandos para leer, convertir y manipular datos geográficos (raster y vector). Incluye `ogr2ogr` y `ogrinfo`. | Convertir Shapefile → GeoPackage e importar datos a PostGIS desde la terminal |
| **ogr2ogr** | Comando que viene dentro de `gdal-bin`. Convierte datos vectoriales entre formatos (SHP, GPKG, PostGIS, GeoJSON, KML, etc.). | Es la "navaja suiza" para mover datos entre formatos y bases de datos |
| **ogrinfo** | Otro comando de `gdal-bin`. Muestra información sobre archivos geográficos sin abrirlos en un SIG. | Verificar que nuestras conversiones e importaciones salieron correctas |
| **GeoPackage (.gpkg)** | Formato de archivo basado en SQLite. Un solo archivo que puede contener múltiples capas, sin las limitaciones del Shapefile. | Reemplaza al Shapefile como formato intermedio moderno y sin límites |
| **psql** | Cliente de línea de comandos de PostgreSQL. Permite ejecutar consultas SQL desde la terminal. | Administrar la base de datos, crear usuarios, verificar datos importados |

---

## Paso 1 — Actualizar el sistema

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot    # Solo si se actualizó el kernel
```

---

## Paso 2 — Instalar PostgreSQL 17

### En Debian 13 (ya viene en los repos):

```bash
sudo apt install -y postgresql-17 postgresql-client-17 postgresql-contrib-17
```

> `postgresql-contrib-17` incluye utilidades extra como extensiones de texto, UUID y herramientas de mantenimiento.

### En Ubuntu 24.04 (necesita el repo oficial):

```bash
sudo apt install -y postgresql-common ca-certificates
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
sudo apt install -y postgresql-17 postgresql-client-17 postgresql-contrib-17
```

> El script `apt.postgresql.org.sh` agrega el repositorio oficial del proyecto PostgreSQL (PGDG), que siempre tiene la versión más reciente.

### Verificar:

```bash
sudo systemctl enable postgresql          # Que inicie con el servidor
sudo systemctl status postgresql           # Debe decir: active
sudo -u postgres psql -c "SELECT version();"  # Muestra la versión
```

---

## Paso 3 — Instalar PostGIS y GDAL

```bash
sudo apt install -y postgresql-17-postgis-3 postgresql-17-postgis-3-scripts gdal-bin
```

> `postgresql-17-postgis-3-scripts` incluye funciones SQL adicionales de PostGIS como geocodificación y topología.

### Verificar:

```bash
ogr2ogr --version    # Debe mostrar GDAL 3.x.x
```

---

## Paso 4 — Crear usuario, base de datos y habilitar PostGIS

```bash
sudo -u postgres psql
```

> `sudo -u postgres` ejecuta el comando como el usuario del sistema `postgres`, que es el superusuario creado automáticamente durante la instalación.

Dentro de `psql`:

```sql
-- 1. Contraseña para el superusuario (seguridad básica)
ALTER USER postgres PASSWORD 'SuperContraseña123!';

-- 2. Crear un usuario dedicado para tu proyecto
CREATE USER geouser WITH ENCRYPTED PASSWORD 'MiContraseñaGeo2026';

-- 3. Crear la base de datos y asignarle dueño
CREATE DATABASE geodb OWNER geouser;

-- 4. Conectarse a la nueva base de datos
\c geodb

-- 5. Habilitar PostGIS (convierte la BD en una BD espacial)
CREATE EXTENSION IF NOT EXISTS postgis;

-- 6. Verificar
SELECT PostGIS_Full_Version();

-- 7. Permisos completos para el usuario
GRANT ALL PRIVILEGES ON DATABASE geodb TO geouser;
GRANT USAGE ON SCHEMA public TO geouser;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO geouser;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO geouser;

\q
```

> **¿Por qué un usuario separado?** Nunca conectes aplicaciones (GeoServer, QGIS, visores web) como `postgres`. Si la conexión se compromete, un atacante tendría control total. `geouser` solo opera en su base de datos.

---

## Paso 5 — Configurar acceso remoto

Por defecto PostgreSQL solo acepta conexiones locales. Necesitamos cambiar **dos archivos**.

### 5.1 — Que escuche en todas las interfaces

```bash
sudo nano /etc/postgresql/17/main/postgresql.conf
```

Busca y cambia:

```ini
# Cambiar de:
#listen_addresses = 'localhost'
# A:
listen_addresses = '*'
```

> `listen_addresses = '*'` le dice a PostgreSQL que acepte conexiones TCP desde cualquier interfaz de red del servidor, no solo desde sí mismo.

### 5.2 — Permitir conexiones remotas con contraseña

```bash
sudo nano /etc/postgresql/17/main/pg_hba.conf
```

Agrega al final:

```conf
# Conexiones remotas — usuario geouser a la base geodb
# TIPO    BASE     USUARIO    DIRECCIÓN        MÉTODO
host      geodb    geouser    0.0.0.0/0        scram-sha-256
host      geodb    geouser    ::0/0            scram-sha-256
```

> **pg_hba.conf** es el archivo de reglas de acceso de PostgreSQL (HBA = Host-Based Authentication). Cada línea dice: "desde esta dirección, este usuario puede acceder a esta base usando este método de autenticación".
>
> `scram-sha-256` es el método más seguro disponible. Nunca uses `trust` (permite acceso sin contraseña) en conexiones remotas.
>
> `0.0.0.0/0` = cualquier IPv4. Para restringir a tu red local: `192.168.1.0/24`.
>
> `::0/0` = cualquier IPv6.

### 5.3 — Reiniciar y verificar

```bash
sudo systemctl restart postgresql
sudo ss -tlnp | grep 5432
# Debe mostrar: 0.0.0.0:5432 (escuchando en todas las interfaces)
```

> `ss -tlnp` muestra los puertos TCP en escucha. Si ves `127.0.0.1:5432`, el archivo `postgresql.conf` no se guardó bien.

### 5.4 — Abrir el firewall

```bash
sudo ufw allow 5432/tcp
sudo ufw status
```

> `ufw` (Uncomplicated Firewall) es el firewall por defecto en Debian/Ubuntu. Sin esta regla, el puerto 5432 estaría bloqueado aunque PostgreSQL esté escuchando.

### 5.5 — Probar desde otra máquina

```bash
psql -h IP_DEL_SERVIDOR -U geouser -d geodb -p 5432
```

Si ves el prompt `geodb=>`, la conexión remota funciona.

> **Si falla, verifica en este orden:**
> 1. ¿PostgreSQL corre? → `sudo systemctl status postgresql`
> 2. ¿Escucha en 0.0.0.0? → `sudo ss -tlnp | grep 5432`
> 3. ¿Firewall abierto? → `sudo ufw status`
> 4. ¿Regla en pg_hba.conf? → revisa IP y nombre de BD
> 5. ¿Hay conectividad? → `ping IP_DEL_SERVIDOR`

---

## Paso 6 — Convertir Shapefile a GeoPackage

### ¿Por qué convertir?

| Limitación del Shapefile | GeoPackage |
|--------------------------|-----------|
| Nombres de campo máx. 10 caracteres | Sin límite |
| Necesita 4-6 archivos (.shp, .dbf, .shx, .prj...) | 1 solo archivo .gpkg |
| Máx. 2 GB por archivo | Sin límite práctico |
| Un tipo de geometría por archivo | Múltiples capas y tipos |

### Conversión básica:

```bash
ogr2ogr -f GPKG departamentos.gpkg departamentos.shp
```

### Conversión profesional:

```bash
ogr2ogr \
  -f GPKG \
  -nlt MULTIPOLYGON \
  -nln departamentos \
  -lco FID=gid \
  -lco GEOMETRY_NAME=geom \
  departamentos.gpkg \
  departamentos.shp
```

| Parámetro | Qué hace |
|-----------|----------|
| `-f GPKG` | Formato de salida: GeoPackage |
| `-nlt MULTIPOLYGON` | Fuerza tipo MultiPolygon (evita mezcla Polygon/MultiPolygon que causa errores en PostGIS) |
| `-nln departamentos` | Nombre de la capa dentro del GeoPackage |
| `-lco FID=gid` | Nombre de la columna de ID → `gid` (convención PostGIS) |
| `-lco GEOMETRY_NAME=geom` | Nombre de la columna de geometría → `geom` (convención PostGIS) |

> `-lco` = Layer Creation Option (opción de creación de capa). Son instrucciones específicas para el formato de salida.

### Verificar:

```bash
ogrinfo -so departamentos.gpkg departamentos
```

> `-so` = Summary Only. Muestra resumen sin listar cada feature. Verás: tipo de geometría, cantidad de registros, extensión, SRS y lista de campos.

### Múltiples shapefiles en un solo GeoPackage:

```bash
ogr2ogr -f GPKG bolivia.gpkg departamentos.shp -nln departamentos
ogr2ogr -append -f GPKG bolivia.gpkg rios.shp -nln rios
ogr2ogr -append -f GPKG bolivia.gpkg ciudades.shp -nln ciudades
```

> `-append` agrega una nueva capa al GeoPackage existente sin borrar las anteriores.

### Alternativa desde QGIS:

1. Abre el shapefile en QGIS
2. Clic derecho → **Exportar → Guardar objetos como...**
3. Formato: **GeoPackage**, SRC: `EPSG:4326`, Capa: `departamentos`
4. Aceptar

---

## Paso 7 — Importar a PostGIS

### Método A: Desde terminal con ogr2ogr (recomendado)

#### Importar GeoPackage:

```bash
ogr2ogr \
  -f "PostgreSQL" \
  PG:"host=IP_SERVIDOR dbname=geodb user=geouser password=TuContraseña" \
  departamentos.gpkg \
  -nln departamentos \
  -nlt MULTIPOLYGON \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid \
  -lco PRECISION=NO \
  -overwrite \
  --config PG_USE_COPY YES
```

| Parámetro | Qué hace |
|-----------|----------|
| `-f "PostgreSQL"` | Destino: base de datos PostgreSQL |
| `PG:"..."` | Cadena de conexión (host, base, usuario, contraseña) |
| `-lco PRECISION=NO` | Evita crear campos numéricos con precisión fija (previene errores con algunos datos) |
| `-overwrite` | Si la tabla ya existe, la borra y la recrea |
| `--config PG_USE_COPY YES` | Usa el comando COPY de PostgreSQL en vez de INSERT fila por fila → **mucho más rápido** |

> `PG_USE_COPY YES` puede hacer una importación **10-50x más rápida** en tablas grandes. Siempre úsalo.

#### Importar Shapefile directo (sin pasar por GeoPackage):

```bash
ogr2ogr \
  -f "PostgreSQL" \
  PG:"host=IP_SERVIDOR dbname=geodb user=geouser password=TuContraseña" \
  departamentos.shp \
  -nln departamentos \
  -nlt PROMOTE_TO_MULTI \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid \
  -overwrite \
  --config PG_USE_COPY YES
```

> `-nlt PROMOTE_TO_MULTI` convierte automáticamente Polygon → MultiPolygon. PostGIS es estricto con los tipos; si tu shapefile mezcla ambos, la importación falla sin esto.

#### Importar con reproyección:

```bash
ogr2ogr \
  -f "PostgreSQL" \
  PG:"host=IP_SERVIDOR dbname=geodb user=geouser password=TuContraseña" \
  municipios_utm.shp \
  -nln municipios \
  -s_srs EPSG:32719 \
  -t_srs EPSG:4326 \
  -nlt PROMOTE_TO_MULTI \
  -lco GEOMETRY_NAME=geom \
  -lco FID=gid \
  -overwrite \
  --config PG_USE_COPY YES
```

> `-s_srs` = Source SRS (SRS de origen). `-t_srs` = Target SRS (SRS de destino). `ogr2ogr` reproyecta las coordenadas al vuelo durante la importación.

### Método B: Desde QGIS

1. **Conectar:** Panel del Navegador → PostgreSQL → Nueva conexión → llenar datos → Probar conexión
2. **Importar:** Menú Base de datos → Administrador de BD → Importar capa/archivo
   - Seleccionar archivo, tabla destino: `departamentos`, geometría: `geom`, FID: `gid`
   - ☑ Crear índice espacial → Aceptar
3. **Alternativa rápida:** Arrastra la capa desde el panel de Capas y suéltala sobre el schema de PostGIS en el panel del Navegador

---

## Paso 8 — Verificar la importación

```bash
psql -h IP_SERVIDOR -U geouser -d geodb
```

```sql
-- Ver todas las tablas espaciales
SELECT f_table_name, f_geometry_column, srid, type
FROM geometry_columns;

-- Contar registros
SELECT COUNT(*) FROM departamentos;

-- Ver datos de ejemplo
SELECT gid, departamen, ST_AsText(ST_Centroid(geom)) FROM departamentos LIMIT 3;

-- Verificar el SRID
SELECT ST_SRID(geom) FROM departamentos LIMIT 1;

\q
```

> `geometry_columns` es una vista especial de PostGIS que lista todas las tablas que tienen columnas de geometría, con su SRID y tipo.

---

## Paso 9 — Post-importación (buenas prácticas)

```sql
-- Crear índice espacial (acelera consultas geográficas)
CREATE INDEX IF NOT EXISTS idx_departamentos_geom
ON departamentos USING GIST (geom);

-- Actualizar estadísticas (el planificador de consultas necesita esto)
VACUUM ANALYZE departamentos;

-- Buscar y reparar geometrías inválidas
UPDATE departamentos SET geom = ST_MakeValid(geom) WHERE NOT ST_IsValid(geom);
```

> **Índice GIST:** Es un tipo de índice especializado para datos espaciales. Sin él, cada consulta geográfica escanea TODOS los polígonos. Con él, PostgreSQL descarta rápidamente los que no están en el área de interés.
>
> **VACUUM ANALYZE:** Le dice a PostgreSQL que recalcule las estadísticas internas sobre la distribución de los datos. Sin esto, el optimizador de consultas puede elegir estrategias ineficientes.
>
> **ST_MakeValid:** Repara geometrías con errores topológicos (auto-intersecciones, anillos invertidos). Estos errores son comunes en shapefiles exportados de distintos software.

---

## Paso 10 — Conectar GeoServer a PostGIS

1. GeoServer → **Almacenes de datos → Agregar nuevo almacén → PostGIS**
2. Llenar: host, puerto `5432`, base `geodb`, schema `public`, usuario `geouser`
3. Guardar → Aparece la lista de tablas → **Publicar** la que necesites

---

## Script rápido: importar todos los shapefiles de un directorio

```bash
#!/bin/bash
PG="host=IP_SERVIDOR dbname=geodb user=geouser password=TuContraseña"

for SHP in /ruta/a/shapefiles/*.shp; do
    NOMBRE=$(basename "$SHP" .shp | tr '[:upper:]' '[:lower:]' | tr ' -' '__')
    echo "Importando: $NOMBRE"
    ogr2ogr -f "PostgreSQL" PG:"$PG" "$SHP" \
      -nln "$NOMBRE" -nlt PROMOTE_TO_MULTI \
      -lco GEOMETRY_NAME=geom -lco FID=gid \
      -overwrite --config PG_USE_COPY YES
done
```

> El `tr` convierte el nombre a minúsculas y reemplaza espacios/guiones por guiones bajos, porque PostgreSQL no maneja bien nombres con mayúsculas o espacios sin comillas.

---

## Referencia rápida de comandos

| Quiero... | Comando |
|-----------|---------|
| Ver versión de PostgreSQL | `sudo -u postgres psql -c "SELECT version();"` |
| Ver versión de PostGIS | `psql -c "SELECT PostGIS_Full_Version();"` |
| Ver versión de GDAL | `ogr2ogr --version` |
| Inspeccionar un shapefile | `ogrinfo -so archivo.shp nombre_capa` |
| Convertir SHP → GPKG | `ogr2ogr -f GPKG salida.gpkg entrada.shp` |
| Importar a PostGIS | `ogr2ogr -f "PostgreSQL" PG:"host=... dbname=..." archivo.gpkg` |
| Ver tablas espaciales | `SELECT * FROM geometry_columns;` |
| Estado de PostgreSQL | `sudo systemctl status postgresql` |
| Reiniciar PostgreSQL | `sudo systemctl restart postgresql` |
| Ver puertos abiertos | `sudo ss -tlnp \| grep 5432` |
| Estado del firewall | `sudo ufw status` |

---

*ARTECLAB — Material de referencia para clases de GeoServer + PostGIS*
