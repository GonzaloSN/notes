# 🗺️ Tema 1: Desarrollo Web orientado a GIS

Este documento es una introducción práctica para programadores que deseen incorporar visualización geográfica en la web usando tecnologías **open source**. No necesitas ser geógrafo: con saber de programación y entender los conceptos claves de SIG (GIS), puedes crear soluciones muy potentes.

---

## 🧰 Tecnologías Web GIS

### 🌿 Leaflet
**¿Qué es?** Una librería JavaScript open source para mapas interactivos.

**Ventajas:**
- Liviana (~40KB).
- Fácil de usar.
- Amplia comunidad y plugins.

**Ideal para:** proyectos donde necesitas flexibilidad y velocidad.

---

### 🧭 MapLibre GL JS
**¿Qué es?** Una librería open source para mapas vectoriales renderizados en el navegador (WebGL).

**Ventajas:**
- Soporte para **vector tiles**.
- Estilos personalizables estilo Mapbox.
- Fluidez en zoom y rotación.

**Ideal para:** mapas modernos con alto rendimiento y estilo.

---

## 📐 Sistemas de Coordenadas para Programadores

Cuando trabajas con mapas, los datos geográficos deben estar "referenciados". Aquí van los conceptos básicos:

### 🗺️ WGS84 y EPSG:4326
- **WGS84** es el sistema global que usa **latitud** y **longitud**.
- En códigos EPSG se conoce como **EPSG:4326**.
- Es el más usado en APIs web y GPS.

### 📏 Coordenadas proyectadas
- Representan el mundo plano (x, y), ideal para cálculos de distancia o áreas.
- Un ejemplo común en Chile: **EPSG:32719** (UTM zona 19S).
- Necesitarás transformarlas si vas a mostrar datos en mapas web.

### 📚 ¿Qué debe saber un programador?
- Siempre identifica en qué sistema están tus datos geográficos.
- Usa bibliotecas como **proj**, **pyproj**, **GDAL** o **PostGIS** para transformar coordenadas.
- Leaflet y MapLibre esperan **WGS84 (EPSG:4326 o EPSG:3857)** por defecto.

---

## 🔌 Tipos de servicios GIS

Los servidores de mapas ofrecen distintos servicios que los clientes web pueden consumir:

| Tipo | Sigla | ¿Qué entrega? | Usado en |
|------|-------|----------------|----------|
| WMS  | Web Map Service | Imágenes de mapas (raster) | Leaflet, QGIS |
| WFS  | Web Feature Service | Datos vectoriales (GeoJSON, GML) | apps que consultan geometrías |
| WMTS | Web Map Tile Service | Mosaicos de imágenes en tiles | rendimiento mejorado |
| MVT  | Mapbox Vector Tiles | Tiles vectoriales renderizados en cliente | MapLibre, Mapbox |

---

## 🧱 Servidores GIS open source

### 🌐 GeoServer
- Permite publicar capas WMS, WFS, WMTS.
- Panel web intuitivo.
- Soporta estilos SLD.
- **Incluye GWC** (GeoWebCache) para cachear tiles WMTS.

### 🚀 pg_tileserv
- Servidor muy liviano para publicar capas **MVT** directamente desde **PostGIS**.
- Ideal cuando trabajas con grandes volúmenes de datos vectoriales.

---

## 💡 Mi experiencia con `pg_tileserv` y datos grandes

Cuando trabajé con un shapefile muy grande, me enfrenté a tiempos de carga inaceptables al intentar publicarlo vía GeoServer como WFS. La solución fue levantar un servidor con `pg_tileserv` que sirviera **vector tiles directamente desde la base de datos PostGIS**.

### 🐳 Cómo lo levanté en Docker Compose

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
