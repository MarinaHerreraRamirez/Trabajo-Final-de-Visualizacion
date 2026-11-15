# Diccionario de datos — ficha tabla por tabla

Revisión de los 7 CSV del repositorio (versión del 4 de noviembre de 2026). Todos vienen del **INE**, separados por tabuladores, en UTF-8 con BOM y con **formato numérico español** (punto para los miles y coma para los decimales).

> **Periodo analizado propuesto: 2016–2025, por años completos.** Es el rango del gasto por CCAA (①), que es la tabla con menos cobertura. 2020 y 2021 se marcan como años atípicos por el COVID.

---

## Resumen

| # | Archivo | Fuente | Periodo | Registro mínimo (1 fila =) | Filas | Papel |
|---|---|---|---|---|---|---|
| ① | Gasto … por comunidad autónoma … y país de residencia | EGATUR | **Anual** 2016–2025 | Año × Región EGATUR × Mercado × Métrica | 3.840 | Núcleo |
| ② | Gasto … según país de residencia | EGATUR | Mensual 2015M10–2026M09 | Mes × Mercado × Métrica | 14.784 | Validación + estacionalidad del gasto |
| ③ | Movimientos Turísticos en Fronteras | FRONTUR | Mensual 2015M10–2026M09 | Mes × Tipo de visitante | 1.584 | Contexto |
| ④ | Pernoctaciones … por provincias, país de residencia y meses | EOH (hoteles) | **Solo 2025** | Mes × Provincia × País | 18.928 | Foto fija 2025 |
| ⑤ | Viajeros … por provincias, país de residencia y meses | EOH (hoteles) | **Solo 2025** | Mes × Provincia × País | 18.928 | Foto fija 2025 |
| ⑥ | Viajeros y pernoctaciones por comunidades autónomas | EOH + 4 encuestas extrahoteleras | Mensual 2014M01–2026M09 | Mes × Tipo de alojamiento × CCAA × Residencia × Medida | 91.800 | Núcleo (ocupación por CCAA) |
| ⑦ | Viajeros y pernoctaciones según país de residencia | EOH (hoteles) | Mensual 1999M01–2026M09 | Mes × País × Medida | 21.312 | Núcleo (ocupación por mercado) |

---

## ① Gasto por CCAA y país de residencia (EGATUR)

- **Columnas:** País de residencia · Total Nacional y CCAA · Tipo de dato · Gastos y duración media de los viajes · Periodo · Total
- **Periodo:** anual, de 2016 a 2025. No hay meses.
- **Mercados (5 + Total):** Reino Unido, Países Nórdicos, Alemania, Francia, Italia. **No hay fila "Resto"**.
- **Geografía (6 + Otras + Total):** Andalucía, Illes Balears, Canarias, Cataluña, C. Valenciana, Madrid y "Otras Comunidades Autónomas".
- **Métricas:** Gasto total (**millones de €**), Gasto medio por persona (€), Gasto medio diario por persona (€) y Duración media de los viajes (días).
- **Limpieza:**
  - Filtrar `Tipo de dato = "Dato base"` y quitar la tasa de variación anual. Si no se quita, se suma dos veces.
  - Pivotar la columna de métricas para tener una columna por métrica.
  - Las filas "Total" sirven para validar, no para sumar.
  - **No tiene número de turistas.** Se reconstruye así: `Turistas = Gasto total × 1.000.000 / Gasto medio por persona`. Comprobación: en 2025, 134.743,32 M€ / 1.392 € ≈ 96,8 M de turistas. `Días = Turistas × Duración media`.
  - El "Resto" de mercados solo se puede calcular para el gasto total (Total − suma de los 5). Las medias no se pueden restar.

## ② Gasto por país de residencia (EGATUR)

- **Columnas:** País de residencia · Tipo de dato · Gastos y duración media de los viajes · Periodo · Total
- **Periodo:** mensual, de 2015M10 a 2026M09.
- **Mercados:** los mismos 5 + **Resto del Mundo** + Total.
- **Tipo de dato (4):** Dato base · Tasa de variación anual · Acumulado en lo que va de año · Tasa de variación acumulada. **Hay que quedarse solo con "Dato base".**
- **Calidad:** 112 celdas vacías y 56 con "..".
- Permite estudiar la **estacionalidad del gasto** a nivel nacional.

## ③ Movimientos Turísticos en Fronteras (FRONTUR)

- **Columnas:** Tipo de visitante · Tipo de dato · Periodo · Total
- **Periodo:** mensual, de 2015M10 a 2026M09.
- **Tipo de visitante:** Total · Turista · Excursionista.
- ⚠️ **No incluye vía de acceso, país ni CCAA.** Hay que quitar "vía de acceso" de la presentación.
- **Tipo de dato (4):** filtrar "Dato base".
- **Calidad:** 18 celdas vacías y 6 con "..".

## ④⑤ Pernoctaciones y viajeros por provincia, país y mes (EOH)

- **Columnas:** Provincias · País de residencia · Meses · Total
- **Periodo:** ⚠️ **no hay columna de año**. Es **2025**: la suma de viajeros (118.327.077) cuadra exactamente con el total de 2025 de la tabla ⑦.
- **Meses:** Enero a Diciembre, más "Total" (quitarlo).
- **52 provincias.** Se pasan a CCAA con `dim_geografia.csv`.
- **Origen (28 categorías):** incluye **agregados** mezclados con países (Total, Residentes en España, No residentes en España, Unión Europea27). Se distinguen con `map_origen.csv`.
- **Secreto estadístico:** ~1.930 celdas con "..". Hay que convertirlas en **nulo**, no en 0. Por eso la suma de países (58,2 M) es menor que "No residentes" (63,7 M). **Para los totales se usan las filas agregadas, no la suma de países.**
- **Uso propuesto:** foto fija de 2025 (mapa y ranking). No sirve para comparaciones interanuales.

## ⑥ Viajeros y pernoctaciones por CCAA

- **Columnas:** Operación · Total Nacional · Comunidades y Ciudades Autónomas · Viajeros y pernoctaciones · Residencia: Nivel 1 · Residencia: Nivel 2 · Periodo · Total
- **Periodo:** mensual, de 2014M01 a 2026M09.
- **Operación (5):** Encuesta de Ocupación Hotelera · Campings · Apartamentos Turísticos · Alojamientos de Turismo Rural · Albergues. ⚠️ No es solo la EOH.
- **CCAA (19):** con código INE (01 Andalucía … 19 Melilla). Si está vacía, es el Total Nacional.
- **Residencia:** si Nivel 2 está vacío, es el total. Si no: Residentes en España / Residentes en el Extranjero. **No hay desglose por país.**
- **Calidad:** 5.520 celdas vacías y 5.902 con "..".
- Es el archivo más grande (11 MB): conviene filtrar pronto en Power Query.

## ⑦ Viajeros y pernoctaciones según país de residencia (EOH)

- **Columnas:** RESIDENCIA/ORIGEN · Países · Viajeros y pernoctaciones · Periodo · Total
- **Periodo:** mensual, de 1999M01 a 2026M09. Solo nivel nacional.
- ⚠️ **El país está repartido en dos columnas.** Si `Países` está vacío, el origen está en `RESIDENCIA/ORIGEN` (América sin EEUU, Estados Unidos de América, Japón, Asia sin Japón). Se combinan en una sola columna.
- **Agregados que no se suman:** Total, Residentes en el Extranjero, UE28 sin España y UE27_2020 sin España. Ya se ha comprobado que el resto de categorías no se solapan.
- Rusia aparece sin datos en los periodos recientes.

---

## Problemas comunes a todas las tablas

1. **Configuración regional:** al cambiar el tipo a número, usar *Cambiar tipo → Usar configuración regional → Español (España)*.
2. **Periodo:** convertir "2025M07" a fecha (1 de julio de 2025). En ① el periodo es solo el año.
3. **Filas "Total" y agregados:** filtrarlas o marcarlas para que no se cuenten dos veces.
4. **"Tipo de dato":** en ①②③ quedarse solo con "Dato base".
5. **"..":** convertir en nulo (secreto estadístico o dato no disponible).

## Granularidad común propuesta

| Eje | Nivel común | Cómo |
|---|---|---|
| Geografía | **CCAA**, con el atributo **Región EGATUR** (6 + Otras) | `dim_geografia.csv` |
| Origen | **Mercado EGATUR** (5 + Resto), con el país como detalle | `map_origen.csv` |
| Tiempo | **Año** (el gasto es anual). El mes solo para estacionalidad | Tabla calendario en DAX |
