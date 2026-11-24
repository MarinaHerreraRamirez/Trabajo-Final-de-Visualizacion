# Limpieza de la tabla ① — Gasto por CCAA y país de residencia (EGATUR)

**Resultado:** consulta `Fact_Gasto_CCAA`, que es la tabla de hechos del gasto turístico.
**Archivo:** `Turismo_Internacional_Espana_v01.pbix`

---

## 1. Antes y después

| | Antes (CSV del INE) | Después (Fact_Gasto_CCAA) |
|---|---|---|
| **Filas** | 3.840 | **420** |
| **Registro mínimo** (1 fila =) | Año × Región × País × Tipo de dato × **una métrica** | Año × Región EGATUR × Mercado, **con las métricas en columnas** |
| **Mercados** | 5 + Total | 5 + **Resto del Mundo** (sin Total) |
| **Regiones** | 6 CCAA + Otras + Total | 6 CCAA + Otras (sin Total) |
| **Turistas y días** | No vienen | **Calculados** |

420 = 6 mercados × 7 regiones × 10 años (2016–2025).

---

## 2. Las tres consultas

```
CSV del INE (enlace de GitHub)
      │
      ▼
 Gasto_Base ──────────────┐   (limpieza común; no se carga)
      │                    │
      ▼                    ▼
 Gasto_Resto          Fact_Gasto_CCAA
 (calcula el Resto;    = 5 mercados (sin Total)
  no se carga)           + Resto del Mundo (anexado)  ✅ se carga al modelo
```

**¿Por qué tres consultas?** Para **no mezclar totales con detalle**. Las consultas intermedias tienen la carga deshabilitada, así que el modelo solo ve la tabla final, en la que todo se puede sumar sin duplicar.

---

## 3. Paso a paso y por qué

La columna **Paso en Power BI** es el nombre exacto que aparece en *Pasos aplicados*. El primer paso de cada consulta se llama siempre **Origen**: Power BI no deja renombrarlo, así que su explicación va en la descripción del paso (clic derecho → *Propiedades*).

### Gasto_Base: limpieza común

| # | Paso en Power BI | Qué hace | Por qué |
|---|---|---|---|
| 1 | Origen | Conexión **Web** al enlace "raw" del CSV en GitHub (UTF-8, tabulación, sin detectar tipos) | La misma ruta funciona en el ordenador de los cuatro y el .pbix se actualiza en cualquier sitio. Sin detección automática, Power BI no puede leer mal los números |
| 2 | Columnas como texto | Paso automático de la importación: deja todas las columnas como texto | Los tipos los decidimos nosotros más adelante |
| 3 | Primera fila como encabezado | Usa la fila 1 como nombres de columna | Los nombres venían en la primera fila |
| 4 | Solo Dato base | Filtra **Tipo de dato = "Dato base"** | El CSV mezcla cifras con **tasas de variación**. Si no se quitan, se suman y se duplican |
| 5 | Quitar Tipo de dato | Elimina esa columna | Después del filtro ya no aporta nada |
| 6 | Quitar Total CCAA | Quita la fila "Total" de las comunidades | El total se obtiene sumando las 7 regiones. Si se deja, se cuenta dos veces |
| 7 | Renombrar a Region_EGATUR | Renombra "Total Nacional y CCAA" | Coincide con la columna de `dim_geografia`, con la que se relacionará |
| 8 | Total a número (es-ES) | `Total` → Número decimal con **configuración regional Español (España)** | El INE escribe `134.743,32`. Con configuración inglesa se leería **134,74** y se perderían tres cifras sin ningún aviso |
| 9 | Periodo a entero | `Periodo` → Número entero | El gasto por CCAA es **anual** |
| 10 | Renombrar a Año | Renombra "Periodo" | Nombre más claro |
| 11 | Métricas a columnas | **Columna dinámica** sobre la columna de métricas (valores: Total, sin agregar) | Pasa de 1 métrica por fila a **1 columna por métrica**. Es el formato que necesita una tabla de hechos |
| 12 | Calcular Turistas | `[Gasto total] * 1000000 / [Gasto medio por persona]` | El archivo **no trae el número de turistas**. Se multiplica por un millón porque el gasto total viene en **millones de €** |
| 13 | Turistas a decimal | Tipo Número decimal | Las columnas personalizadas salen sin tipo |
| 14 | Calcular Días | `[Turistas] * [Duración media de los viajes]` | Hace falta para recalcular bien el **gasto medio diario** en DAX |
| 15 | Días a decimal | Tipo Número decimal | Las columnas personalizadas salen sin tipo |

### Gasto_Resto: cálculo del Resto del Mundo

| # | Paso en Power BI | Qué hace | Por qué |
|---|---|---|---|
| 1 | Origen | Referencia a `Gasto_Base` | Parte del resultado ya limpio y se actualiza sola |
| 2 | Signo Total y mercados | Columna condicional: **+1 si es Total, −1 si es un mercado** | Es el truco: Resto = Total − los 5 mercados |
| 3 | Signo a entero | Tipo Número entero | — |
| 4 | Gasto con signo | `G = [Gasto total] * [Signo]` | Así, al sumar, sale directamente el Resto |
| 5 | Turistas con signo | `T = [Turistas] * [Signo]` | Igual |
| 6 | Días con signo | `D = [Días] * [Signo]` | Igual |
| 7 | Columnas con signo a decimal | G, T y D → Número decimal | — |
| 8 | Sumar por región y año | **Agrupar por** Región + Año, sumando G, T y D | Da una fila de Resto por región y año: 70 filas |
| 9 | Renombrar a Gasto total | Corrige "Gasto Total" → "Gasto total" | Power Query distingue mayúsculas y minúsculas. Sin esto, al anexar saldrían dos columnas distintas |
| 10 | País Resto del Mundo | Columna `País de residencia` = "Resto del Mundo" | Identifica las filas nuevas |
| 11 | País a texto | Tipo Texto | — |
| 12 | Calcular gasto medio por persona | `[Gasto total] * 1000000 / [Turistas]` | Las medias del Resto no vienen en el INE; se deducen de los totales |
| 13 | Gasto medio por persona a decimal | Tipo Número decimal | — |
| 14 | Calcular gasto medio diario | `[Gasto total] * 1000000 / [Días]` | Igual |
| 15 | Gasto medio diario a decimal | Tipo Número decimal | — |
| 16 | Calcular duración media | `[Días] / [Turistas]` | Igual |
| 17 | Duración media a decimal | Tipo Número decimal | — |

### Fact_Gasto_CCAA: tabla final

| # | Paso en Power BI | Qué hace | Por qué |
|---|---|---|---|
| 1 | Origen | Referencia a `Gasto_Base` | — |
| 2 | Quitar Total de países | Quita el país "Total" (quedan 350 filas) | El total ya estará representado por la suma de los 6 mercados |
| 3 | Añadir Resto del Mundo | **Anexar consultas** con `Gasto_Resto` | 350 + 70 = **420 filas**: ahora los 6 mercados **suman el total** |
| 4 | Redondear Turistas | 0 decimales | No existen 0,2 turistas. Se redondea **al final**, después de calcular el Resto, para que los errores de redondeo no se arrastren a la resta |
| 5 | Redondear Días | 0 decimales | Igual |
| 6 | Turistas y Días a entero | Tipo Número entero | — |
| 7 | Redondear gasto medio por persona | 0 decimales | Para que las filas del Resto tengan **el mismo formato que publica el INE** para los demás mercados |
| 8 | Redondear gasto medio diario | 0 decimales | Igual |
| 9 | Medias a entero | Tipo Número entero | — |
| 10 | Redondear duración media | 2 decimales | Igual que el INE |

**Además:** `Gasto_Base` y `Gasto_Resto` tienen la **carga deshabilitada** (aparecen en cursiva), así que el modelo solo ve `Fact_Gasto_CCAA`.

---

## 4. Comprobaciones hechas

| Comprobación | Valor esperado | ✅ |
|---|---|---|
| Configuración regional (Andalucía, Total, 2025) | 20.032,82 M€, no 20,03 | ✅ |
| Turistas (Alemania, Andalucía, 2016) | 1.121,33 M€ / 1.132 € = **990.574** | ✅ |
| Coherencia con el gasto diario | 1.121,33 M€ / 11,03 M días = 101,6 € (el INE da **102**) | ✅ |
| Resto del Mundo (Andalucía, 2016) | 11.318,92 − 5 mercados = **4.871,99** | ✅ |
| Medias del Resto (Andalucía, 2016) | 1.232 €/persona · 132 €/día · 9,3 días | ✅ |
| Filas finales | **420** | ✅ |
| Turistas totales en 2025 | ≈ **96,8 millones** | ✅ |

---

## 5. Preguntas que nos puede hacer el profe

**¿Cuál es el registro mínimo de esta tabla?**
Una fila = un mercado emisor × una región EGATUR × un año, con 4 métricas del INE (gasto total, gasto medio por persona, gasto medio diario y duración media) y 2 calculadas (turistas y días).

**¿Por qué la granularidad es "región EGATUR" y no CCAA?**
Porque EGATUR solo detalla 6 comunidades y agrupa el resto en "Otras". Es el nivel más fino que tiene esta fuente. Para cruzar con la EOH, que sí tiene todas las CCAA, usamos `dim_geografia`, que asigna cada CCAA a su región EGATUR.

**¿Por qué no usasteis los viajeros de la EOH para calcular el gasto por turista?**
Porque miden cosas distintas. La EOH solo cuenta a quien se aloja en establecimientos reglados, como hoteles, e incluye a los residentes en España. EGATUR mide a todos los turistas extranjeros, se alojen donde se alojen. Mezclarlos daría un ratio sin sentido.

**¿Por qué pivotasteis?**
Porque en el CSV cada métrica era una fila distinta. Para una tabla de hechos interesa que cada fila sea un hecho (mercado, región y año) con sus medidas en columnas. Así se pueden sumar y combinar en DAX.

**¿Por qué no sumáis el gasto medio o la duración media?**
Porque **las medias no se suman**. El gasto medio de toda España no es la suma de los de cada región. Por eso calculamos turistas y días: en DAX, el gasto medio se recalcula como `SUM(gasto) / SUM(turistas)`, que sí es correcto a cualquier nivel.

**¿Por qué calculasteis el Resto del Mundo?**
Para que los mercados sumen el total. Sin el Resto, la cuota de Alemania se calcularía sobre la suma de solo 5 países y saldría inflada.

**¿Por qué redondeasteis en Power Query y no con el formato del modelo?**
Porque redondear en Power Query **cambia el dato**, y en turistas y días los decimales no tienen sentido real. El formato del modelo solo cambia cómo se ve. Lo usaremos para importes y porcentajes, donde sí interesa conservar la precisión.

**¿Por qué cambiasteis el nombre de los pasos?**
Para que el proceso se entienda sin abrir cada paso. Cualquiera que abra el .pbix puede leer en *Pasos aplicados* qué se hizo y en qué orden. Es documentación dentro del propio archivo.

**¿Qué diferencia hay entre Power Query y el modelo?**
Power Query **limpia y transforma** cada tabla. Las **relaciones** entre tablas se crean después, en la vista de modelo (el modelo semántico). En Power Query no relacionamos nada.

**¿Qué pasa si el INE publica datos de 2026?**
Basta con sustituir el CSV en GitHub por el nuevo, manteniendo el mismo nombre, y pulsar **Actualizar**. Todos los pasos se repiten solos, porque son reproducibles.

**¿Qué riesgos o limitaciones tiene esta tabla?**
Es **anual**, así que no permite ver la estacionalidad (para eso están las tablas mensuales ② y ⑦). Solo hay 5 mercados detallados. Y 2020 y 2021 son años atípicos por el COVID.
