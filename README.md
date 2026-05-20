# Seguro Agrícola Indexado de Café — Quindío y Nariño

**Universidad de los Andes · MIAD · 2026**

> Un tercero puede reproducir resultados end-to-end en ≤ 30 min siguiendo estas instrucciones sin contactar al equipo.

---

## Problema

Menos del 3 % del área cafetera colombiana tiene seguro agrícola. Los seguros tradicionales requieren ajustadores de campo para verificar pérdidas individuales, haciéndolos prohibitivamente costosos para pequeños caficultores. Este proyecto diseña y valida un **seguro indexado** basado en el índice climático SPI-3 (Standardized Precipitation Index, escala 3 meses) que activa pagos automáticamente cuando la precipitación supera umbrales predefinidos por departamento, eliminando el riesgo moral y los costos de ajuste.

**Meta:** reducir el riesgo base de ~25–30 % a ≤ 15–20 % mediante un índice climático que refleje mejor las pérdidas reales del productor.

El modelo cubre los departamentos de **Quindío y Nariño** (Colombia) con datos históricos 2000–2024 y opera en dos tracks complementarios:

| Track | Pregunta central |
|-------|-----------------|
| **A — Índice SPI-3** | ¿Qué umbrales de precipitación deben activar el seguro? |
| **B — Rendimiento** | ¿El índice climático predice pérdidas reales de café (kg/ha)? |

---

## Instalación

### Requisitos previos

- Python ≥ 3.9
- Jupyter Notebook o JupyterLab (o Google Colab)
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
jupyter notebook notebooks/seguro_cafe_completo_vf.ipynb
```

> **Google Colab:** abrir el notebook directamente desde GitHub o subir el repo. Los datos se cargan automáticamente desde la ruta `BASE` configurada en la celda 0.

---

## Datos

Todas las fuentes son **públicas y sin costo**. Los archivos residen en `data/raw/`.

| Fuente | Archivo | Cobertura | Variable(s) | MD5 |
|--------|---------|-----------|-------------|-----|
| **ERA5** (ECMWF / Open-Meteo) | `era5_precip_quindio_narino_consolidado.csv` | 2000–2017 · mensual | Precipitación mm/mes | `3eaf5e` |
| **ERA5** extensión | `era5_precip_quindio_narino_2018_2024.csv` | 2018–2024 · mensual | Precipitación mm/mes | `bec102` |
| **EVA** (MADR / Socrata) | `eva_cafe_quindio_narino_actualizado.csv` | 2007–2024 · municipal | Rendimiento kg/ha, área ha | `a38213` |
| **NOAA CPC** | `noaa_oni_index.csv` | 1950–2026 | Índice El Niño/La Niña (ONI) | `7da1b8` |
| **IDEAM** | `df_tmedia_aire.xlsx` | 1972–2024 | Temperatura media (°C) | `e93037` |
| **IDEAM** | `df_tmax_aire.xlsx` | 1972–2024 | Temperatura máxima (°C) | `386368` |
| **IDEAM** | `df_tmin_aire.xlsx` | 1972–2024 | Temperatura mínima (°C) | `3e2589` |
| **IDEAM** | `df_hr.xlsx` | 1972–2024 | Humedad relativa (%) | `949826` |
| **IDEAM** | `df_anomalia_temp.xlsx` | 2000–2024 | Anomalía temperatura (°C) | `4dfe90` |
| **MODIS** (NASA) | `df_modis_anual.csv` | 2012–2019 | NDVI/EVI anual | `be436b` |
| **MODIS** (NASA) | `df_modis_mensual ndvi.csv` | 2012–2019 | NDVI/EVI por ventana fenológica | `244d39` |
| **FNC / Almacafé** | `Precios-area-y-produccion-de-cafe-2026-3.xlsx` | 1944–2026 | Precio interno COP/carga 125 kg | `49d608` |
| **FNC** | `detalle_agricola_departamental_cafe.csv` | 2007–2024 | Área cosechada/sembrada ha | `2ce0c5` |

> Los MD5 completos están en [`data/checksums.md5`](data/checksums.md5).  
> **Nota ERA5 Nariño:** 14 de los 16 puntos de grilla cubren la costa Pacífica (sin café). Se aplica ponderación por área cafetera EVA: punto (1.5°N, 77.5°W) = 95.5 %, punto (1.5°N, 76.5°W) = 4.5 %.

**Panel final:** 18 años × 2 departamentos = **36 observaciones departamentales**; 922 observaciones a nivel municipal.

---

## Ejecución

El notebook está diseñado para ejecutarse **de principio a fin** con `Kernel → Restart & Run All`. Tiempo estimado: **15–25 min** en Colab (CPU estándar).

```
Sección 0  · Configuración, imports, semilla SEED=42
Sección 1  · ETL — carga y limpieza de 13 fuentes de datos
Sección 2  · Track A — SPI-3, eventos, validación N1–N4, bondad de ajuste Gamma
Sección 3  · Track B — panel integrado, LOYO, SHAP, D1–D4, supuestos S1–S5
Sección 4  · Tabla resumen de requerimientos
Sección 5  · Hedging Effectiveness (HE)
Sección 6  · Requerimientos funcionales F2, F5, F6
```

### Reproducibilidad

| Elemento | Valor trazado |
|----------|--------------|
| Semilla aleatoria | `SEED = 42` (celda 0, propagada a todos los modelos) |
| Versión datos ERA5 | Open-Meteo ERA5 reanalysis, descarga 2026-05-04 |
| Versión datos EVA | API Socrata `2pnw-mmge` + `uejq-wxrr`, descarga 2026-05-04 |
| Versión datos ONI | NOAA CPC `cwlink/home/indices/oni/ascii...`, descarga 2026-05-04 |
| Fallback ONI | Diccionario hardcoded 2000–2024 (celda 27) si NOAA inaccesible |
| Python | ≥ 3.9 |
| Dependencias | `requirements.txt` — versiones exactas fijadas |

### Outputs generados

Al finalizar, la carpeta `outputs/` contiene:

| Archivo | Contenido |
|---------|-----------|
| `kpis_resumen.csv` | R², RMSE, HE por departamento |
| `activaciones_historicas.csv` | Serie SPI-3 mensual con clasificación de eventos |
| `umbrales_departamento.csv` | Umbrales P12/P88 calibrados y frecuencias |
| `pred_vs_real.csv` | Predicciones LOYO vs rendimiento real por municipio |
| `shap_importancia.csv` | Importancia SHAP por variable y departamento |
| `validacion_historica_n1.csv` | Detección años 2012 (roya) y 2015 (El Niño) |
| `d1_holdout_departamental.csv` | RMSE/MAE hold-out 2018–2019 nivel departamental |
| `tabla_requerimientos_dashboard.csv` | Estado N1–N4, D1–D4, F1–F6 |
| `metadata_proyecto.csv` | Versión, fecha y configuración experimental |

---

## Resultados

### Track A — Índice SPI-3

| Req | Criterio | Resultado | Estado |
|-----|---------|-----------|--------|
| **N1** Validación histórica | Detectar sequía 2012 y El Niño 2015 | 4 meses activados en ambos eventos y departamentos | ✅ |
| **N2** Poder predictivo | R² ≥ 0.70 in-sample (sin FE) | Quindío **0.921** · Nariño **0.618** | ⚠️ Parcial |
| **N3** Frecuencia activación | 15–25 % de meses históricos | ~21 % ambos departamentos | ✅ |
| **N4** Umbrales diferenciados | P12/P88 por departamento | Nariño ≠ Quindío · diferencia significativa | ✅ |

**Umbrales calibrados (ERA5 ponderado 2000–2024):**

| Departamento | Umbral sequía (P12) | Umbral exceso (P88) | Frec. activación |
|---|---|---|---|
| Nariño | SPI-3 ≤ −0.827 | SPI-3 ≥ +1.813 | ~21 % |
| Quindío | SPI-3 ≤ −1.137 | SPI-3 ≥ +1.221 | ~21 % |

**Bondad de ajuste Gamma (KS-test):** 0/24 combinaciones mes × departamento rechazan H₀ (p > 0.05) → ajuste válido al 100 %.

### Track B — Modelo de rendimiento

| Req | Criterio | Resultado | Estado |
|-----|---------|-----------|--------|
| **D1** RMSE hold-out | ≤ 149 kg/ha (nivel departamental) | Nariño **135.8** · Quindío **65.2** kg/ha | ✅ |
| **D2** Coherencia SHAP | SPI-3 desarrollo en top-3 climático | Confirmado ambos departamentos | ✅ |
| **D3** Estabilidad temporal | ΔR² < 0.15 · std < 0.10 | ΔR² > 1.0 (ruptura estructural roya 2012–2014) | ⚠️ |
| **D4** RBIM vs estadístico | ΔR² tolerable | Nariño ΔR²=0.16 · Quindío ΔR²=0.59 | ⚠️ Parcial |

> **Nota D3:** la inestabilidad proviene del choque fitosanitario de la roya (2012–2014), no del modelo climático. Al excluir esos años del training, `rend_mun_media` sobreestima el rendimiento base → R² negativo en el fold. El SPI-3 es estacionario; la ruptura es biológica.

### Hedging Effectiveness

| Departamento | HE varianza | Riesgo base | Prima actuarial |
|---|---|---|---|
| Nariño | −0.006 | 44.4 % | 9.9 % |
| Quindío | −0.026 | 38.9 % | 9.5 % |

HE negativo porque el precio del café (CV = 0.52) domina la varianza del ingreso frente al rendimiento (CV = 0.11). Mejora recomendada: trigger compuesto SPI-3 + precio FNC + ONI.

### Supuestos modelo Ridge LOYO (S1–S5)

| Supuesto | Test | Nariño | Quindío | Impacto |
|----------|------|--------|---------|---------|
| S1 Linealidad | corr(y, ŷ) | 0.511 ✅ | 0.562 ✅ | Bajo |
| S2 Multicolinealidad | VIF | 15/17 > 100 ⚠️ | 16/17 > 100 ⚠️ | Justifica Ridge L2 |
| S3 Normalidad | Shapiro-Wilk + JB | p=0.034 ❌ | p=0.152 ✅ | IC formales solo Quindío |
| S4 Autocorrelación | Durbin-Watson | DW=1.609 ✅ | DW=0.975 ⚠️ | Errores std Quindío subestimados |
| S5 Homocedasticidad | Spearman\|ε\|,ŷ | p=0.026 ⚠️ | p=0.001 ⚠️ | Leve — no invalida predicción |

---

## Estructura del repositorio

```
Proyecto_Seguro_Cafe_Indexado/
├── notebooks/
│   ├── seguro_cafe_completo_vf.ipynb   # Notebook principal — único punto de entrada
│   └── outputs/                        # CSVs y gráficos generados por el notebook
├── data/
│   ├── raw/                            # 13 fuentes originales (ver tabla Datos)
│   ├── processed/                      # Datos intermedios generados por el notebook
│   └── checksums.md5                   # Hashes MD5 de todos los archivos raw
├── outputs/                            # Outputs exportados para dashboard
├── src/                                # Módulos Python reutilizables (extensión futura)
├── requirements.txt                    # Dependencias con versiones exactas
├── README.md                           # Este archivo
├── pipeline_seguro_cafe.pdf            # Diagrama esquemático del pipeline completo
├── reporte_experimentos.md             # Configuración experimental, modelos y resultados
└── rubrica_diligenciada.md             # Rúbrica M3 diligenciada con evidencia por criterio
```

---

## Evidencia de pruebas

El pipeline fue verificado mediante los siguientes mecanismos documentados:

| Tipo | Qué verifica | Celda / Archivo |
|------|-------------|-----------------|
| **KS-test Gamma** (24 tests) | Validez distribución para SPI-3 | Celda 35 — 0/24 rechazan H₀ |
| **LOYO temporal** (11 folds) | Generalización out-of-sample Track B | Celda 61 |
| **Hold-out 2018–2019** | RMSE en datos nunca vistos (D1) | Celda 73 |
| **Anti-leakage rend_mun_media** | No contaminación train/test | Celda 60, 73 |
| **Supuestos S1–S5** (tests formales) | Validez estadística del modelo Ridge | Celda 63 |
| **Checksums MD5** | Integridad de archivos de datos | `data/checksums.md5` |
| **SEED=42** (todas las celdas) | Reproducibilidad numérica exacta | Celda 0 |

Para verificar integridad de datos antes de ejecutar:

```bash
md5sum -c data/checksums.md5
```

---

## Hallazgos clave

1. **Gamma válida al 100 %**: KS-test confirma ajuste en las 24 combinaciones mes × departamento. Justificación estadística formal del SPI-3.
2. **Ponderación ERA5 Nariño**: 14 de 16 puntos de grilla cubren zonas sin café. La ponderación por área cafetera EVA resuelve la sub-detección de sequías (Nariño 2012: 0 → 4 meses detectados).
3. **ONI segundo predictor**: `oni_mean_lag1` es la segunda variable más importante por SHAP (~15 %) en ambos departamentos, por encima de las variables SPI-3 individuales.
4. **Selección SHAP mejora generalización**: reducir de 17 a 6–7 features baja el RMSE hold-out de 167–173 a 65–136 kg/ha (D1 cumple).
5. **Roya 2012–2014 como ruptura estructural**: rendimiento cae ~15 % Nariño / ~12 % Quindío. Causa principal de inestabilidad D3 y techo de N2 en Nariño.
6. **Riesgo base alto (~39–44 %)**: el SPI-3 explica ~25 % de la varianza del rendimiento municipal. El resto es efecto fijo municipal, precio FNC y fitosanidad.

---

## Anexos

| Documento | Descripción | Ruta |
|-----------|-------------|------|
| **Diagrama del pipeline** | Esquema visual del flujo ETL → Track A → Track B → outputs | `pipeline_seguro_cafe.pdf` |
| **Reporte de experimentos** | Configuración experimental completa: datasets, modelos, hiperparámetros, resultados por versión | `reporte_experimentos.md` |
| **Rúbrica diligenciada** | Evaluación criterio a criterio (N1–N4, D1–D4, F1–F6) con evidencia, limitaciones y recomendaciones | `rubrica_diligenciada.md` |

---

## Equipo

**Universidad de los Andes · MIAD 2026 · Proyecto Aplicado**

| Integrante | Rol |
|-----------|-----|
| Luis Miguel Pérez | Modelamiento, ETL, validación, documentación |

---

## Bibliografía clave

| Referencia | Aporte |
|-----------|--------|
| Avelino et al. (2015). *Food Security*, 7, 303–321 | Cuantifica caída 31 % producción por roya — base del dummy `roya` |
| Bebber & Castillo (2016). *Phil. Trans. R. Soc. B*, 371 | Modela riesgo roya con ERA5, confirma período 2012–2014 |
| Lobell & Burke (2010). *Nature Climate Change* | Umbral RMSE 10–24 % del rango — referencia D1 |
| Abregó-Pérez et al. (2025). *Agric. Systems* | SHAP para café colombiano — referencia D2 y umbral 60 % |
| McKee et al. (1993). *8th AMS Conf. Applied Climatology* | Definición estándar SPI |
| IPCC (2021). *AR6 WG1* | Criterio ΔR² < 0.15 estabilidad temporal — referencia D3 |

---

## Licencia

Proyecto académico — Universidad de los Andes · MIAD 2025–2026.  
Datos: ERA5 (Copernicus open), EVA (datos.gov.co abierto), IDEAM (dominio público Colombia), MODIS (NASA open), NOAA ONI (dominio público), FNC (publicación oficial).
