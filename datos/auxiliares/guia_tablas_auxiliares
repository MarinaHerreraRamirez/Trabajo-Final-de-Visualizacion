# Tablas auxiliares

Estas dos tablas **no vienen del INE**. Las hemos creado nosotros para que los datos de distintas encuestas puedan **cruzarse con la misma granularidad**: CCAA y Mercado EGATUR. Se usan en **Power Query**: sirven para unificar nombres y agrupar categorías en las consultas. Las relaciones entre tablas se crean después, en la vista de modelo.

Las dos usan **punto y coma (`;`) como separador** y codificación UTF-8.

---

## `dim_geografia.csv` — de provincia a CCAA y a Región EGATUR

| Columna | Qué es | Ejemplo |
|---|---|---|
| `Provincia` | Nombre de la provincia, escrito igual que en el INE | `Coruña, A` |
| `Cod_CCAA` | Código INE de la comunidad autónoma | `12` |
| `CCAA` | Nombre de la CCAA, igual que en las tablas ① y ⑥ | `12 Galicia` |
| `CCAA_corto` | Nombre corto para los visuales | `Galicia` |
| `Region_EGATUR` | Grupo que usa EGATUR: las 6 CCAA que detalla o "Otras Comunidades Autónomas" | `Otras Comunidades Autónomas` |

**Cuándo usarla:**
- **Tablas ④ y ⑤ (EOH por provincia):** se combina por `Provincia` para pasar cada provincia a su CCAA y agrupar a nivel CCAA.
- **Tabla ⑥ (por CCAA):** se combina por `CCAA` para añadir la `Region_EGATUR`.
- **Para cruzar ocupación con gasto (①):** se agrupa la ocupación por `Region_EGATUR`. EGATUR solo detalla 6 comunidades, así que es el único nivel común entre las dos.
- **Como dimensión Geografía del modelo**, con la jerarquía Región EGATUR → CCAA → Provincia.

---

## `map_origen.csv` — de país a Mercado EGATUR

| Columna | Qué es | Ejemplo |
|---|---|---|
| `Archivo` | En qué tabla aparece ese nombre | `EOH_paises (⑦)` |
| `Nombre_en_archivo` | El nombre exacto tal como viene en ese CSV | `Estados Unidos de América` |
| `Pais_estandar` | Nombre unificado para todas las tablas | `Estados Unidos` |
| `Mercado_EGATUR` | Reino Unido, Alemania, Francia, Italia, Países Nórdicos o Resto del Mundo | `Resto del Mundo` |
| `Tipo` | `Mercado`, `País o grupo`, `Residentes` o `Agregado` | `País o grupo` |

**Cuándo usarla:**
- **Tablas ④, ⑤ y ⑦ (EOH):** se combina por `Archivo` + `Nombre_en_archivo` para asignar a cada país su Mercado EGATUR y poder compararlo con el gasto.
- **Para filtrar los agregados:** las filas con `Tipo = Agregado` (Total, UE27, Residentes en el Extranjero…) **no se suman** con los países, porque se contarían dos veces.
- **Residentes en España:** tienen su propio tipo porque no existen en EGATUR, que solo mide turismo extranjero. Se excluyen al comparar con el gasto.
- **Como dimensión Origen del modelo**, con la jerarquía Mercado EGATUR → País.

> ⚠️ La columna `Archivo` es necesaria porque un mismo nombre no siempre significa lo mismo. Por ejemplo, "Resto del Mundo" en EGATUR incluye todo lo que no son sus 5 mercados, y en la EOH es solo un grupo residual de países.

---

## Resumen: qué tabla auxiliar usar con cada CSV

| CSV | `dim_geografia` | `map_origen` |
|---|---|---|
| ① Gasto por CCAA y país | Ya viene en Región EGATUR | Ya viene en Mercado EGATUR |
| ② Gasto por país | — | Ya viene en Mercado EGATUR |
| ③ FRONTUR | — | — |
| ④⑤ EOH por provincia | ✅ Provincia → CCAA → Región | ✅ País → Mercado |
| ⑥ EOH por CCAA | ✅ CCAA → Región | — |
| ⑦ EOH por país | — | ✅ País → Mercado |
