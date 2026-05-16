# Proyecto Power BI – Dashboard Diageo España SA

## Contexto

Trabajo final ADE (UAM). **Fecha límite: 19/05/2026** vía Moodle (.pbix en .zip + excels originales y finales).

- **Empresa principal:** Diageo España SA
- **Competidoras:** Pernod Ricard España, Bacardi España, Beam Suntory España
- **Horizonte temporal:** 2019–2024 (ventana común a las 4 empresas; Diageo y Pernod tienen histórico más largo si se quiere ampliar)
- **Historia a contar:** contexto del sector → análisis de Diageo → benchmark frente a competidoras

---

## Fuentes de datos

| Archivo | Contenido real | Uso |
|---|---|---|
| `SABI DIAGEO.xlsx` | KPIs financieros Diageo España 2000–2024, cierre 30 jun, mil EUR | Empresa principal |
| `SABI PERNOD RICARD.xlsx` | KPIs financieros Pernod Ricard 2001–2025, cierre 30 jun, **EUR** (÷1000) | Competidora |
| `SABI BACARDI.xlsx` | KPIs financieros Bacardi España 1999–2024, cierre 31 mar, mil EUR | Competidora |
| `SABI BEAM SUNTORY.xlsx` | KPIs financieros Beam Suntory 2019–2024, cierre 31 dic, mil EUR | Competidora (histórico corto) |
| `EUROSTAT.xlsx` | Estadísticas sector fabricación bebidas España 2011–2020, anual | Contexto macro histórico |
| `INE tabla 76104...xlsx` | IPC bebidas espirituosas/vino/cerveza España, mensual hasta 2026 | Contexto macro reciente |

---

## Modelo en estrella

### Dimensiones

**`dim_empresa`**
| Campo | Valores |
|---|---|
| id_empresa | diageo, pernod, bacardi, beam |
| nombre | Diageo España SA, … |
| ciudad | Madrid, Málaga, Barcelona, Madrid |
| tipo | Empresa principal / Competidora |

**`dim_tiempo`**
| Campo | Valores |
|---|---|
| año | 2019, 2020, 2021, 2022, 2023, 2024 |

> Solo nivel año. Todas las fuentes son anuales. El IPC mensual del INE se agrega a media anual antes de cargar.

### Tabla de hechos empresarial: `fact_kpis`

Granularidad: **1 fila = 1 empresa × 1 año**

| Métrica | Origen SABI | Notas |
|---|---|---|
| `ingresos` | Ingresos de explotación | mil EUR |
| `resultado_antes_impuestos` | Result. ordinarios antes Impuestos | mil EUR |
| `resultado_neto` | Resultado del Ejercicio | mil EUR |
| `activo_total` | Total Activo | mil EUR |
| `fondos_propios` | Fondos propios | mil EUR |
| `empleados` | Número empleados | unidades |
| `roi_pct` | Rentabilidad económica (%) | ya calculado en SABI |
| `roe_pct` | Rentabilidad financiera (%) | ya calculado en SABI |
| `liquidez` | Liquidez general | ya calculado en SABI |
| `endeudamiento_pct` | Endeudamiento (%) | ya calculado en SABI |
| `margen_neto_pct` | resultado_neto / ingresos × 100 | **columna calculada en Power Query** |
| `ingresos_por_empleado` | ingresos / empleados | **columna calculada en Power Query** |

### Tabla de hechos macro: `fact_macro`

Granularidad: **1 fila = 1 indicador × 1 año** (solo España)

| Métrica | Origen | Años |
|---|---|---|
| `num_empresas_sector` | EUROSTAT Sheet 1 — *Enterprises - number* | 2011–2020 |
| `facturacion_sector_meur` | EUROSTAT Sheet 2 — *Turnover - million euro* | 2011–2020 |
| `valor_produccion_meur` | EUROSTAT Sheet 3 — *Production value - million euro* | 2011–2020 |
| `valor_anyadido_meur` | EUROSTAT Sheet 5 — *Value added at factor cost - million euro* | 2011–2020 |
| `ipc_espirituosas` | INE 02.1.1 (media anual) | 2020–2026 |
| `ipc_vino` | INE 02.1.2 (media anual) | 2020–2026 |
| `ipc_cerveza` | INE 02.1.3 (media anual) | 2020–2026 |

> EUROSTAT Sheet 4 — *Gross margin on goods for resale* — descartado: valores inconsistentes (salto de 313 a 1.749 M€ en 2018, probable cambio metodológico) y poco interpretable para el relato del dashboard.

---

## Plan de acción

### Paso 1 — Extracción en Power Query (qué coger de cada fichero)

**SABIs (×4):**
- Hoja: `Page 1`
- Fila 10: cabecera de años (celdas con fechas)
- Filas a extraer: 15 (ingresos), 16 (result. antes imp.), 17 (result. neto), 18 (activo), 19 (fondos propios), 22 (ROI%), 23 (ROE%), 24 (liquidez), 25 (endeudamiento%), 27 (empleados)
- Estrategia: transponer la tabla para que años queden como filas, métricas como columnas
- Añadir columna `empresa` con el nombre fijo en cada query
- Hacer **Append** de las 4 queries → `fact_kpis`

**EUROSTAT:**
- Hojas: Sheet 1, 2, 3, 5 (ignorar Sheet 4 y Summary)
- De cada hoja: solo la fila de **Spain**
- Fila 9: cabecera de años
- Combinar las 4 en una sola tabla con columnas: año, num_empresas, facturacion, valor_produccion, valor_anyadido

**INE:**
- Hoja: `tabla-76104`
- Fila 8: cabecera de meses (formato 2025M01)
- Filas 9, 10, 11: espirituosas, vino, cerveza
- Transponer → agrupar por año (media de los 12 meses) → `ipc_espirituosas`, `ipc_vino`, `ipc_cerveza`

### Paso 2 — Transformaciones recomendadas

| Transformación | Motivo |
|---|---|
| Pernod Ricard: dividir todas las métricas monetarias ÷ 1000 | Viene en EUR, el resto en miles EUR |
| Extraer año de la fecha de cierre fiscal (30/06/2024 → 2024) | Homogeneizar dim_tiempo entre empresas con distinto cierre |
| Filtrar años: quedarse con 2019–2024 en `fact_kpis` | Ventana común a las 4 empresas |
| Calcular `margen_neto_pct` = resultado_neto / ingresos × 100 | No viene calculado en SABI |
| Calcular `ingresos_por_empleado` = ingresos / empleados | KPI de productividad útil para benchmark |
| INE: extraer año de "2025M01" → 2025, calcular media anual por categoría | Agregar mensual a anual para coherencia con el resto |
| EUROSTAT: eliminar columnas vacías intercaladas (el export tiene Nones entre valores) | El formato EUROSTAT exporta con columnas de flags intermedias |
| Tipado correcto: métricas como Decimal, año como entero | Evitar problemas en DAX |
| Crear `dim_empresa` manualmente (tabla introducida) | 4 filas fijas, no necesita query compleja |
| Crear `dim_tiempo` como lista de años 2019–2024 (o más amplia si se usa histórico largo) | Tabla de referencia para el eje temporal |

### Paso 3 — Visualización en Power BI

**Página 1 – Contexto del sector**
- Propósito: situar el mercado antes de hablar de Diageo
- Visuales:
  - Línea: facturación del sector bebidas España (EUROSTAT 2011–2020) → tendencia pre/post crisis
  - Línea: IPC bebidas espirituosas vs IPC vino vs IPC cerveza (INE 2020–2026) → evolución de precios reciente
  - KPI card: nº de empresas en el sector (último dato disponible EUROSTAT)
- Slicer: año (actúa sobre los gráficos de la página)

**Página 2 – Análisis Diageo España**
- Propósito: contar la evolución interna de la empresa principal
- Visuales:
  - Línea o barras: ingresos anuales 2019–2024
  - Línea: margen neto % y ROI% en el mismo eje (escala secundaria o combinado)
  - Barras: número de empleados por año
  - KPI cards: último año — ingresos, resultado neto, ROI%, ROE%
  - Scatter opcional: ROI% vs endeudamiento% por año (muestra la relación rentabilidad/riesgo)
- Slicer: año

**Página 3 – Benchmark competitivo**
- Propósito: posicionar a Diageo frente a sus competidoras
- Visuales:
  - Barras agrupadas: ingresos por empresa por año (Diageo destacado)
  - Barras agrupadas o líneas múltiples: ROI% por empresa — quién es más rentable
  - Barras agrupadas: endeudamiento% por empresa — quién tiene más riesgo financiero
  - Tabla resumen: último año disponible, todas las métricas clave, las 4 empresas
  - Scatter: ingresos vs ROI% por empresa (burbujas, tamaño = empleados) — visión global de posicionamiento
- Slicer: año (permite comparar el mismo año entre empresas)

**Consideraciones de diseño:**
- Diageo siempre en color destacado (primario), competidoras en tonos secundarios
- Mismo slicer de año en todas las páginas si se usa el panel de sincronización de segmentaciones
- Tooltips enriquecidos en los gráficos de benchmark (al pasar el ratón, ver todas las métricas)

---

## Criterios de evaluación a cubrir

| Criterio | Cómo se cubre |
|---|---|
| Adecuación de fuentes | SABI (empresa), EUROSTAT (sector EU), INE (precios España) |
| Transformaciones documentadas | Normalización unidades, extracción año fiscal, cálculo de métricas derivadas |
| Tratamiento de atípicos/ausencias | Beam Suntory sin datos pre-2019 → se excluye o se nota en visual |
| Variables generadas | margen_neto_pct, ingresos_por_empleado |
| Presentación armonizada | JSON de tema corporativo + color Diageo destacado |
| Interacciones | Slicers de año sincronizados entre páginas |
| Coherencia con la "historia" | Macro (sector) → Micro (Diageo) → Relativo (benchmark) |
