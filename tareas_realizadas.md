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

## 4. Modelo tabular final

Dos tablas de hechos con cometidos distintos:

**Tabla SABI** — `Datos-Tratados/SABI.xlsx` hoja `Datos finales`, granularidad empresa × año:

| Campo | Descripción |
|---|---|
| ID Empresa, Nombre, Ciudad, Tipo | Atributos de empresa |
| Año, Cierre Fiscal, Fecha Cierre | Dimensión temporal |
| Ingresos | Miles de EUR |
| Resultados ant impuestos | Miles de EUR |
| Resultado Neto | Miles de EUR (Pernod 2019 en blanco — valor anómalo) |
| Activo Total, Fondos Propios | Miles de EUR |
| ROI pct, ROE pct | Ya calculados en SABI |
| Liquidez, Endeudamiento Porcentual | Ya calculados en SABI |
| Empleados | Unidades |

**Tabla EUROSTAT** — `Datos-Tratados/EUROSTAT.xlsx` hoja `Datos finales`, granularidad país × año:

| Campo | Descripción |
|---|---|
| Región Mercado | España / Europa |
| País | Nombre del país |
| Año | 2011–2020 |
| Num Empresas | Unidades enteras |
| Facturación € | Euros (convertido desde M€ ×1.000.000) |
| Valor Producción € | Euros (convertido desde M€ ×1.000.000) |
| Valor Añadido € | Euros (convertido desde M€ ×1.000.000) |

**Tablas maestras:** `dim_empresa` (id, nombre, NIF, ciudad, tipo) y `dim_tiempo` (año)

**Relaciones del modelo:**

```
┌─────────────┐        ┌───────────────┐
│    Años     │        │   Empresas    │
│  (dim)      │        │   (dim)       │
└──────┬──────┘        └──────┬────────┘
       │ 1                    │ 1
       │                      │
       │ *                    │ *
┌──────┴──────┐        ┌──────┴────────┐
│    SABI     │        │   EUROSTAT    │
│  (hechos)   │        │   (hechos)    │
└─────────────┘        └───────────────┘
       │                      │
       └──────────────────────┘
              Año (común)
```

- **Años → SABI** (1:*): filtra KPIs financieros por año
- **Años → EUROSTAT** (1:*): filtra indicadores macro por año
- **Empresas → SABI** (1:*): filtra por empresa o tipo (Principal/Competidora)
- El año común (2019–2020) permite solapar métricas SABI y EUROSTAT en el mismo gráfico

## 5. Fondo de pantalla Power BI
- Diseñado manualmente en Paint: fondo gris claro (`#EBEEF3`), barra azul `#11295E` en la parte superior, zona blanca bajo la barra
- Dimensiones: 1536×863 px
- Guardado en `PowerBI/Recursos de PowerBI/fondo_de_pantalla.png`

## 6. Logo Diageo
- Logo original (`logo_diageo.png`) en color rojo/carmín sobre fondo blanco
- Recoloreado a azul `#11295E` manteniendo fondo blanco
- Guardado en `PowerBI/Recursos de PowerBI/logo_diageo.png`

## 7. Tablas Maestras
- Creado `Datos/Tablas Maestras.xlsx` con dos hojas:
  - **Años:** 2001–2026
  - **Empresas:** rellenada con los 4 registros (id_empresa, nombre, NIF, ciudad, tipo) extraídos de los SABIs

## 8. Transformación EUROSTAT — `Datos-Tratados/EUROSTAT.xlsx`

El objetivo fue consolidar en una única hoja limpia (`Datos finales`) los indicadores del sector de fabricación de bebidas en España, partiendo de las hojas originales del fichero exportado de EUROSTAT. El proceso se realizó íntegramente en Excel:

- **Consolidación:** las 4 hojas útiles (Nº empresas, Facturación, Valor producción, Valor añadido) unificadas en una sola hoja con columna identificadora de métrica. Hoja de Margen Bruto descartada.
- **Limpieza:** eliminadas columnas de flags intercaladas, filas de agregados geográficos (totales UE, grupos regionales) y ceros sustituidos por blancos.
- **Transposición:** de años como columnas a estructura fila por país × año.
- **Unificación de unidades:** M€ → € (×1.000.000) para homogeneizar con nº de empresas.
- **Tabla final:** columnas `Región Mercado`, `País`, `Año`, `Num Empresas`, `Facturación €`, `Valor Producción €`, `Margen Bruto €`, `Valor Añadido €`.

## 9. Transformación SABIs — `Datos-Tratados/SABI.xlsx`

El objetivo fue consolidar en una única hoja limpia (`Datos finales`) los indicadores financieros de las 4 empresas, partiendo de los 4 ficheros SABI individuales:

- **Consolidación:** cada empresa en su hoja de origen dentro del mismo libro.
- **Limpieza:** eliminadas columnas vacías intercaladas, filas de metadatos, y años fuera de la ventana 2019–2024.
- **Transposición:** de años como columnas a estructura fila por empresa × año.
- **Unificación de unidades:** Pernod Ricard (EUR) ÷ 1.000 → todo en miles de EUR. Valores anómalos de Pernod Ricard en 2019 (resultado neto y antes de impuestos) dejados en blanco.
- **Tabla final:** columnas `ID Empresa`, `Nombre`, `Ciudad`, `Tipo`, `Año`, `Cierre Fiscal`, `Fecha Cierre`, `Ingresos`, `Resultados ant impuestos`, `Resultado Neto`, `Activo Total`, `Fondos Propios`, `ROI pct`, `ROE pct`, `Liquidez`, `Endeudamiento Porcentual`, `Empleados`.

## 11. Diseño de las páginas del dashboard

**Página 1 – "Comparativo Nacional"**

Objetivo: comparar Diageo España vs las 3 competidoras (Pernod, Bacardi, Beam Suntory) con datos SABI 2019–2024.

- Slicer de año (2019–2024)
- KPI cards: ingresos y empleados de Diageo vs competidoras de un vistazo
- Gráficos comparativos por empresa para la métrica seleccionada (barras agrupadas o líneas)
- Selector de métrica SABI (parámetro de campo): cambia dinámicamente entre ingresos, resultado neto, ROI%, ROE%, liquidez, endeudamiento, empleados, etc.
- Diageo destacado en `#11295E`; competidoras en tonos secundarios

**Página 2 – "Tendencia Macro"**

Objetivo: comparar la evolución de Diageo España contra el sector europeo (EUROSTAT) en el solapamiento 2019–2020.

- Slicer de país EUROSTAT: compara contra 1 país, varios o todos
- Gráfico de líneas temporal: métrica SABI de Diageo vs métrica EUROSTAT del sector, cruzadas por año común
- Selector de métrica SABI (ej. "Ingresos") y selector de métrica EUROSTAT (ej. "Facturación €")
- El solapamiento 2019–2020 permite contrastar si la tendencia de Diageo se alinea con la media europea

**Diseño común:** fondo `fondo_de_pantalla.png`, logo `logo_diageo.png`, color corporativo `#11295E`.

## 10. Carga y configuración en Power BI

- **Inclusión de orígenes:** los ficheros tratados (`SABI.xlsx` y `EUROSTAT.xlsx`) cargados en Power BI vía Excel, junto con las Tablas Maestras e INE.
- **Tipificado de datos:** métricas numéricas con sus decimales, porcentajes configurados como tanto por ciento, fechas tipadas correctamente.
- **Renombrado de campos:** nombres amigables para el usuario final en todos los campos de ambas tablas.
- **Configuración de agregaciones:** definida la agregación correcta para cada métrica (suma, media, ninguna según corresponda).
- **Tablas selectoras de métricas (parámetro de campo):** para permitir al usuario seleccionar dinámicamente qué métrica visualizar en los gráficos:
  1. Se creó un **parámetro de campo** nuevo en Power BI con las métricas seleccionables
  2. Se usó la columna generada por el parámetro (en lugar de la métrica directa) en los visuales para preservar el agregador configurado y que el slicer funcione correctamente
