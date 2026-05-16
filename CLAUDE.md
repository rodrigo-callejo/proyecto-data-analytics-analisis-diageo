# Proyecto Power BI – Dashboard Diageo España SA

## Contexto

Trabajo final ADE (UAM). **Fecha límite: 19/05/2026** vía Moodle (.pbix en .zip + excels originales y finales).

- **Empresa principal:** Diageo España SA
- **Competidoras:** Pernod Ricard España, Bacardi España, Beam Suntory España
- **Horizonte temporal:** 2019–2024 (ventana común a las 4 empresas)
- **Historia a contar:** contexto del sector → análisis de Diageo → benchmark frente a competidoras

---

## Fuentes de datos

| Archivo original | Archivo tratado | Contenido | Uso |
|---|---|---|---|
| `SABI DIAGEO.xlsx` + 3 competidoras | `Datos-Tratados/SABI.xlsx` | KPIs financieros 4 empresas 2019–2024 | Tabla de hechos empresarial |
| `EUROSTAT.xlsx` | `Datos-Tratados/EUROSTAT.xlsx` | Estadísticas sector fabricación bebidas por país 2011–2020 | Contexto macro sectorial |
| `INE tabla 76104...xlsx` | — (se trata en Power Query) | IPC bebidas espirituosas/vino/cerveza mensual hasta 2026 | Contexto macro precios reciente |
| `Tablas Maestras.xlsx` | — | dim_empresa (4 registros) y dim_tiempo (años) | Dimensiones del modelo |

---

## Modelo tabular

### Tabla SABI — `Datos-Tratados/SABI.xlsx` hoja `Datos finales`

Granularidad: **1 fila = 1 empresa × 1 año**

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

### Tabla EUROSTAT — `Datos-Tratados/EUROSTAT.xlsx` hoja `Datos finales`

Granularidad: **1 fila = 1 país × 1 año**

| Campo | Descripción |
|---|---|
| Región Mercado | España / Europa (para filtrar y comparar) |
| País | Nombre del país |
| Año | 2011–2020 |
| Num Empresas | Unidades enteras |
| Facturación € | Euros (convertido desde M€ ×1.000.000) |
| Valor Producción € | Euros (convertido desde M€ ×1.000.000) |
| Valor Añadido € | Euros (convertido desde M€ ×1.000.000) |

> EUROSTAT Sheet 4 (Gross margin) descartado: salto de 313 a 1.749 M€ en 2018, probable cambio metodológico.

### Tablas maestras

- **Empresas:** id_empresa, nombre, NIF, ciudad, tipo (Principal/Competidora)
- **Años:** lista de años 2001–2026 como dimensión de tiempo

---

## Relaciones del modelo

Las 4 tablas se relacionan a través de dos dimensiones maestras:

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

- **Años → SABI** (1:*): permite filtrar y cruzar los KPIs financieros por año
- **Años → EUROSTAT** (1:*): permite filtrar y cruzar los indicadores macro por año
- **La relación clave** es que ambas tablas de hechos comparten `Años` como dimensión, lo que permite solapar en un mismo gráfico métricas de SABI (empresa española) con métricas de EUROSTAT (sector europeo por país), cruzadas por el año en común
- **Empresas → SABI** (1:*): permite filtrar por empresa o tipo (Principal/Competidora), aunque los atributos de empresa ya vienen en SABI como columnas

---

## Estado actual del proyecto en Power BI

### Completado
- Carga de orígenes tratados vía Excel
- Tipificado de datos (decimales, porcentajes, fechas)
- Renombrado de campos a nombres amigables para el usuario
- Configuración de agregaciones por métrica
- Tablas selectoras de métricas mediante parámetro de campo:
  1. Creado parámetro de campo con las métricas seleccionables
  2. Usada la columna del parámetro (no la métrica directa) en los visuales para preservar el agregador

### Pendiente
- Visualización: 3 páginas del dashboard
- Aplicación del tema corporativo (JSON de estilos)
- Configuración de slicers sincronizados entre páginas

---

## Diseño de visualización

**Página 1 – "Comparativo Nacional"** (`2a42891ea5b30cd4e373`)

Objetivo: comparar Diageo España vs las 3 competidoras (Pernod, Bacardi, Beam Suntory) usando datos SABI 2019–2024.

- Slicer de año (eje temporal 2019–2024)
- KPI cards: ingresos y empleados de Diageo vs competidoras (de un vistazo)
- Gráficos comparativos por empresa para la métrica seleccionada (barras agrupadas o líneas por empresa)
- **Selector de métrica SABI** (parámetro de campo): permite cambiar dinámicamente entre ingresos, resultado neto, ROI%, ROE%, liquidez, endeudamiento, empleados, etc.
- Diageo siempre destacado en `#11295E`; competidoras en tonos secundarios

**Página 2 – "Tendencia Macro"** (`c7f79876d8aaaf51aedb`)

Objetivo: comparar la evolución de Diageo España contra el sector europeo (EUROSTAT) aprovechando el solapamiento de años 2019–2020.

- Slicer de país EUROSTAT: permite elegir comparar contra 1 país, varios, o todos los países europeos
- Gráfico de líneas temporal: métrica SABI de Diageo (eje izquierdo) vs métrica EUROSTAT del sector (eje derecho / escala normalizada), cruzadas por el año común
- **Selector de métrica SABI** (parámetro de campo): ej. "Ingresos" de Diageo
- **Selector de métrica EUROSTAT** (parámetro de campo): ej. "Facturación €" del sector
- El solapamiento 2019–2020 permite contrastar si la tendencia de Diageo se alinea con la media europea del sector

**Diseño:** Diageo en color `#11295E` (primario), comparativas en tonos secundarios. Fondo `fondo_de_pantalla.png`, logo `logo_diageo.png`.

---

## Criterios de evaluación a cubrir

| Criterio | Cómo se cubre |
|---|---|
| Adecuación de fuentes | SABI (empresa), EUROSTAT (sector EU), INE (precios España) |
| Transformaciones documentadas | Normalización unidades, limpieza EUROSTAT, valor anómalo Pernod 2019 |
| Variables generadas | Parámetros de campo como selectores de métrica |
| Presentación armonizada | Tema corporativo `#11295E` + logo Diageo |
| Interacciones | Slicers sincronizados, parámetros de campo dinámicos |
| Coherencia con la "historia" | Macro (sector) → Micro (Diageo) → Relativo (benchmark) |
