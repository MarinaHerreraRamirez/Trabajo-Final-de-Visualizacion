# Limpieza de la tabla ⑤ — Viajeros entrados por provincias, país de residencia y meses (EOH)

**Resultado:** consulta `Fact_Viajeros_Region`, que es la tabla de hechos de viajeros entrados en establecimientos hoteleros agregada a nivel de **Región EGATUR** y mercado emisor (foto fija 2025).
**Archivo:** `Turismo_Internacional_Espana_v02.pbix`

## 1. Antes y después

|  | Antes (CSV del INE) | Después (`Fact_Viajeros_Region`) | 
| ----- | ----- | ----- | 
| **Filas** | 18.928 | **2.016**  | 
| **Registro mínimo** (1 fila =) | Provincia × País de residencia × Mes | **Fecha (mes) × Región EGATUR × País** | 
| **Ámbito geográfico** | 52 provincias (50 provincias + Ceuta y Melilla) | **7 regiones EGATUR** (`Región_EGATUR`, sin totales) | 
| **Orígenes / Mercados** | 28 categorías (25 orígenes + 3 subtotales) | **25 orígenes estandarizados** (`Pais_estandar`) | 
| **Periodo** | Texto del mes sin año (`Enero` – `Diciembre` + `Total`) | **Fecha** (`01/01/2025` – `01/12/2025`) | 
| **Métrica** | Texto con puntos de miles y `..` / `.` | **`Viajeros`** (Número entero sumado por región) | 

2.100 = 7 regiones EGATUR × 25 orígenes × 12 meses (año completo 2025). *(Nota: antes de agrupar por región, la tabla intermedia limpia cuenta con 15.600 filas provinciales = 52 provincias × 25 orígenes × 12 meses; si se filtró también `"Residentes en España"` para trabajar exclusivamente con mercados internacionales como EGATUR, son 24 orígenes = 2.016 filas).*

## 2. Flujo de la consulta

```
CSV del INE (enlace raw de GitHub)           map_origen.csv (GitHub)
      │                                              │
      │                                       [Habilitar carga ❌]
      │                                       [Quitar duplicados]
      └───────────────► Combinar ◄───────────────────┘
                           │
                           ▼
             Mapeo 52 Provincias → 7 Regiones EGATUR
                           │
                           ▼
             Agrupar por (Suma de Viajeros)
                           │
                           ▼
             Fact_Viajeros_Region   ✅ se carga al modelo
             (7 regiones EGATUR × 25 orígenes × 12 meses = 2.100 filas)

```

**¿Por qué se cruza con `map_origen`, se limpian errores como `null` y se agrupa a Región EGATUR?**

1. **Homologación de mercados:** el INE nombra algunos mercados de forma distinta a EGATUR/FRONTUR (por ejemplo, `"Residentes en España"` o `"República China"`). Cruzar con `map_origen.csv` (deshabilitando su carga) traduce todos los orígenes a una clave única (`Pais_estandar`).

2. **Tratamiento del secreto estadístico:** las `1.933` celdas con `..` y `.` representan secreto estadístico del INE, no cero viajeros. Convertirlas a `null` evita errores de conversión numérica y permite sumar sin distorsionar promedios.

3. **Alineación de granularidad con EGATUR:** mientras la EOH publica los viajeros a nivel provincial (52 provincias), las tablas de gasto turístico (EGATUR) trabajan por regiones turísticas (las 6 comunidades autónomas principales más `Resto de CC. AA.`). Mapear y agrupar las provincias en `Región_EGATUR` permite relacionar ambas tablas de hechos (`1 → *`) con `Dim_Region_EGATUR` sin generar relaciones muchos-a-muchos ni valores en blanco.

## 3. Paso a paso y por qué

La columna **Paso en Power BI** recoge el nombre exacto que aparece en *Pasos aplicados* dentro del Editor de Power Query para que cualquiera del equipo pueda auditar la transformación sin abrir cada paso.

### Consulta auxiliar `map_origen` (sin carga al modelo)

| \# | Paso en Power BI | Qué hace | Por qué | 
| ----- | ----- | ----- | ----- | 
| 1 | `Origen` + `Encabezados` | Conecta `map_origen.csv` desde GitHub (`UTF-8`) y promueve encabezados | Centraliza el diccionario de equivalencias de países del proyecto | 
| 2 | `Quitar duplicados` | Elimina duplicados sobre la columna `Nombre_en_archivo` | `map_origen` reúne etiquetas de varios ficheros (EGATUR, EOH...); quitar duplicados garantiza una relación `1:1` y evita multiplicar filas al expandir | 
| 3 | `Deshabilitar carga` | Clic derecho sobre la consulta → desmarcar **Habilitar carga** | Evita que la tabla auxiliar aparezca en la vista de modelo de Power BI | 

### `Fact_Viajeros_Region`: limpieza, mapeo regional y transformación

| \# | Paso en Power BI | Qué hace | Por qué | 
| ----- | ----- | ----- | ----- | 
| 1 | `Origen` | Conexión **Web** al enlace *raw* del CSV en GitHub (`UTF-8`, delimitador `Tabulador`) | Garantiza que el `.pbix` sea reproducible desde los ordenadores de todo el equipo sin depender de rutas locales | 
| 2 | `Columnas como texto` | Paso automático de la importación (o eliminación del `Tipo cambiado` inicial): mantiene todas las columnas como texto | Evita errores de lectura por los puntos de miles y los símbolos `..` / `.` | 
| 3 | `Primera fila como encabezado` | Sube la fila 1 como nombres de columna (`Provincias`, `País de residencia`, `Meses`, `Total`) | El CSV ya viene en formato vertical (largo), por lo que no requiere anular dinamización | 
| 4 | `Quitar Mes Total` | Filtra `Meses <> "Total"` (baja de 18.928 a 17.472 filas) | Elimina el acumulado anual del INE para no duplicar las cifras mensuales | 
| 5 | `Quitar agregados País` | Desmarca `"Total"`, `"No residentes en España"` y `"Unión Europea27 (sin España)"` (baja a `15.600` filas provinciales; `14.976` si se desmarca también `"Residentes en España"`) | Deja únicamente orígenes mutuamente excluyentes (`Residentes en España` + 24 mercados internacionales) | 
| 6 | `Quitar puntos de miles` | Reemplaza `"."` por `""` (vacío) en `Total` **desmarcando** *"Coincidir con el contenido de toda la celda"* | Elimina los separadores de miles de las cifras válidas (`4.909` → `4909`) y deja vacías las celdas de secreto estadístico | 
| 7 | `Viajeros a entero y reemplazar errores` | Convierte `Total` a **Número entero** y aplica **Transformar > Reemplazar errores** con `null` | Transforma los `1.933` registros ocultos por secreto estadístico (`1.866` con `..` y `67` con `.`) en `null` reales con `0%` de error | 
| 8 | `Combinar con map_origen` | **Combinar consultas** (*Externa izquierda*) emparejando `País de residencia` con `map_origen[Nombre_en_archivo]` | Traduce cada mercado del INE al nombre estandarizado del proyecto (**100 % de filas coincidentes**) | 
| 9 | `Expandir Pais_estandar` | Expande únicamente la columna `Pais_estandar` desmarcando *"Usar el nombre de columna original como prefijo"* | Incorpora el nombre normalizado del país directamente como columna limpia | 
| 10 | `Quitar País de residencia` | Elimina la columna original `País de residencia` | Evita redundancia tras haber traído `Pais_estandar` | 
| 11 | `Crear columna Fecha` | Agrega columna personalizada `Date.FromText("01/" & [Meses] & "/2025", "es-ES")` con tipo **Fecha** | Asigna el día 1 de cada mes de 2025 para permitir la relación `1 → *` con `Dim_Calendario[Fecha]` | 
| 12 | `Quitar Meses` | Elimina la columna de texto `Meses` | La información temporal ya queda recogida en `Fecha` | 
| 13 | `Agregar Región_EGATUR` | Crea una **Columna condicional** que mapea las 52 provincias a las 7 regiones EGATUR (`Andalucía`, `Balears, Illes`, `Canarias`, `Cataluña`, `Comunitat Valenciana`, `Madrid, Comunidad de` y `Resto de CC. AA.`) | Homologa la división provincial de la EOH con las regiones de publicación de EGATUR | 
| 14 | `Agrupar por Región, País y Fecha` | **Agrupar por** `Región_EGATUR`, `Pais_estandar` y `Fecha` calculando la **Suma** de `Total` en la nueva columna `Viajeros` | Sube la granularidad de 52 provincias a 7 regiones EGATUR (**2.100 filas finales**) y descarta `Provincias` | 

#### Tabla de equivalencia aplicada en el paso 13 (`Provincias` → `Región_EGATUR`)

| Región EGATUR | Provincias agrupadas | 
| ----- | ----- | 
| **`Andalucía`** | `Almería`, `Cádiz`, `Córdoba`, `Granada`, `Huelva`, `Jaén`, `Málaga`, `Sevilla` | 
| **`Balears, Illes`** | `Balears, Illes` | 
| **`Canarias`** | `Palmas, Las`, `Santa Cruz de Tenerife` | 
| **`Cataluña`** | `Barcelona`, `Girona`, `Lleida`, `Tarragona` | 
| **`Comunitat Valenciana`** | `Alicante/Alacant`, `Castellón/Castelló`, `Valencia/València` | 
| **`Madrid, Comunidad de`** | `Madrid` | 
| **`Resto de CC. AA.`** | Las 33 provincias restantes + `Ceuta` y `Melilla` (cláusula *De lo contrario*) | 

## 4. Comprobaciones hechas

| Comprobación | Valor esperado | ✅ | 
| ----- | ----- | ----- | 
| Dimensiones finales de la tabla | **4 columnas, 2.100 filas** (`Región_EGATUR`, `Pais_estandar`, `Fecha`, `Viajeros`) | ✅ | 
| Cobertura geográfica | **7 regiones EGATUR** sin totales ni nulos (`Andalucía`, `Balears, Illes`, `Canarias`, `Cataluña`, `Comunitat Valenciana`, `Madrid, Comunidad de`, `Resto de CC. AA.`) | ✅ | 
| Cruce con `map_origen.csv` | **100 % de filas coincidentes** (`0` nulos en `Pais_estandar`) | ✅ | 
| Calidad de `Viajeros` | **`0%` de errores** tras limpiar los `1.933` registros de secreto estadístico (`..` y `.`) | ✅ | 
| Ejemplo de control puntual (región uniprovincial) | **Balears, Illes – Alemania – Enero 2025 = `53.309`** (cifra íntegra sin truncar) | ✅ | 
| Ejemplo de control puntual (previo a agrupar) | **Albacete – Residentes en España – Enero 2025 = `19.299`** (sumado dentro de `Resto de CC. AA.`) | ✅ | 
| Rango temporal | **12 meses completos de 2025** (`01/01/2025` – `01/12/2025`) | ✅ | 

## 5. Preguntas que nos puede hacer el profe

**¿Cuál es el registro mínimo de esta tabla?**
Una fila = un mes (`Fecha`, como día 1 de cada mes de 2025) × una región turística (`Región_EGATUR`) × un mercado de origen (`Pais_estandar`), con el total de viajeros entrados en establecimientos hoteleros (`Viajeros`).

**¿Por qué agrupasteis las 52 provincias en 7 regiones EGATUR en lugar de dejar el detalle provincial?**
Porque la granularidad común de todo nuestro modelo estrella es **Región EGATUR**, que es el nivel territorial al que se publican las estadísticas de gasto turístico (EGATUR). Agrupar los viajeros de la EOH a esas mismas 7 regiones evita relaciones muchos-a-muchos en Power BI, permite filtrar todas las tablas de hechos simultáneamente desde `Dim_Region_EGATUR` y hace posible cruzar métricas directamente (como gasto por viajero o estancia media por región).

**¿Qué ocurre con los valores de secreto estadístico (`..` y `.`) al sumar provincias dentro de una misma región?**
En el archivo original del INE hay `1.933` celdas ocultas por secreto estadístico (`1.866` con `..` y `67` con `.`), concentradas sobre todo en provincias pequeñas y meses de temporada baja. Al convertirlas en `null` antes de agrupar, Power Query las ignora en la suma sin generar errores. Metodológicamente conviene tener en cuenta que, en las regiones multiprovinciales (como `Resto de CC. AA.`), la suma de las provincias publicadas puede ser ligeramente inferior al dato agregado por comunidad autónoma que publica el INE.

**¿Por qué utilizasteis "Reemplazar errores" con `null` en lugar de "Reemplazar valores" escribiendo `null`?**
Porque en el cuadro *Reemplazar los valores* de Power Query, escribir `null` introduce la cadena de texto `"null"`, lo que provoca errores de tipo al convertir la columna a número entero. Al eliminar primero los puntos de miles (`"."` → `""`), convertir la columna a **Número entero** y aplicar después **Reemplazar errores** con `null`, Power Query asigna nulos nativos y deja la columna con un `0 %` de errores.

**¿Qué diferencia aporta esta tabla frente a `Fact_Pernoctaciones` y frente a FRONTUR/EGATUR?**
Mientras que `Fact_Pernoctaciones` mide el volumen total de noches consumidas en hoteles, esta tabla mide **personas alojadas (viajeros entrados)**, lo que permite calcular en DAX la **estancia media hotelera** ($\text{Pernoctaciones} / \text{Viajeros}$). Además, al compararla con FRONTUR y EGATUR (que miden el total de turistas internacionales por cualquier vía de alojamiento), permite analizar qué cuota del turismo internacional elige establecimientos hoteleros en cada región.