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

## 8. Tablas Maestras
- Creado `Datos/Tablas Maestras.xlsx` con dos hojas:
  - **Años:** 2001–2026
  - **Empresas:** rellenada con los 4 registros (id_empresa, nombre, NIF, ciudad, tipo) extraídos de los SABIs
