# Limpieza de la tabla ④ — Pernoctaciones por provincias, país de residencia y meses (EOH)

**Resultado:** consulta `Fact_Pernoctaciones_Provincia`, que es la tabla de hechos de pernoctaciones hoteleras desglosadas por provincia española y mercado emisor (foto fija 2025).
**Archivo:** `Turismo_Internacional_Espana_v02.pbix`

---

## 1. Antes y después

| | Antes (CSV del INE) | Después (`Fact_Pernoctaciones_Provincia`) |
|---|---|---|
| **Filas** | 18.928 | **15.600** |
| **Registro mínimo** (1 fila =) | Provincia × País de residencia × Mes | **Fecha (mes) × Provincia × País** |
| **Ámbito geográfico** | 52 provincias (50 provincias + Ceuta y Melilla) | **52 provincias** (`Provincia`, sin totales) |
| **Orígenes / Mercados** | 28 categorías (25 orígenes + 3 subtotales) | **25 orígenes estandarizados** (`Pais_estandar`) |
| **Periodo** | Texto del mes sin año (`Enero` – `Diciembre` + `Total`) | **Fecha** (`01/01/2025` – `01/12/2025`) |
| **Métrica** | Texto con puntos de miles y `..` / `.` | **`Pernoctaciones`** (Número entero con `null`) |

15.600 = 52 provincias (50 + Ceuta y Melilla) × 25 orígenes × 12 meses (año completo 2025).

---

## 2. Flujo de la consulta

```
CSV del INE (enlace raw de GitHub)           map_origen.csv (GitHub)
      │                                              │
      │                                       [Habilitar carga ❌]
      │                                       [Quitar duplicados]
      └───────────────► Combinar ◄───────────────────┘
                           │
                           ▼
             Fact_Pernoctaciones_Provincia   ✅ se carga al modelo
             (52 provincias × 25 orígenes × 12 meses = 15.600 filas)
```

**¿Por qué se cruza con `map_origen` y se mantienen los `null`?**
1. El INE nombra algunos mercados de forma distinta a EGATUR/FRONTUR (por ejemplo, `"Residentes en España"` o `"República China"`). Cruzar con la tabla auxiliar `map_origen.csv` (deshabilitando su carga al modelo) traduce todos los orígenes a una clave única (`Pais_estandar`).
2. Las celdas con `..` y `.` representan **secreto estadístico** del INE, no cero pernoctaciones. Convertirlas a `null` evita errores de tipo y permite que DAX calcule sumas y promedios sin distorsionarlos con ceros ficticios.

---

## 3. Paso a paso y por qué

La columna **Paso en Power BI** recoge el nombre exacto que aparece en *Pasos aplicados* dentro del Editor de Power Query para que cualquiera del equipo pueda auditar la transformación sin abrir cada paso.

### Consulta auxiliar `map_origen` (sin carga al modelo)

| # | Paso en Power BI | Qué hace | Por qué |
|---|---|---|---|
| 1 | `Origen` + `Encabezados` | Conecta `map_origen.csv` desde GitHub (`UTF-8`) y promueve encabezados | Centraliza el diccionario de equivalencias de países del proyecto |
| 2 | `Quitar duplicados` | Elimina duplicados sobre la columna `Nombre_en_archivo` | `map_origen` reúne etiquetas de varios ficheros (EGATUR, EOH...); quitar duplicados garantiza una relación `1:1` y evita multiplicar las 15.600 filas al expandir |
| 3 | `Deshabilitar carga` | Clic derecho sobre la consulta → desmarcar **Habilitar carga** | Evita que la tabla auxiliar aparezca en la vista de modelo de Power BI |

### `Fact_Pernoctaciones_Provincia`: limpieza y transformación

| # | Paso en Power BI | Qué hace | Por qué |
|---|---|---|---|
| 1 | `Origen` | Conexión **Web** al enlace *raw* del CSV en GitHub (`UTF-8`, delimitador `Tabulador`) | Garantiza que el `.pbix` sea reproducible desde los ordenadores de los cuatro sin depender de rutas locales |
| 2 | `Columnas como texto` | Paso automático de la importación: mantiene todas las columnas como texto | Evita errores de lectura por los puntos de miles y los símbolos `..` / `.` |
| 3 | `Primera fila como encabezado` | Sube la fila 1 como nombres de columna (`Provincias`, `País de residencia`, `Meses`, `Total`) | El CSV ya viene en formato vertical (largo), por lo que no requiere anular dinamización |
| 4 | `Quitar Mes Total` | Filtra `Meses <> "Total"` (baja de 18.928 a 17.472 filas) | Elimina el acumulado anual del INE para no duplicar las cifras mensuales |
| 5 | `Quitar agregados País` | Desmarca `"Total"`, `"No residentes en España"` y `"Unión Europea27 (sin España)"` (baja a **15.600 filas**) | Deja únicamente los 25 orígenes mutuamente excluyentes (`Residentes en España` + 24 mercados internacionales) |
| 6 | `Reemplazar .. por null` | Sustituye `".."` por `null` en `Total` marcando **Coincidir con el contenido de toda la celda** | Limpia los datos protegidos por secreto estadístico sin borrar los puntos de miles de las cifras válidas |
| 7 | `Reemplazar . por null` | Sustituye `"."` por `null` en `Total` marcando **Coincidir con el contenido de toda la celda** | Completa la limpieza de celdas sin dato publicado (`67` filas con punto simple) |
| 8 | `Pernoctaciones a entero (es-ES)` | Convierte `Total` a **Número entero** con configuración regional **Español (España)** | Interpreta correctamente el punto de miles (`868.963`) y deja los nulos limpios sin errores |
| 9 | `Combinar con map_origen` | **Combinar consultas** (*Externa izquierda*) emparejando `País de residencia` con `map_origen[Nombre_en_archivo]` | Traduce cada mercado del INE al nombre estandarizado del proyecto (**15.600 de 15.600 filas coincidentes**) |
| 10 | `Expandir Pais_estandar` | Expande únicamente la columna `Pais_estandar` desmarcando *"Usar el nombre de columna original como prefijo"* | Incorpora el nombre normalizado del país directamente como columna limpia |
| 11 | `Quitar País de residencia` | Elimina la columna original `País de residencia` | Evita redundancia tras haber traído `Pais_estandar` |
| 12 | `Crear columna Fecha` | Agrega columna personalizada `Date.FromText("01/" & [Meses] & "/2025", "es-ES")` con tipo **Fecha** | El archivo solo trae el nombre del mes sin año; asignarle el día 1 de cada mes de 2025 permite relacionarlo `1 → *` con `Dim_Calendario[Fecha]` |
| 13 | `Quitar Meses` | Elimina la columna de texto `Meses` | La información temporal ya queda recogida en `Fecha` |
| 14 | `Renombrar columnas` | Renombra `Provincias` → `Provincia` y `Total` → `Pernoctaciones` | Nombres limpios y alineados con `Dim_Geografia` y el resto del modelo |

---

## 4. Comprobaciones hechas

| Comprobación | Valor esperado | ✅ |
|---|---|---|
| Dimensiones finales de la tabla | **4 columnas, 15.600 filas** (`Provincia`, `Pais_estandar`, `Pernoctaciones`, `Fecha`) | ✅ |
| Cobertura provincial | **52 provincias** (50 provincias + Ceuta y Melilla, sin CCAA ni Total Nacional) | ✅ |
| Cruce con `map_origen.csv` | **15.600 de 15.600 filas coincidentes** (`0` nulos en `Pais_estandar`) | ✅ |
| Calidad de `Pernoctaciones` | **`0%` de errores** y **`1.925` valores `null`** (`12,34%` por secreto estadístico del INE) | ✅ |
| Ejemplo de control puntual (punto de miles) | **Albacete – Residentes en España – Enero 2025 = `27.908`** (cifra íntegra sin truncar) | ✅ |
| Rango temporal | **12 meses completos de 2025** (`01/01/2025` – `01/12/2025`) | ✅ |

---

## 5. Preguntas que nos puede hacer el profe

**¿Cuál es el registro mínimo de esta tabla?**
Una fila = un mes (`Fecha`, como día 1 de cada mes de 2025) × una provincia española (`Provincia`) × un mercado de origen (`Pais_estandar`), con su volumen de noches de hotel (`Pernoctaciones`).

**¿Por qué esta tabla solo cubre 2025 mientras que FRONTUR o EGATUR abarcan 2016–2025?**
Porque cumple el rol de **foto fija provincial de 2025** dentro del modelo: permite bajar al máximo detalle geográfico (52 provincias) y cruzar turismo nacional (`Residentes en España`) frente a mercados internacionales en el último año completo disponible.

**¿Por qué no filtrasteis filas en la columna `Provincias`?**
Porque, a diferencia de otras tablas del INE, este fichero de la EOH ya viene descargado directamente a nivel provincial (las 50 provincias más Ceuta y Melilla) sin incluir filas agregadas de `"Total Nacional"` ni de comunidades autónomas.

**¿Qué significan los puntos (`..` y `.`) en el archivo original y por qué los dejasteis como `null` en lugar de cambiarlos a `0`?**
En la metodología del INE indican **secreto estadístico** (cuando en una provincia hay pocos establecimientos abiertos y publicar el dato permitiría identificar empresas concretas) o dato no disponible. Cambiarlos a `0` falsearía los promedios y mínimos en Power BI; dejarlos como `null` permite que DAX los ignore automáticamente al agregar.

**¿Por qué fue necesario quitar duplicados en `map_origen` antes de combinar?**
Porque `map_origen.csv` es una tabla maestra compartida que contiene filas para distintos ficheros del proyecto. Si un mismo país aparece más de una vez en `Nombre_en_archivo`, el cruce *Externa izquierda* multiplica las filas de la tabla de hechos. Al eliminar duplicados previamente aseguramos una correspondencia exacta de `15.600` a `15.600`.

**¿Cómo construisteis la columna `Fecha` si el CSV solo traía el nombre del mes (`"Enero"`, `"Febrero"`...)?**
Concatenando el día 1 y el año 2025 mediante `Date.FromText("01/" & [Meses] & "/2025", "es-ES")`. De este modo transformamos un texto suelto en una fecha real capaz de relacionarse de uno a muchos (`1 → *`) con `Dim_Calendario[Fecha]`.

**¿Qué aporta esta tabla al análisis frente a EGATUR y FRONTUR?**
Mientras que FRONTUR y EGATUR miden llegadas y gasto internacional a nivel nacional o por comunidad autónoma, esta tabla de la EOH aporta **capilaridad provincial** y permite comparar el peso del **turismo residente en España** frente al extranjero en los hoteles de cada provincia.