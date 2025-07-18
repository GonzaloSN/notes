# 🗺️ Tema 1: Desarrollo Web orientado a GIS

Este documento es una introducción práctica para programadores que deseen incorporar visualización geográfica en la web usando tecnologías **open source**. No necesitas ser geógrafo: con saber de programación y entender los conceptos claves de SIG (GIS), puedes crear soluciones muy potentes.

Estas son herramientas que yo he usado, pero si eres curioso, ¡podrás encontrar muchas más!.
---

## 🧰 Tecnologías Web GIS

### 🌿 Leaflet
**¿Qué es?** Una librería JavaScript open source para mapas interactivos.

Siempre he utilizado Leaflet porque encuentro que es una herramienta fabulosa, con una gran comunidad y un ecosistema amplio de plugins que la hacen ideal para empezar a trabajar con mapas.

Sin embargo, en una ocasión tuve que mostrar etiquetas en una capa vectorial utilizando un servidor de tiles. Ahí fue cuando tuve que migrar a otra tecnología. Aunque logré implementar la funcionalidad mediante un script que manejaba varias capas, el código creció mucho en complejidad. Esto terminó afectando el rendimiento: la experiencia de usuario fue muy pobre, con renderizados lentos y el navegador llegando a congelarse.

*Aunque L.VectorGrid, la extensión de Leaflet que utilicé para renderizar capas vectoriales en formato tile, sí permite manejar eventos como hover y click sobre los features, no cuenta con un método `pointToLayer` que permita personalizar la representación de cada punto ni facilita mantener etiquetas permanentemente visibles. Esto se debe a que L.VectorGrid trabaja procesando y renderizando los datos en fragmentos tileados para optimizar el rendimiento, lo que limita la manipulación individual de cada objeto GeoJSON. En mi caso, esta limitación fue crítica porque necesitaba que las etiquetas estuvieran fijas y visibles en todo momento además controlar el nivel de zoom. Para lograr ese nivel de control, generalmente se debe usar L.geoJSON u otra capa que soporte la manipulación directa de objetos GeoJSON, aunque con un costo mayor en rendimiento para grandes volúmenes de datos.*

**Ventajas:**
- Liviana (~40KB).
- Fácil de usar.
- Amplia comunidad y plugins.

**Ideal para:** proyectos donde necesitas flexibilidad y velocidad.

---

### 🧭 MapLibre GL JS
**¿Qué es?** Una librería open source para mapas vectoriales renderizados en el navegador (WebGL).

Si leíste mi experiencia anterior, MapLibre GL JS solucionó ese problema 😎. Esta librería surge como un fork open source de Mapbox GL tras su cambio a modelo privado. Es una herramienta poderosa, especialmente diseñada para trabajar con vector tiles, ya que aprovecha la GPU para el renderizado de mapas, ofreciendo un rendimiento y fluidez excepcionales.

Su modelo de trabajo es algo diferente al de Leaflet, por lo que al principio puede resultar desafiante adaptarse. Sin embargo, vale totalmente la pena el esfuerzo, sobre todo cuando empiezas a manejar grandes volúmenes de datos y necesitas un desempeño óptimo.

**Ventajas:**
- Soporte para **vector tiles**.
- Estilos personalizables estilo Mapbox.
- Fluidez en zoom y rotación.

**Ideal para:** mapas modernos con alto rendimiento y estilo.

---

## 📐 Sistemas de Coordenadas para Programadores

Cuando trabajas con mapas, necesitas saber en qué "lenguaje" están las coordenadas de tus datos. Esto se llama referenciar los datos. Al principio puede sonar complicado, pero aquí te dejo lo básico para arrancar sin miedo:

### 🗺️ ¿Qué es WGS84 o EPSG:4326?
- Es el sistema más común y sencillo.
- Usa latitud y longitud para ubicar puntos en el planeta.
- Es el estándar en casi todas las APIs web y GPS.

### 📏 ¿Qué son las coordenadas proyectadas?
- Son coordenadas planas (como en un plano X y Y). Ideal para cálculos de distancia o áreas.
- Se usan para medir distancias o áreas con precisión.
- Por ejemplo, en Chile se usa EPSG:32719 (UTM zona 19S).
- Si usas mapas web, casi siempre tendrás que convertir estas coordenadas a WGS84.

### 📚 ¿Qué debe saber un programador?
- Siempre identifica en qué sistema están tus datos geográficos.
- Usa bibliotecas como **proj**, **pyproj**, **GDAL** o **PostGIS** para transformar coordenadas.
- Las librerías de mapas web como Leaflet o MapLibre esperan que uses coordenadas en WGS84 (EPSG:4326 o EPSG:3857).

---

## 🔌 Tipos de servicios GIS

Los servidores de mapas ofrecen diferentes tipos de servicios que las aplicaciones web pueden usar para mostrar mapas. En otras palabras, ¿cómo puedo tomar un mapa que está en un GeoServer o guardado en algún lugar y mostrarlo dentro del mapa de mi sitio web?

| Tipo | Sigla | ¿Qué entrega? | Usado en |
|------|-------|----------------|----------|
| WMS  | Web Map Service | Imágenes de mapas (raster) | Leaflet, QGIS |
| WFS  | Web Feature Service | Datos vectoriales (GeoJSON, GML) | apps que consultan geometrías |
| WMTS | Web Map Tile Service | Mosaicos de imágenes en tiles | rendimiento mejorado |
| MVT  | Mapbox Vector Tiles | Tiles vectoriales renderizados en cliente | MapLibre, Mapbox |

Un ejemplo básico para mostrar una capa que está alojado en GeoServer, mediante WMS:

```bash
// Crear el mapa centrado en coordenadas y nivel de zoom deseado
const map = L.map('map').setView([-33.45, -70.65], 10);

// Agregar capa base (OpenStreetMap)
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  attribution: '© OpenStreetMap contributors'
}).addTo(map);

// Agregar capa WMS desde GeoServer (ejemplo)
const wmsLayer = L.tileLayer.wms('https://tuservidor/geoserver/wms', {
  layers: 'nombre_del_workspace:nombre_de_la_capa',
  format: 'image/png',
  transparent: true,
  attribution: '© Tu GeoServer',
  tileSize: 256
}).addTo(map);
```

---

## 🧱 Servidores GIS open source
¿Qué es esto? En pocas palabras, es una herramienta que te permite alojar mapas en internet y acceder a ellos de forma remota desde tu aplicación o sitio web.

Uno de los más usados es, sin duda, GeoServer. Su principal desventaja es que requiere más recursos que una app común: se recomienda al menos 8 GB de RAM para que funcione de manera fluida, ya que funciona sobre una máquina virtual Java (JVM). Si cuentas con eso, tu instancia de GeoServer debería funcionar sin problemas. Es una herramiento excepcional además cuenta con una interfaz de usuario.

Un servidor que he estado utilizando últimamente es pg_tileserv, un geoservidor liviano y eficiente que permite exponer capas vectoriales como tiles directamente desde una base de datos PostGIS. Es una herramienta muy potente, ya que al estar integrada al ecosistema PostgreSQL, aprovecha al máximo las capacidades de PostGIS, especialmente en lo que respecta a consultas espaciales complejas y procesamiento topológico robusto.

También he oído hablar de `MapServer`, una herramienta muy utilizada por instituciones grandes como la NASA para ciertos tipos de tareas geoespaciales. Aún no he tenido la necesidad de migrar hacia él, pero está en mi radar: me gustaría hacer algunas pruebas o experimentar con su funcionamiento para ver qué tan bien se adapta a distintos flujos de trabajo.

### 🌐 GeoServer
- Permite publicar capas WMS, WFS, WMTS.
- Panel web intuitivo.
- Soporta estilos SLD.
- Escrito en Java
- **Incluye GWC** (GeoWebCache) para cachear tiles WMTS.

### 🚀 pg_tileserv
- Servidor muy liviano para publicar capas **MVT** directamente desde **PostGIS**.
- Ideal cuando trabajas con grandes volúmenes de datos vectoriales.

---

### 🐳 Cómo levanto ambos servicios en Docker (usando docker-compose)

```bash
  pg_tileserv:
    <<: *default-settings
    image: pramsey/pg_tileserv:latest
    container_name: pg_tileserv4${PROJECT_NAME}
    environment:
      <<: *default-tz-env
      DATABASE_URL: postgresql://postgres:postgres@db:5432/dbname
    ports:
      - "7800:7800"
    depends_on:
      - db

  geoserver:
    <<: *default-settings
    image: kartoza/geoserver:2.26.1
    container_name: geoserver4${PROJECT_NAME}
    environment:
      <<: *default-tz-env
    volumes:
      - geoserver-data:/opt/geoserver/data_dir
    ports:
      - "8080:8080"
```

Sugiero mucho el uso de pg_tileserv (GWC de GeoServer hace lo mismo, pero no lo he usado) cuando tengas que trabajar con vectores de grandes volúmenes de datos, En segundos, tenía mis capas sirviéndose como MVT, con respuesta inmediata y compatible con MapLibre GL JS y Leaflet (acá debes usar un plugin).
Muy útil para visualizaciones interactivas y mapas dinámicos.

## 🗂️ Bases de datos espaciales

Una de las herramientas más potentes y utilizadas en el mundo de los datos geoespaciales es PostgreSQL junto a su extensión espacial PostGIS. Esta combinación convierte a PostgreSQL en una base de datos espacial de alto rendimiento, ideal para almacenar, consultar y analizar geometrías de manera eficiente.

**PostGI**S extiende las capacidades del motor relacional con funciones específicas para trabajar con datos espaciales, como puntos, líneas, polígonos y geometrías más complejas. Su fortaleza principal radica en su robusto sistema de funciones para el análisis espacial y procesamiento topológico.


#### 🔧 Algunos de los métodos más utilizados:

| Función PostGIS | Descripción |
|-----------------|-------------|
| `ST_Intersects(geomA, geomB)` | Verifica si dos geometrías se intersectan |
| `ST_Within(geomA, geomB)` | Verifica si una geometría está completamente dentro de otra |
| `ST_Distance(geomA, geomB)` | Calcula la distancia entre dos geometrías |
| `ST_Union(geomSet)` | Une múltiples geometrías en una sola |
| `ST_Buffer(geom, radius)` | Crea un buffer (área de influencia) alrededor de una geometría |
| `ST_Contains(geomA, geomB)` | Verifica si una geometría contiene completamente a otra |
| `ST_Area(geom)` | Calcula el área de una geometría (polígono) |
| `ST_Length(geom)` | Calcula la longitud de una geometría lineal |
| `ST_Touches(geomA, geomB)` | Verifica si dos geometrías se tocan pero no se solapan |
| `ST_DWithin(geomA, geomB, distance)` | Verifica si dos geometrías están dentro de una distancia determinada |

📘 Puedes revisar toda la documentación oficial de PostGIS aquí:

👉 [https://postgis.net/docs/](https://postgis.net/docs/)

### 📦 Próximamente...
Cómo usar Leaflet y MapLibre con capas WMS y MVT.
