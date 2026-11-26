# Limpieza de la tabla ③ — Movimientos turísticos en fronteras (FRONTUR)

**Resultado:** consulta `Fact_Visitantes_Frontera`, que es la tabla de hechos de llegada de visitantes internacionales a España (turistas y excursionistas).
**Archivo:** `Turismo_Internacional_Espana_v02.pbix`

---

## 1. Antes y después

| | Antes (CSV del INE) | Después (`Fact_Visitantes_Frontera`) |
|---|---|---|
| **Filas** | 1.584 | **240** |
| **Registro mínimo** (1 fila =) | Mes × Tipo de visitante × Tipo de dato | **Fecha (mes) × Tipo de visitante** |
| **Categorías** | Total, Turista, Excursionista | **Turista** y **Excursionista** (sin Total) |
| **Periodo** | Texto (`2015M10` – `2026M09`) | **Fecha** (`01/01/2016` – `31/12/2025`) |
| **Métrica** | Texto con punto de miles (`14.596.676`) | **`Visitantes`** (Número entero) |

240 = 2 tipos de visitante × 12 meses × 10 años completos (2016–2025).

---

## 2. Flujo de la consulta

```
CSV del INE (enlace raw de GitHub)
      │
      ▼
 Fact_Visitantes_Frontera   ✅ se carga al modelo
 (2 categorías × 120 meses = 240 filas)
```

**¿Por qué una sola consulta y sin pivotar?** A diferencia de la tabla ① de EGATUR (donde cada fila era una métrica distinta con unidades diferentes), aquí todas las filas miden **personas** repartidas en dos categorías mutuamente excluyentes (`Turista` y `Excursionista`). Mantener `Tipo_Visitante` como columna de categoría permite usarla directamente como leyenda en los visuales y sumar ambos grupos sin duplicar personas.

---

## 3. Paso a paso y por qué

La columna **Paso en Power BI** recoge el nombre exacto que aparece en *Pasos aplicados* dentro del Editor de Power Query para que cualquiera del equipo pueda auditar la transformación sin abrir cada paso.

### Fact_Visitantes_Frontera: limpieza y transformación

| # | Paso en Power BI | Qué hace | Por qué |
|---|---|---|---|
| 1 | `Origen` | Conexión **Web** al enlace *raw* del CSV en GitHub (`UTF-8`, tabulación, sin detectar tipos) | Garantiza que el `.pbix` sea reproducible y se actualice desde los ordenadores de los cuatro sin depender de rutas locales |
| 2 | `Columnas como texto` | Paso automático de la importación: deja todas las columnas como texto | Evita que Power BI interprete mal los puntos de miles del INE antes de configurar la región |
| 3 | `Primera fila como encabezado` | Sube la fila 1 como nombres de columna | Los encabezados (`Tipo de visitante`, `Tipo de dato`, `Periodo`, `Total`) venían en la primera fila |
| 4 | `Solo Dato base` | Filtra **Tipo de dato = "Dato base"** (baja a 396 filas) | El CSV mezcla cifras mensuales con tasas de variación y acumulados anuales (que además incluyen `".."` o vacíos `""` en 2015 y 2021). Si no se filtran, se duplican las sumas |
| 5 | `Quitar Tipo de dato` | Elimina la columna `Tipo de dato` | Tras filtrar solo `"Dato base"`, la columna ya no aporta información |
| 6 | `Quitar Total visitantes` | Desmarca `"Total"` en `Tipo de visitante` y deja solo **Turista** y **Excursionista** (baja a 264 filas) | El total de visitantes se obtiene sumando ambas categorías en DAX; dejar `"Total"` duplicaría el volumen |
| 7 | `Reemplazar M por guion` | Sustituye `"M"` por `"-"` en la columna `Periodo` | Convierte textos como `"2025M12"` en `"2025-12"`, formato reconocible como fecha |
| 8 | `Periodo a Fecha` | Convierte `Periodo` al tipo **Fecha** | Transforma `"2025-12"` en el primer día de cada mes (`01/12/2025`) |
| 9 | `Renombrar a Fecha` | Renombra `Periodo` a `Fecha` | Estandariza el nombre de la clave para relacionarla `1 → *` con `Dim_Calendario[Fecha]` |
| 10 | `Filtrar 2016-2025` | Filtra `Fecha` entre `01/01/2016` y `31/12/2025` (baja a **240 filas**) | El periodo de estudio del proyecto son años completos de 2016 a 2025; dejar 3 meses de 2015 o 9 de 2026 distorsionaría los totales anuales |
| 11 | `Visitantes a entero (es-ES)` | Convierte `Total` a **Número entero** con configuración regional **Español (España)** | El INE usa punto para los miles (`14.596.676`). Sin configuración española se truncarían cifras y al tratarse de personas no tienen sentido los decimales |
| 12 | `Renombrar columnas` | Renombra `Total` → `Visitantes` y `Tipo de visitante` → `Tipo_Visitante` | Nombres limpios, autoexplicativos y coherentes con el resto del modelo |

---

## 4. Comprobaciones hechas

| Comprobación | Valor esperado | ✅ |
|---|---|---|
| Dimensiones finales de la tabla | **3 columnas, 240 filas** (`Fecha`, `Tipo_Visitante`, `Visitantes`) | ✅ |
| Turistas totales en 2025 (suma de los 12 meses) | **96.803.893** (~96,8 millones; cuadra con EGATUR) | ✅ |
| Excursionistas totales en 2025 (suma de los 12 meses) | **47.233.807** (~47,2 millones) | ✅ |
| Total visitantes internacionales en 2025 | **144.037.700** (~144,0 millones, sin truncar puntos) | ✅ |
| Cierre de fronteras COVID (abril y mayo de 2020) | **`0`** en ambas categorías (sin `null` ni errores) | ✅ |

---

## 5. Preguntas que nos puede hacer el profe

**¿Cuál es el registro mínimo de esta tabla?**
Una fila = un mes (`Fecha`, como día 1 de cada mes) × un tipo de visitante internacional (`Turista` o `Excursionista`), con la métrica de volumen (`Visitantes`).

**¿Qué diferencia hay entre turista y excursionista en FRONTUR?**
Según la metodología del INE, ambos son visitantes internacionales no residentes en España: el **turista** pernocta al menos una noche en el país, mientras que el **excursionista** entra y sale en el mismo día sin pernoctar (por ejemplo, tráfico transfronterizo terrestre o escalas de cruceros de un día).

**¿Por qué aquí no pivotasteis `Tipo_Visitante` como hicisteis con las métricas en EGATUR?**
Porque en EGATUR las filas eran métricas distintas con unidades incompatibles (millones de euros, euros/día, noches). En FRONTUR todas las filas miden **personas** divididas en dos grupos mutuamente excluyentes. Mantener `Tipo_Visitante` en una sola columna facilita filtrar o desglosar por color/leyenda en los gráficos sin tener que crear medidas separadas para cada categoría.

**¿Por qué filtrasteis `2016–2025` si el CSV del INE traía datos hasta septiembre de 2026?**
Para mantener la consistencia temporal con el resto del proyecto (años completos 2016–2025) y evitar que los gráficos anuales muestren una caída artificial en 2026 al faltar el último trimestre del año.

**¿Cómo se valida esta tabla frente a la tabla ① de EGATUR?**
Sumando los 12 meses de `Tipo_Visitante = "Turista"` en 2025 obtenemos **96,80 millones**, exactamente la misma cifra que reconstruimos en `Fact_Gasto_CCAA` al dividir el gasto total de EGATUR entre el gasto medio por persona. Esto demuestra que ambas operaciones del INE son coherentes entre sí.

**¿Por qué convertisteis `"2025M12"` en fecha (`01/12/2025`) en lugar de dejar año y mes como texto?**
Para poder crear una relación limpia de uno a muchos (`1 → *`) con nuestra tabla `Dim_Calendario[Fecha]` y usar jerarquías temporales y funciones de inteligencia de tiempo en DAX.

**¿Qué papel juega esta tabla en el informe y qué limitaciones tiene?**
Aporta el **contexto macro** de llegadas y permite responder al análisis complementario de **turistas frente a excursionistas** (mostrando el peso de los visitantes de un día, que no generan pernoctación reglada y tienen un impacto económico distinto). Su principal limitación es que esta serie agregada no desglosa por país emisor ni por comunidad autónoma de destino.