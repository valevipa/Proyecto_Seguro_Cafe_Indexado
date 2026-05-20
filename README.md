# Seguro Agrícola Indexado de Café — Quindío y Nariño

**Módulo 2 · Modelamiento Climatico-Agrícola · 2024**

---

## Problema

Los seguros agrícolas tradicionales requieren ajustadores de campo para verificar pérdidas individuales, lo que los hace prohibitivamente costosos para pequeños caficultores colombianos. Este proyecto diseña y valida un **seguro indexado** basado en el índice climático SPI-3 (Standardized Precipitation Index, escala 3 meses) que activa pagos automáticamente cuando la precipitación supera umbrales predefinidos por departamento, eliminando el riesgo moral y los costos de ajuste.

El modelo cubre los departamentos de **Quindío y Nariño** (Colombia) con datos históricos 2000–2024 y opera en dos tracks complementarios:

| Track | Pregunta central |
|-------|-----------------|
| **A — Índice SPI-3** | ¿Qué umbrales de precipitación deben activar el seguro? |
| **B — Rendimiento** | ¿El índice climático predice pérdidas reales de café (kg/ha)? |

---

## Instalación

### Requisitos previos

- Python ≥ 3.9
- Jupyter Notebook o JupyterLab
- Git

### Pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/valevipa/Proyecto_Seguro_Cafe_Indexado.git
cd Proyecto_Seguro_Cafe_Indexado

# 2. Crear entorno virtual (recomendado)
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
# .venv\Scripts\activate         # Windows

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Abrir el notebook
jupyter notebook seguro_cafe_completo_vf.ipynb
```

> **Nota:** La primera celda ejecuta `!pip install -q yfinance` automáticamente si el paquete no está disponible.

---

## Datos

Todas las fuentes son **públicas y sin costo**. Los archivos se descargan automáticamente desde el repositorio GitHub vía la variable `BASE`:

```
BASE = 'https://raw.githubusercontent.com/valevipa/Proyecto_Seguro_Cafe_Indexado/refs/heads/main/data/raw/'
```

| Fuente | Archivo(s) | Cobertura | Variable(s) |
|--------|-----------|-----------|-------------|
| **ERA5** (ECMWF) | `era5_precip_quindio_narino_consolidado.csv`, `era5_precip_quindio_narino_2018_2024.csv` | 2000–2024 · mensual | Precipitación mm/mes |
| **EVA** (MADR) | `eva_cafe_quindio_narino_actualizado.csv` | 2007–2024 · anual · municipal | Rendimiento kg/ha, área cosechada ha |
| **IDEAM** | `df_tmedia_aire.xlsx`, `df_tmax_aire.xlsx`, `df_tmin_aire.xlsx`, `df_hr.xlsx`, `df_anomalia_temp.xlsx` | Variable | Temperatura, humedad relativa |
| **MODIS** (NASA) | `df_modis_anual.csv`, `df_modis_mensual.csv` | 2012–2019 | NDVI, EVI por ventana fenológica |
| **NOAA ONI** | `noaa_oni_index.csv` | 2000–2024 | Índice El Niño/La Niña |
| **FNC** (precios) | `Precios-area-y-produccion-de-cafe-2026-3.xlsx` | 2005–2024 | Precio interno COP/carga 125 kg |
| **FNC** (área) | `detalle_agricola_departamental_cafe.csv` | 2007–2024 | Área cosechada/sembrada ha |

**Panel final:** 18 años × 2 departamentos = **36 observaciones departamentales**; ~800+ obs a nivel municipal.

---

## Ejecución

El notebook está diseñado para ejecutarse **de principio a fin** con `Kernel → Restart & Run All`:

```
Sección 0  · Configuración e imports
Sección 1  · ETL — carga y limpieza de 9 fuentes de datos
Sección 2  · Track A — cálculo SPI-3, clasificación de eventos, validación N1–N4
Sección 3  · Track B — panel integrado, modelos LOYO, SHAP, evaluación D1–D4
Sección 4  · Tabla resumen de requerimientos
Sección 5  · Hedging Effectiveness (HE)
Sección 6  · Requerimientos funcionales F2, F5, F6
Sección 7  · Hallazgos y próximos pasos
Exportación · Outputs CSV para dashboard
```

### Outputs generados

Al finalizar la ejecución, la carpeta `outputs/` contiene:

| Archivo | Contenido |
|---------|-----------|
| `kpis_resumen.csv` | KPIs principales por departamento (R², RMSE, HE) |
| `activaciones_historicas.csv` | Serie SPI-3 mensual con clasificación de eventos |
| `umbrales_departamento.csv` | Umbrales P12/P88 calibrados y frecuencias |
| `pred_vs_real.csv` | Predicciones LOYO vs rendimiento real por municipio |
| `shap_importancia.csv` | Importancia SHAP de variables por departamento |
| `validacion_historica_n1.csv` | Detección de años 2012 (roya) y 2015 (El Niño) |
| `d1_holdout_departamental.csv` | RMSE/MAE hold-out 2018–2019 nivel departamental |
| `tabla_requerimientos_dashboard.csv` | Estado de todos los requerimientos N1–N4, D1–D4, F1–F6 |
| `metadata_proyecto.csv` | Metadatos de versión y fecha de exportación |

Los gráficos (`.png`) se guardan en el directorio de trabajo.

---

## Resultados

### Track A — Índice SPI-3

| Requerimiento | Criterio | Resultado | Estado |
|--------------|---------|-----------|--------|
| **N1** Validación histórica | Detectar 2012 (roya) y 2015 (El Niño) | Ambos años detectados en ambos departamentos | OK |
| **N2** Poder predictivo | R² ≥ 0.70 in-sample | Quindío: 0.921 · Nariño: 0.618 | Parcial |
| **N3** Frecuencia activación | 15%–25% de meses | Dentro del rango actuarial objetivo | OK |
| **N4** Calibración umbrales | Percentiles empíricos P12/P88 | Ajustados por departamento con prueba Mann-Whitney | OK |

**Umbrales calibrados:**
- Nariño: sequía SPI-3 ≤ -0.827 · exceso SPI-3 ≥ +P88
- Quindío: sequía SPI-3 ≤ -1.137 · exceso SPI-3 ≥ +1.221

### Track B — Modelo de Rendimiento

| Requerimiento | Criterio | Resultado | Estado |
|--------------|---------|-----------|--------|
| **D1** RMSE hold-out | ≤ 149 kg/ha (nivel departamental) | Nariño: ~149 · Quindío: ~160 kg/ha | Límite |
| **D2** Coherencia SHAP | SPI-3 desarrollo en top-3 variables | Confirmado en ambos departamentos | OK |
| **D3** Estabilidad temporal | ΔR² < 0.15 · std < 0.10 | ΔR² > 1.0 por ruptura estructural roya 2012–2014 | No cumple |
| **D4** RBIM vs estadístico | ΔR² tolerable | Nariño ΔR²=0.16 · Quindío ΔR²=0.59 | Parcial |

**Nota D3:** La inestabilidad proviene del choque endógeno de la roya (2012–2014), no del modelo climático. El SPI-3 captura ~25% de la varianza del rendimiento; el resto corresponde a precio FNC y fitosanidad.

### Hedging Effectiveness

El HE calculado sobre ingreso bruto es bajo porque el precio del café (CV≈0.23) domina la varianza del ingreso frente al rendimiento (CV≈0.11). El HE sobre **rendimiento exclusivo** es el indicador correcto de riesgo climático.

---

## Estructura del repositorio

```
Proyecto_Seguro_Cafe_Indexado/
├── seguro_cafe_completo_vf.ipynb   # Notebook principal (único punto de entrada)
├── requirements.txt                 # Dependencias Python
├── README.md                        # Este archivo
├── data/
│   └── raw/                         # Fuentes originales (cargadas automáticamente)
│       ├── era5_precip_*.csv
│       ├── eva_cafe_*.csv
│       ├── df_tmedia_aire.xlsx
│       ├── df_tmax_aire.xlsx
│       ├── df_tmin_aire.xlsx
│       ├── df_hr.xlsx
│       ├── df_anomalia_temp.xlsx
│       ├── df_modis_anual.csv
│       ├── df_modis_mensual.csv
│       ├── noaa_oni_index.csv
│       ├── Precios-area-y-produccion-de-cafe-2026-3.xlsx
│       └── detalle_agricola_departamental_cafe.csv
└── outputs/                         # Generado al ejecutar el notebook
    ├── kpis_resumen.csv
    ├── activaciones_historicas.csv
    ├── umbrales_departamento.csv
    ├── pred_vs_real.csv
    ├── shap_importancia.csv
    ├── validacion_historica_n1.csv
    ├── d1_holdout_departamental.csv
    ├── tabla_requerimientos_dashboard.csv
    └── metadata_proyecto.csv
```

---

## Equipo

**Proyecto de Investigación Agroeconómica — Modelamiento Climático-Agrícola**

| Rol | Responsabilidad |
|-----|----------------|
| Equipo de modelamiento | Diseño del pipeline, ETL, modelos ML, validación |
| Supervisión académica | Revisión de requerimientos y metodología |

---

## Licencia y fuentes

Todos los datos utilizados son de acceso público: ERA5 (licencia Copernicus), EVA/MADR (datos.gov.co), IDEAM (dominio público Colombia), MODIS (NASA open data), NOAA ONI (dominio público), FNC precios (publicación oficial). El código es de uso académico.
