# Trabajo Final de Visualización e Inteligencia de Negocio

**Máster Universitario en Análisis de Negocios · Universidad Nebrija**
**Equipo:** [Sergio González Rodríguez] · [Marina Herrera Ramírez] · [Yuridia Navarro Díaz] · [Esteban Giraldo Valencia]

## Pregunta de negocio

> ¿Qué mercados emisores internacionales dejan más gasto por turista en España y cómo varía según la región de destino?

- **Problema:** España recibe un número récord de turistas, pero el volumen no garantiza el valor.
- **Hipótesis:** H1, los mercados que traen más turistas no son los que dejan más gasto medio diario. H2, el peso de cada mercado en el gasto cambia según la comunidad de destino.
- **Periodo analizado:** 2016–2025, por años completos. 2020 y 2021 se tratan como años atípicos por el COVID.

## KPI

| KPI | Fórmula | Qué responde |
|---|---|---|
| % Cuota de gasto | Gasto del mercado / gasto total | ¿Quién aporta más dinero? |
| % Cuota de turistas | Turistas del mercado / turistas totales | ¿Quién aporta más volumen? |
| Brecha de valor (p.p.) | % cuota de gasto − % cuota de turistas | ¿Quién aporta más valor que volumen? |
| % Variación interanual | (Año − año anterior) / año anterior | ¿Qué mercados crecen? |

**Análisis complementarios:** estacionalidad, residentes frente a extranjeros, y turistas frente a excursionistas.

## Metodología

Seguimos el proceso de 8 pasos en 4 fases del Tema 1: **Entender** (problema y objetivo) → **Preparar** (obtener, limpiar y modelar) → **Visualizar** (diseñar y construir) → **Comprobar y entregar** (validar y distribuir). El reparto de tareas por semanas está en [`docs/CRONOGRAMA.md`](docs/CRONOGRAMA.md).

## Estructura del repositorio

```
datos/originales/     los 7 CSV descargados del INE, sin tocar
datos/auxiliares/     dim_geografia.csv · map_origen.csv
docs/                 DICCIONARIO_DATOS.md · CRONOGRAMA.md · memoria · presentación
powerbi/              archivo .pbix
```

## Fuentes y licencia

Todos los datos son del **Instituto Nacional de Estadística (INE)**. Se reutilizan según el [aviso legal del INE](https://www.ine.es/aviso_legal), citando la fuente.

| Operación | Siglas | Tablas |
|---|---|---|
| Encuesta de Gasto Turístico | EGATUR | ① ② |
| Movimientos Turísticos en Fronteras | FRONTUR | ③ |
| Encuesta de Ocupación Hotelera | EOH | ④ ⑤ ⑦ |
| EOH + Encuestas de Ocupación en Alojamientos Turísticos extrahoteleros | — | ⑥ |

En [`docs/DICCIONARIO_DATOS.md`](docs/DICCIONARIO_DATOS.md) está la ficha de cada tabla: periodo, registro mínimo, columnas y limpieza necesaria.

## Tablas auxiliares

- **`dim_geografia.csv`**: las 52 provincias, tal como aparecen en el INE, con su CCAA (código y nombre) y su **Región EGATUR** (las 6 comunidades que detalla EGATUR + "Otras Comunidades Autónomas").
- **`map_origen.csv`**: cada nombre de país o grupo de cada archivo, asignado a un **país estándar**, a su **Mercado EGATUR** (Reino Unido, Alemania, Francia, Italia, Países Nórdicos o Resto del Mundo) y a su tipo (país, agregado o residentes). Sirve para unificar orígenes en Power Query y no sumar agregados.
