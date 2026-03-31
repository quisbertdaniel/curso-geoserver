# 🗺️ Guía Maestra: Filtrado de Datos en Estilos SLD (GeoServer)

Esta guía explica cómo utilizar el estándar **OGC Filter Encoding** dentro de los archivos **SLD (Styled Layer Descriptor)**. Aprenderás a controlar qué elementos geográficos se dibujan en el mapa basándote en sus atributos y su geometría.

---

## 1. Introducción al Concepto de Filtro
En cartografía digital, no siempre queremos dibujar todos los datos de la misma forma. Un **Filtro** es una regla lógica que evalúa cada fila de nuestra base de datos (PostGIS, Shapefile, etc.). 
* Si la regla es **VERDADERA**, el elemento se dibuja con el estilo definido.
* Si es **FALSA**, el elemento se ignora o pasa a la siguiente regla.

---

## 2. Operadores de Comparación (Atributos)
Se utilizan para comparar una columna de datos (`PropertyName`) con un valor específico (`Literal`).

| Operador | Significado | Uso Pedagógico |
| :--- | :--- | :--- |
| `<PropertyIsEqualTo>` | **Igual a** | Para clasificar categorías exactas (ej. Tipo = 'Autopista'). |
| `<PropertyIsNotEqualTo>` | **Distinto de** | Para excluir elementos específicos del mapa. |
| `<PropertyIsLessThan>` | **Menor que** | Para representar rangos numéricos bajos. |
| `<PropertyIsGreaterThan>` | **Mayor que** | Para resaltar valores altos (ej. Población > 1,000,000). |
| `<PropertyIsBetween>` | **Está entre** | Ideal para rangos intermedios (requiere un límite inferior y superior). |
| `<PropertyIsLike>` | **Similar a** | Búsqueda parcial de texto (usa `*` como comodín). |
| `<PropertyIsNull>` | **Es nulo** | Para identificar dónde faltan datos en la tabla. |

---

## 3. Operadores Lógicos (Combinaciones)
A veces una sola condición no es suficiente. Los operadores lógicos nos permiten crear reglas complejas:

> 💡 **Ejemplo de lógica:** "Quiero ver los ríos que sean *principales* **Y** que tengan una *longitud mayor a 500km*."

* **`<And>`**: Todas las condiciones internas deben ser ciertas.
* **`<Or>`**: Basta con que una de las condiciones sea cierta.
* **`<Not>`**: Invierte el resultado (lo que era verdad ahora es falso).

---

## 4. El Filtro "Por Defecto" (`<ElseFilter/>`)
Este es un recurso pedagógico vital. El `<ElseFilter/>` actúa como una "red de seguridad". Se coloca en la última regla para atrapar todos los elementos que **no cumplieron** ninguna de las condiciones anteriores. 

*Evita que queden "huecos" o datos sin representar en tu mapa.*

---

## 5. Operadores Espaciales (Avanzado)
SLD no solo filtra por números o letras, ¡también por el espacio!

* **`<Intersects>`**: Filtra elementos que toquen una zona específica.
* **`<Within>`**: Filtra elementos que estén totalmente contenidos dentro de un polígono.
* **`<DWithin>`**: (Distance Within) Filtra elementos que se encuentren a una distancia "X" de un punto (ej. "Hospitales a menos de 5km").

---

## 6. Ejemplo Práctico: Clasificación de Ríos
A continuación, un ejemplo de cómo se estructuran estas reglas en el código:

```xml
<Rule>
  <ogc:Filter>
    <ogc:PropertyIsGreaterThan>
      <ogc:PropertyName>longitud_km</ogc:PropertyName>
      <ogc:Literal>500</ogc:Literal>
    </ogc:PropertyIsGreaterThan>
  </ogc:Filter>
  </Rule>

<Rule>
  <ElseFilter/>
  </Rule>
