# 🗺️ Guía Maestra de Estilos SLD y Filtrado OGC en GeoServer

Esta guía técnica proporciona una referencia completa sobre el uso de **OGC Filter Encoding** dentro de los archivos **SLD (Styled Layer Descriptor)**. Es un recurso esencial para estudiantes de ingeniería, geografía y desarrolladores GIS que buscan dominar la simbología dinámica en servidores de mapas.

---

## 📖 1. Fundamentos del Filtrado
En GeoServer, un filtro es una expresión lógica que determina qué objetos espaciales (features) reciben un estilo específico. Se basa en el estándar **XML**, por lo que la jerarquía y el cierre de etiquetas son fundamentales.

La estructura básica de una regla con filtro es:
1.  **`<Rule>`**: Contenedor de la lógica y el estilo.
2.  **`<ogc:Filter>`**: Define la condición.
3.  **`<Symbolizer>`**: Define cómo se ve el dato si cumple la condición (Punto, Línea o Polígono).

---

## 📑 2. Catálogo de Operadores Lógicos y de Comparación

### A. Comparación de Atributos (Numéricos y Texto)
| Etiqueta OGC | Descripción | Ejemplo de Uso |
| :--- | :--- | :--- |
| `<ogc:PropertyIsEqualTo>` | **Igual a** | `ESTADO = 'Activo'` |
| `<ogc:PropertyIsNotEqualTo>` | **Distinto de** | `CATEGORIA != 'Temporal'` |
| `<ogc:PropertyIsGreaterThan>` | **Mayor que** | `POBLACION > 5000` |
| `<ogc:PropertyIsLessThan>` | **Menor que** | `DISTANCIA < 100` |
| `<ogc:PropertyIsBetween>` | **Rango inclusivo** | `ALTURA` entre 10 y 20 |
| `<ogc:PropertyIsLike>` | **Búsqueda parcial** | `NOMBRE` empieza con 'San' |
| `<ogc:PropertyIsNull>` | **Valor Nulo** | Detectar campos vacíos |

### B. Operadores Lógicos (Combinación)
Para consultas complejas, envolvemos los operadores anteriores en:
* **`<ogc:And>`**: Se deben cumplir todas las condiciones.
* **`<ogc:Or>`**: Se debe cumplir al menos una.
* **`<ogc:Not>`**: Niega la condición (selecciona lo opuesto).

---

## 🛰️ 3. Operadores Espaciales
GeoServer permite filtrar por geometría. Estos son los más comunes:

1.  **`<ogc:BBOX>`**: Filtra elementos dentro de un cuadro delimitador (Bounding Box).
2.  **`<ogc:Intersects>`**: Selecciona elementos que se tocan o cruzan.
3.  **`<ogc:DWithin>`**: Filtra elementos a una **distancia máxima** de otro punto o línea (útil para áreas de influencia).

---

## 🛠️ 4. Ejemplo Práctico: Clasificación de una Red Hidrográfica
Este ejemplo muestra cómo organizar niveles de jerarquía en un solo archivo SLD.

```xml
<Rule>
  <Name>rio_principal</Name>
  <Title>Ríos Mayores a 500km</Title>
  <ogc:Filter>
    <ogc:PropertyIsGreaterThan>
      <ogc:PropertyName>longitud_km</ogc:PropertyName>
      <ogc:Literal>500</ogc:Literal>
    </ogc:PropertyIsGreaterThan>
  </ogc:Filter>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke">#0000FF</CssParameter>
      <CssParameter name="stroke-width">3</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>

<Rule>
  <Name>rio_secundario</Name>
  <ogc:Filter>
    <ogc:And>
      <ogc:PropertyIsGreaterThanOrEqualTo>
        <ogc:PropertyName>longitud_km</ogc:PropertyName>
        <ogc:Literal>100</ogc:Literal>
      </ogc:PropertyIsGreaterThanOrEqualTo>
      <ogc:PropertyIsLessThanOrEqualTo>
        <ogc:PropertyName>longitud_km</ogc:PropertyName>
        <ogc:Literal>500</ogc:Literal>
      </ogc:PropertyIsLessThanOrEqualTo>
    </ogc:And>
  </ogc:Filter>
  <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke">#4A90E2</CssParameter>
      <CssParameter name="stroke-width">1.5</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>

<Rule>
  <Name>otros</Name>
  <Title>Cuerpos de agua menores</Title>
  <ElseFilter/> <LineSymbolizer>
    <Stroke>
      <CssParameter name="stroke">#A4C2F4</CssParameter>
      <CssParameter name="stroke-width">0.5</CssParameter>
      <CssParameter name="stroke-dasharray">4 2</CssParameter>
    </Stroke>
  </LineSymbolizer>
</Rule>
