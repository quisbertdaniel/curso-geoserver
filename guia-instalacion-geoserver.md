# Guía de Instalación — Java 21 + Tomcat 9

> **Curso GeoServer · ARTECLAB**
> Prerrequisitos para instalar GeoServer 2.28.2

---

## ¿Qué vamos a instalar y por qué?

GeoServer es una aplicación Java que corre dentro de un servidor de aplicaciones. Necesitas dos cosas antes de instalar GeoServer:

| Software | ¿Qué es? | ¿Por qué lo necesitas? |
|----------|----------|----------------------|
| **Java 21 (OpenJDK)** | El lenguaje en el que está escrito GeoServer | Sin Java, GeoServer no arranca. Usamos OpenJDK 21 LTS (gratuito, de código abierto). |
| **Apache Tomcat 9** | El servidor de aplicaciones que ejecuta GeoServer | Es donde "vive" GeoServer. Recibe peticiones HTTP y las envía a GeoServer. |

GeoServer 2.28.2 es compatible con Java 17 y Java 21. Usamos **Java 21** porque es la versión LTS más reciente y tendrá soporte hasta 2029.

---

## Parte 1 — Instalar Java 21 (OpenJDK)

Usamos **Eclipse Temurin** de Adoptium — la distribución OpenJDK más confiable y usada en producción.

### Windows

**Paso 1 — Descargar el instalador**

Ir a: **https://adoptium.net/temurin/releases/**

Seleccionar:
- Operating System: **Windows**
- Architecture: **x64**
- Package Type: **JDK**
- Version: **21 - LTS**

Descargar el archivo `.msi` (instalador de Windows).

**Paso 2 — Ejecutar el instalador**

1. Doble clic en el archivo `.msi` descargado.
2. En la pantalla de opciones, **marcar las 3 casillas**:
   - ✅ Add to PATH
   - ✅ Set JAVA_HOME
   - ✅ JavaSoft (Oracle) registry keys
3. Clic en **Next** → **Install** → **Finish**.

**Paso 3 — Verificar la instalación**

Abrir una **nueva** ventana de terminal (PowerShell o CMD) y ejecutar:

```
java -version
```

Deberías ver algo similar a:

```
openjdk version "21.0.10" 2026-01-21 LTS
OpenJDK Runtime Environment Temurin-21.0.10+7 (build 21.0.10+7-LTS)
OpenJDK 64-Bit Server VM Temurin-21.0.10+7 (build 21.0.10+7-LTS, mixed mode, sharing)
```

Verificar que JAVA_HOME está configurado:

```
echo %JAVA_HOME%
```

Debe mostrar una ruta como `C:\Program Files\Eclipse Adoptium\jdk-21.0.10.7-hotspot`.

> **⚠️ Importante:** Si `java -version` dice "no se reconoce como comando", cierra TODAS las ventanas de terminal y abre una nueva. El instalador actualiza el PATH pero las terminales abiertas no lo ven.

---

### Linux (Ubuntu / Debian)

**Opción A — Desde el repositorio de Adoptium (recomendado)**

```bash
# 1. Instalar dependencias
sudo apt update
sudo apt install -y wget apt-transport-https gpg

# 2. Agregar la clave GPG de Adoptium
wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | \
  gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/adoptium.gpg > /dev/null

# 3. Agregar el repositorio
echo "deb https://packages.adoptium.net/artifactory/deb $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/adoptium.list

# 4. Instalar Java 21
sudo apt update
sudo apt install -y temurin-21-jdk

# 5. Verificar
java -version
```

**Opción B — Desde el repositorio de Ubuntu (más simple)**

```bash
sudo apt update
sudo apt install -y openjdk-21-jdk

java -version
```

**Configurar JAVA_HOME (si no se configuró automáticamente)**

```bash
# Buscar dónde está instalado Java
sudo update-alternatives --config java
# Copiar la ruta (sin el /bin/java al final)

# Editar el perfil
sudo nano /etc/environment
# Agregar esta línea (ajustar la ruta según tu instalación):
JAVA_HOME="/usr/lib/jvm/temurin-21-jdk-amd64"

# Aplicar los cambios
source /etc/environment

# Verificar
echo $JAVA_HOME
```

---

## Parte 2 — Instalar Apache Tomcat 9

### ¿Por qué Tomcat 9 y no Tomcat 10?

Tomcat 10 migró a Jakarta EE (cambió los paquetes de `javax.*` a `jakarta.*`). GeoServer 2.28.2 todavía usa la API `javax.servlet`, por lo que **solo es compatible con Tomcat 9**. Si instalas Tomcat 10, GeoServer no arranca.

### Windows

**Paso 1 — Descargar Tomcat 9**

Ir a: **https://tomcat.apache.org/download-90.cgi**

En la sección **Binary Distributions → Core**, descargar:
- **64-bit Windows zip** (recomendado, más control)
- O **32-bit/64-bit Windows Service Installer** (si prefieres que se instale como servicio)

**Paso 2 — Instalar (versión ZIP)**

1. Descomprimir el ZIP en una ubicación simple, por ejemplo:
   ```
   C:\tomcat9
   ```
2. Verificar que la estructura sea:
   ```
   C:\tomcat9\
   ├── bin\
   ├── conf\
   ├── lib\
   ├── logs\
   ├── temp\
   ├── webapps\
   └── work\
   ```

**Paso 3 — Configurar memoria para GeoServer**

GeoServer necesita más memoria de la que Tomcat asigna por defecto. Crear el archivo:

```
C:\tomcat9\bin\setenv.bat
```

Con este contenido:

```bat
set CATALINA_OPTS=-Xms512m -Xmx2048m -XX:+UseG1GC
```

Esto le da a GeoServer entre 512 MB y 2 GB de memoria RAM.

**Paso 4 — Iniciar Tomcat**

```
cd C:\tomcat9\bin
startup.bat
```

Abrir en el navegador: **http://localhost:8080**

Si ves la página de bienvenida de Apache Tomcat, está funcionando.

**Paso 5 — Detener Tomcat**

```
cd C:\tomcat9\bin
shutdown.bat
```

> **⚠️ Si ves "JAVA_HOME is not defined":** Verifica que el Paso 2 de Java se completó correctamente. JAVA_HOME debe apuntar a la carpeta del JDK (no al bin).

---

### Linux (Ubuntu / Debian)

**Opción A — Desde repositorio APT (más simple)**

```bash
# 1. Instalar Tomcat 9
sudo apt update
sudo apt install -y tomcat9 tomcat9-admin

# 2. Verificar que está corriendo
sudo systemctl status tomcat9

# 3. Habilitar inicio automático
sudo systemctl enable tomcat9
```

**Opción B — Instalación manual (más control)**

```bash
# 1. Descargar Tomcat 9
cd /opt
sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.102/bin/apache-tomcat-9.0.102.tar.gz
sudo tar xzf apache-tomcat-9.0.102.tar.gz
sudo mv apache-tomcat-9.0.102 tomcat9
sudo rm apache-tomcat-9.0.102.tar.gz

# 2. Crear usuario del sistema para Tomcat
sudo useradd -r -m -U -d /opt/tomcat9 -s /bin/false tomcat
sudo chown -R tomcat:tomcat /opt/tomcat9

# 3. Iniciar
sudo /opt/tomcat9/bin/startup.sh
```

**Configurar memoria (ambas opciones)**

```bash
# Si instalaste con APT:
sudo nano /usr/share/tomcat9/bin/setenv.sh

# Si instalaste manual:
sudo nano /opt/tomcat9/bin/setenv.sh
```

Escribir:

```bash
export CATALINA_OPTS="-Xms512m -Xmx2048m -XX:+UseG1GC"
```

Guardar (Ctrl+O, Enter, Ctrl+X) y reiniciar:

```bash
# Con APT:
sudo systemctl restart tomcat9

# Manual:
sudo /opt/tomcat9/bin/shutdown.sh
sudo /opt/tomcat9/bin/startup.sh
```

**Verificar:** Abrir **http://localhost:8080** — debe mostrar la página de Tomcat.

---

## Parte 3 — Verificación Final

Antes de instalar GeoServer, verifica que todo está correcto:

### Checklist

```
✅ java -version  →  muestra "openjdk version 21.x.x"
✅ echo JAVA_HOME  →  muestra la ruta del JDK
✅ http://localhost:8080  →  muestra la página de Tomcat
✅ Tomcat tiene setenv configurado con al menos -Xmx1024m
```

Si los 4 puntos están bien, estás listo para instalar GeoServer 2.28.2.

---

## Solución de Problemas

### Java

| Problema | Solución |
|----------|----------|
| `java` no se reconoce como comando | Cerrar la terminal y abrir una nueva. Si persiste, verificar que se marcó "Add to PATH" durante la instalación. En Linux: `sudo update-alternatives --config java` |
| JAVA_HOME no está configurado | **Windows:** Panel de control → Variables de entorno → Nueva variable: `JAVA_HOME` = ruta del JDK. **Linux:** Editar `/etc/environment` |
| Tengo otra versión de Java instalada | Puedes tener varias versiones. Asegúrate de que `java -version` muestre la 21. En Linux: `sudo update-alternatives --config java` para elegir |

### Tomcat

| Problema | Solución |
|----------|----------|
| Puerto 8080 ocupado | Otro programa lo usa. **Windows:** `netstat -ano \| findstr 8080` para identificarlo. **Linux:** `sudo lsof -i :8080`. Cerrar el programa o cambiar el puerto de Tomcat en `conf/server.xml` |
| "JAVA_HOME is not defined" | JAVA_HOME no está configurado. Ver la sección de verificación de Java |
| Tomcat arranca pero la página no carga | Esperar 30-60 segundos. Revisar logs: **Windows:** `C:\tomcat9\logs\catalina.out` **Linux:** `/var/log/tomcat9/catalina.out` o `/opt/tomcat9/logs/catalina.out` |
| Error de permisos (Linux) | `sudo chown -R tomcat:tomcat /opt/tomcat9` o `sudo chown -R tomcat:tomcat /var/lib/tomcat9` |

---

## Enlaces de Referencia

| Recurso | URL |
|---------|-----|
| Adoptium (OpenJDK 21) | https://adoptium.net/temurin/releases/ |
| Apache Tomcat 9 | https://tomcat.apache.org/download-90.cgi |
| GeoServer 2.28.2 WAR | https://geoserver.org/release/stable/ |
| Java + GeoServer compatibilidad | https://docs.geoserver.org/stable/en/user/production/java.html |
| GeoServer instalación WAR | https://docs.geoserver.org/stable/en/user/installation/war.html |

---

## Siguiente Paso

Con Java 21 y Tomcat 9 instalados, estás listo para instalar GeoServer 2.28.2. Descarga el archivo WAR desde https://geoserver.org/release/stable/ y cópialo a la carpeta `webapps` de Tomcat. Los detalles completos están en el Módulo 1 del curso.

---

*Guía de instalación · Curso GeoServer · ARTECLAB · Daniel Quisbert · 2026*
