# Tareas realizadas

## 1. Lectura y análisis del enunciado
- Extraído el contenido del PDF `Trabajo Final DashBoard Power BI_2S.pdf`
- Identificadas las tres capas del dashboard: entorno macro → análisis interno → benchmark competitivo
- Fecha de entrega: 19/05/2026 vía Moodle (.pbix en .zip + excels originales y finales)

## 2. Inspección de los ficheros de datos disponibles
- Revisados los 5 ficheros originales en `/Datos`: SABI Pernod Ricard, SABI Bacardi, SABI Beam Suntory, INE 76104, EUROSTAT
- Identificado que la empresa principal es **Diageo España SA** (los SABIs originales eran de otras empresas)
- Solicitado y añadido el fichero `SABI_Export_1.xlsx` (renombrado a `SABI DIAGEO.xlsx`) con datos de Diageo

## 3. Diagnóstico de cada fuente de datos

### SABIs (×4)
- Estructura: fila 10 = cabecera de años, filas 15–27 = métricas
- Métricas disponibles: ingresos, resultado antes impuestos, resultado neto, activo total, fondos propios, ROI%, ROE%, liquidez general, endeudamiento%, empleados
- **Problema de unidades:** Pernod Ricard viene en EUR; Diageo, Bacardi y Beam Suntory en miles de EUR → normalizar dividiendo Pernod ÷ 1000
- Cierres fiscales distintos: Diageo y Pernod (30 jun), Bacardi (31 mar), Beam Suntory (31 dic)
- Ventana temporal común: 2019–2024 (Beam Suntory solo tiene datos desde 2019)
- Diageo tiene histórico desde 2000; Pernod desde 2001; Bacardi desde 1999

### EUROSTAT
- 5 sheets: Sheet 1 (nº empresas), Sheet 2 (facturación), Sheet 3 (valor producción), Sheet 4 (margen bruto — **descartado**), Sheet 5 (valor añadido)
- Sector: "Manufacture of beverages" (NACE C11), datos anuales **2011–2020**
- Solo se usa la fila de **Spain** en cada sheet
- Sheet 4 descartado por valores inconsistentes (salto inexplicado en 2018)
- **Desfase temporal:** solo solapan 2019–2020 con los datos de empresas → se usa como contexto histórico del sector, no como comparación año a año

### INE tabla 76104
- Contenido: IPC de bebidas (espirituosas, vino, cerveza, otras), frecuencia mensual hasta abril 2026
- Se agregará a media anual para coherencia con el resto de fuentes
- Útil para contexto macro reciente (precios), complementa el EUROSTAT que llega solo a 2020

## 4. Diseño del modelo en estrella
- Definida `fact_kpis`: granularidad empresa × año, métricas SABI + derivadas (margen_neto_pct, ingresos_por_empleado)
- Definida `fact_macro`: datos EUROSTAT (2011–2020) + IPC INE agregado anual
- `dim_empresa`: id, nombre, NIF, ciudad, tipo (Principal/Competidora)
- `dim_tiempo`: solo nivel año (todas las fuentes son anuales)
- Documentado en `CLAUDE.md`

## 5. Plan de acción documentado en CLAUDE.md
- Paso 1: qué extraer de cada fichero en Power Query (filas y columnas concretas)
- Paso 2: transformaciones recomendadas (normalización unidades, extracción año fiscal, cálculo métricas derivadas, limpieza EUROSTAT)
- Paso 3: diseño de visualización por página (contexto sector, análisis Diageo, benchmark competitivo)

## 6. Fondo de pantalla Power BI
- Diseñado manualmente en Paint: fondo gris claro (`#EBEEF3`), barra azul `#11295E` en la parte superior, zona blanca bajo la barra
- Dimensiones: 1536×863 px
- Guardado en `PowerBI/Recursos de PowerBI/fondo_de_pantalla.png`

## 7. Logo Diageo
- Logo original (`logo_diageo.png`) en color rojo/carmín sobre fondo blanco
- Recoloreado a azul `#11295E` manteniendo fondo blanco
- Guardado en `PowerBI/dashboard-assets/logo_diageo.png`

## 8. Power Query — intento de escritura directa en TMDL
- Se intentó escribir las queries M directamente en los ficheros TMDL del .pbip
- Problema: el formato TMDL distribuye el código en `expressions.tmdl` (queries/funciones) y ficheros individuales en `tables/` (tablas físicas con `partition = m`); el `model.tmdl` solo acepta metadatos
- Se descartó el enfoque: Power BI revierte los cambios al guardar desde el IDE y el formato es frágil de mantener manualmente
- **Decisión:** consolidar y limpiar los datos directamente en Excel antes de cargarlos en Power BI, y hacer solo los pasos finales de formateo y tipado en Power Query

## 9. Tablas Maestras
- Creado `Datos/Tablas Maestras.xlsx` con dos hojas:
  - **Años:** 2001–2026
  - **Empresas:** rellenada con los 4 registros (id_empresa, nombre, NIF, ciudad, tipo) extraídos de los SABIs

## 10. Transformación EUROSTAT — `Datos-Tratados/EUROSTAT.xlsx`

El objetivo fue consolidar en una única hoja limpia (`Datos finales`) los indicadores del sector de fabricación de bebidas en España, partiendo de las hojas originales del fichero exportado de EUROSTAT. El proceso se realizó íntegramente en Excel mediante funciones de manipulación, copia, pegado especial y limpieza manual:

**Paso 1 — Consolidación de hojas**
Las cuatro hojas con información aprovechable (Nº de empresas, Facturación, Valor de producción y Valor añadido) se copiaron y unificaron en una única hoja de trabajo (`Datos combinados`), añadiendo una columna identificadora de la métrica correspondiente. La hoja de Margen Bruto se descartó por presentar valores inconsistentes (salto de 313 a 1.749 M€ en 2018, atribuible a un cambio metodológico), al no ser interpretable para el análisis.

**Paso 2 — Eliminación de datos indeseados**
Se eliminaron manualmente las columnas vacías o de flags que el formato de exportación de EUROSTAT intercala entre los valores, así como las filas correspondientes a agregados geográficos (totales UE, grupos regionales) que no aportaban información útil al análisis. Los ceros se sustituyeron por celdas en blanco para no distorsionar medias ni gráficos.

**Paso 3 — Transposición: de columnas de año a filas**
La tabla consolidada presentaba los años como columnas (2011–2020). Para facilitar su uso en Power BI, se reorganizó la información de forma que cada fila corresponde a una combinación única de métrica, país y año, obteniendo así la hoja `Datos transpuestos`.

**Paso 4 — Unificación de unidades**
Las métricas de facturación, valor de producción y valor añadido venían expresadas en millones de euros en el fichero original. Para homogeneizar con el indicador de número de empresas (en unidades enteras) y facilitar los cálculos en Power BI, los valores se multiplicaron por 1.000.000, dejando todas las magnitudes monetarias expresadas en euros.

**Paso 5 — Tabla final (`Datos finales`)**
A partir de `Datos transpuestos`, se construyó la tabla definitiva pivotando las métricas a columnas, de forma que cada fila corresponde a un país y un año, con las columnas: `Región Mercado`, `País`, `Año`, `Num Empresas`, `Facturación €`, `Valor Producción €`, `Margen Bruto €`, `Valor Añadido €`.

---

## 11. Transformación SABIs — `Datos-Tratados/SABI.xlsx`

El objetivo fue consolidar en una única hoja limpia (`Datos finales`) los indicadores financieros de las cuatro empresas del análisis (Diageo, Pernod Ricard, Bacardi y Beam Suntory), partiendo de los cuatro ficheros exportados de SABI. El proceso se realizó en Excel mediante funciones de copia, pegado especial, limpieza y fórmulas:

**Paso 1 — Consolidación de hojas**
Los datos de cada empresa se encontraban en ficheros Excel independientes, cada uno con una hoja `Page 1`. Se creó un único libro de trabajo (`SABI.xlsx`) recogiendo los datos de cada empresa en su propia hoja de origen (`Origen - Diageo España SA`, `Origen - Bacardi España SA`, etc.), identificando en cada una la fila de cabecera con las fechas de cierre fiscal (fila 10) y las filas de cada indicador financiero (filas 15–27).

**Paso 2 — Eliminación de datos indeseados**
Cada fichero SABI contiene columnas vacías intercaladas entre años, filas de metadatos (tipo de cuenta, estado de aprobación, formato contable) y el histórico completo de cada empresa desde sus primeros registros. Se eliminaron todas las columnas y filas que no correspondían a los años 2019–2024, que es la ventana temporal común a las cuatro empresas, y todas las filas auxiliares que no contenían indicadores financieros.

**Paso 3 — Transposición: de columnas de año a filas**
Al igual que en EUROSTAT, los años aparecían como columnas en los ficheros originales. Se reorganizó la información para que cada fila corresponda a una empresa y un año, con los indicadores como columnas.

**Paso 4 — Unificación de unidades**
Pernod Ricard exporta sus cifras en euros, mientras que el resto de empresas las presentan en miles de euros. Para homogeneizar, los valores monetarios de Pernod Ricard se dividieron entre 1.000, dejando todas las magnitudes en miles de euros. Adicionalmente, se identificó que los datos de Pernod Ricard para el ejercicio 2019 presentaban valores anómalos en las partidas de resultado neto y resultado antes de impuestos (cifras de varios miles de millones de euros incompatibles con el volumen de la empresa), por lo que dichas celdas se dejaron en blanco.

**Paso 5 — Tabla final (`Datos finales`)**
Se construyó la tabla definitiva combinando las cuatro hojas de origen en una única hoja, con las columnas: `ID Empresa`, `Nombre`, `Ciudad`, `Tipo`, `Año`, `Cierre Fiscal`, `Fecha Cierre`, `Ingresos`, `Resultados ant impuestos`, `Resultado Neto`, `Activo Total`, `Fondos Propios`, `ROI pct`, `ROE pct`, `Liquidez`, `Endeudamiento Porcentual`, `Empleados`.
