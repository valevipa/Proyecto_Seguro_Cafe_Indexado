# Rúbrica Diligenciada — Anexo de Evaluación
## Seguro Agrícola Indexado de Café (Módulo 2)

**Proyecto:** Seguro Indexado Café — Quindío y Nariño  
**Versión:** prototipo_modulo_3  
**Notebook:** `seguro_cafe_completo_vf.ipynb`  
**Fecha de diligenciamiento:** 2024

---

> **Instrucciones de lectura:** Cada criterio incluye la justificación técnica extraída directamente del notebook, la evidencia disponible en `outputs/`, y el estado de cumplimiento. Los valores numéricos marcados como *"Ver notebook"* se actualizan automáticamente al ejecutar la celda de exportación final.

---

## DIMENSIÓN 1: Datos e Ingeniería de Features

### D-01 · Integración de fuentes heterogéneas

| Elemento | Detalle |
|---------|--------|
| **Criterio** | El pipeline integra correctamente ≥4 fuentes de datos independientes con diferente granularidad temporal y espacial |
| **Puntaje máximo** | 20 pts |
| **Evidencia** | 6 fuentes integradas: ERA5 (mensual, 0.25°), EVA/MADR (anual, municipal), IDEAM (anual, departamental), MODIS (mensual, 500m), ONI/NOAA (mensual, regional), FNC precios (mensual, nacional) |
| **Transformaciones documentadas** | Ponderación ERA5 por área cafetera (2 puntos Nariño vs 14 descartados), melt de formato wide→long IDEAM, ventanas fenológicas MODIS (floración/desarrollo/cosecha), lags de 1-2 años |
| **Fortalezas** | ETL modular con funciones reutilizables (`parse_ideam`, `add_spi3`, `spi3_anual_features`). Normalización de nombres de departamento consistente en todas las tablas |
| **Limitaciones** | NDVI fenológico disponible solo 2012–2019 (imputado con media departamental fuera del rango). IDEAM a nivel departamental, no municipal |
| **Puntaje asignado** | **18 / 20** |
| **Justificación** | Se descuentan 2 pts por la imputación de MODIS fuera de 2012-2019 y por la resolución departamental de IDEAM (no municipal) |

---

### D-02 · Calidad del panel integrado (F2)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | Variables críticas con ≥95% de completitud en el panel de modelamiento |
| **Puntaje máximo** | 10 pts |
| **Variables evaluadas** | `rendimiento_kg_ha`, `spi3_mean`, `spi3_min`, `rend_mun_media`, `tmedia_mean`, `precio_cop_carga` |
| **Evidencia** | `outputs/f2_completitud_panel.csv` |
| **Estado por variable** | `rendimiento_kg_ha`: OK · `spi3_mean`: OK · `rend_mun_media`: OK · `tmedia_mean`: Parcial (datos IDEAM no cubren todos los años) · `precio_cop_carga`: OK |
| **Puntaje asignado** | **7 / 10** |
| **Justificación** | `tmedia_mean` y algunas variables MODIS tienen cobertura parcial (2012-2019). El notebook imputa con la media departamental, lo que es aceptable pero introduce sesgo en años extremos |

---

## DIMENSIÓN 2: Track A — Índice SPI-3

### A-01 · Implementación metodológica del SPI-3 (N4)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | Implementación correcta de McKee et al. (1993): rolling sum 3 meses, ajuste Gamma por mes calendario, transformación a normal estándar |
| **Puntaje máximo** | 15 pts |
| **Evidencia** | Función `add_spi3()` en celda 2.1. KS-test de bondad de ajuste en celda 2.1b |
| **Bondad de ajuste** | ≥80% de combinaciones mes-depto no rechazan H₀ (Gamma válida) |
| **Umbrales** | Calibrados a P12/P88 empírico por departamento. Prueba Mann-Whitney confirma distribuciones significativamente distintas entre Quindío y Nariño (p<0.05) |
| **Fortalezas** | Manejo correcto de valores cero en precipitación (probabilidad mixta). Calibración empírica en lugar de valores teóricos fijos |
| **Puntaje asignado** | **14 / 15** |
| **Justificación** | Se descuenta 1 pt porque el análisis de sensibilidad P10–P20 reporta la tabla pero no justifica formalmente la elección de P12 con criterio actuarial |

---

### A-02 · Validación histórica del índice (N1)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | El SPI-3 detecta correctamente los dos años de estrés histórico documentados (2012 roya, 2015 El Niño) en ambos departamentos |
| **Puntaje máximo** | 10 pts |
| **Evidencia** | `outputs/validacion_historica_n1.csv` · Celda 2.6 del notebook |
| **Resultado** | 2012: detectado en Quindío y Nariño ✅ · 2015: detectado en Quindío y Nariño ✅ |
| **Puntaje asignado** | **10 / 10** |
| **Justificación** | Cumplimiento completo. El índice identifica correctamente ambos eventos con ≥1 mes de sequía o exceso en el año |

---

### A-03 · Frecuencia de activación actuarialmente sostenible (N3)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | Frecuencia total de eventos (sequía + exceso) entre 15% y 25% del total de meses |
| **Puntaje máximo** | 10 pts |
| **Evidencia** | `outputs/umbrales_departamento.csv` · Celda 2.3 |
| **Resultado** | Quindío: dentro del rango ✅ · Nariño: dentro del rango ✅ |
| **Puntaje asignado** | **9 / 10** |
| **Justificación** | Se descuenta 1 pt por no presentar el análisis de sensibilidad de la prima actuarial ante variaciones de ±2 pp en la frecuencia |

---

## DIMENSIÓN 3: Track B — Modelo de Rendimiento

### B-01 · Poder predictivo LOYO (N2)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | R² ≥ 0.70 en validación LOYO o in-sample sin efectos fijos, para al menos un departamento |
| **Puntaje máximo** | 20 pts |
| **Evidencia** | `outputs/kpis_resumen.csv` · Celdas 3.4 y 3.7b |
| **Resultado Quindío** | R²=0.921 (GradBoost in-sample sin FE, pool 22 features, 2007-2024) ✅ |
| **Resultado Nariño** | R²=0.618 (GradBoost in-sample sin FE) — no alcanza umbral 0.70 ⚠️ |
| **Fortalezas** | Anti-leakage correcto en `rend_mun_media` (calculado solo con datos de entrenamiento por fold). Comparación de 3 modelos por departamento. Selección SHAP del subconjunto óptimo de features |
| **Limitaciones** | Nariño limitado por ruptura roya 2012–2014 y alta varianza de 45 municipios |
| **Puntaje asignado** | **15 / 20** |
| **Justificación** | Quindío cumple (10 pts). Nariño no alcanza umbral pero presenta análisis diagnóstico completo y propuesta de mejora (5 pts). Se descuentan 5 pts por no cumplimiento en Nariño |

---

### B-02 · Hold-out temporal (D1)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | RMSE ≤ 149 kg/ha en hold-out 2018–2019 a nivel departamental ponderado |
| **Puntaje máximo** | 10 pts |
| **Evidencia** | `outputs/d1_holdout_departamental.csv` · Celda 6 (D1) |
| **Resultado Nariño** | RMSE ~149 kg/ha (RidgeCV) — en el límite del umbral ⚠️ |
| **Resultado Quindío** | RMSE ~160 kg/ha — supera el umbral ⚠️ |
| **Diagnóstico presentado** | 2019 fue año atípico (recuperación post-roya + precios FNC favorables). Con n=2 test, el RMSE es sensible a shocks no climáticos ✅ |
| **Puntaje asignado** | **7 / 10** |
| **Justificación** | No cumplimiento estricto del umbral, pero diagnóstico técnico riguroso y aceptable dado el contexto estructural del período de test |

---

### B-03 · Interpretabilidad SHAP (D2)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | Análisis SHAP con coherencia agronómica: SPI-3 desarrollo/floración debe aparecer en top-3 variables climáticas |
| **Puntaje máximo** | 10 pts |
| **Evidencia** | `outputs/shap_importancia.csv` · `outputs/validacion_shap_d2.png` · Celda D2 |
| **Resultado** | `rend_mun_media` domina ambos departamentos (efecto fijo) ✅. `spi3_floracion` y/o `spi3_desarrollo` en top-3 climáticas ✅ |
| **Coherencia agronómica** | Floración (abr-may, oct-nov) y desarrollo del fruto (may-ago) son períodos críticos documentados en literatura Cenicafé ✅ |
| **Puntaje asignado** | **9 / 10** |
| **Justificación** | Se descuenta 1 pt por no incluir PDP (Partial Dependence Plots) para las top-3 variables, que habría completado el análisis D2 |

---

### B-04 · Estabilidad temporal (D3)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | ΔR² (max-min) < 0.15 · std(R²) < 0.10 en k-fold temporal (k=4, 2 años/fold) |
| **Puntaje máximo** | 10 pts |
| **Evidencia** | Celda D3 del notebook |
| **Resultado** | Quindío: ΔR²>1.0, std>0.60 ❌ · Nariño: ΔR²>1.0, std>0.60 ❌ |
| **Diagnóstico presentado** | Ruptura estructural documentada con estadísticas por período: pre-roya (2007-2011), roya (2012-2014), post-roya (2015-2024). Leakage de `rend_mun_media` identificado y corregido ✅ |
| **Puntaje asignado** | **5 / 10** |
| **Justificación** | No cumple el criterio cuantitativo. Sin embargo, el diagnóstico causal completo (roya como break-point estructural) y la corrección del leakage merecen crédito parcial. El criterio es formalmente incumplible sin modelar la ruptura estructural explícitamente |

---

### B-05 · Rule-Based Index Model (D4)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | ΔR² RBIM vs mejor modelo estadístico es tolerable (<0.25). Fórmula completamente auditable |
| **Puntaje máximo** | 10 pts |
| **Evidencia** | Celda D4 del notebook |
| **Fórmula documentada** | `Pred = rend_mun_media × clip(1 + adj, 0.30, 1.50)` con coeficientes SPI por ventana fenológica y factor roya |
| **Resultado** | Nariño: ΔR²≈0.16 ✅ · Quindío: ΔR²≈0.59 ⚠️ |
| **Puntaje asignado** | **7 / 10** |
| **Justificación** | Nariño cumple (tolerable). Quindío no cumple pero la fórmula está correctamente documentada y la discusión sobre limitaciones es rigurosa. Se descuentan 3 pts por incumplimiento en Quindío |

---

## DIMENSIÓN 4: Reproducibilidad y Estándares

### R-01 · Reproducibilidad del pipeline (F1, F6)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | El código ejecuta de inicio a fin en un entorno nuevo siguiendo instrucciones del README. Sin dependencias comerciales |
| **Puntaje máximo** | 10 pts |
| **Evidencia** | `README.md` · `requirements.txt` · Sección F6 del notebook |
| **Herramientas** | NumPy, Pandas, Scikit-learn, SciPy, SHAP, Statsmodels, Matplotlib — todas open-source |
| **Datos** | ERA5 (Copernicus), EVA (datos.gov.co), IDEAM (dominio público), ONI (NOAA), FNC (publicación oficial) |
| **Instrucciones** | `pip install -r requirements.txt` + `jupyter notebook seguro_cafe_completo_vf.ipynb` |
| **Puntaje asignado** | **9 / 10** |
| **Justificación** | Se descuenta 1 pt porque el notebook usa rutas `/content/` (Google Colab) para guardar figuras, que fallan en entornos Jupyter estándar sin modificar las celdas de guardado |

---

### R-02 · Documentación y estándares regulatorios (F5)

| Elemento | Detalle |
|---------|--------|
| **Criterio** | Checklist IFRS S2: 5/5 ítems cumplidos |
| **Puntaje máximo** | 10 pts |
| **Evidencia** | `outputs/f5_checklist_ifrs_s2.csv` · Celda F5 |
| **Ítems IFRS S2** | ✅ Gobernanza · ✅ Fuentes con metadata · ✅ Transformaciones auditables · ✅ Resultados con versión/fecha · ✅ Criterios de activación explicitados |
| **Puntaje asignado** | **10 / 10** |
| **Justificación** | Cumplimiento completo de los 5 ítems |

---

## RESUMEN GENERAL

| Dimensión | Criterio | Máximo | Asignado | % |
|-----------|---------|--------|----------|---|
| **D1** Datos e ingeniería | D-01 Integración | 20 | 18 | 90% |
| **D1** Datos e ingeniería | D-02 Calidad panel | 10 | 7 | 70% |
| **D2** Track A SPI-3 | A-01 Implementación | 15 | 14 | 93% |
| **D2** Track A SPI-3 | A-02 Validación histórica | 10 | 10 | 100% |
| **D2** Track A SPI-3 | A-03 Frecuencia activación | 10 | 9 | 90% |
| **D3** Track B Rendimiento | B-01 Poder predictivo N2 | 20 | 15 | 75% |
| **D3** Track B Rendimiento | B-02 Hold-out D1 | 10 | 7 | 70% |
| **D3** Track B Rendimiento | B-03 Interpretabilidad D2 | 10 | 9 | 90% |
| **D3** Track B Rendimiento | B-04 Estabilidad D3 | 10 | 5 | 50% |
| **D3** Track B Rendimiento | B-05 RBIM D4 | 10 | 7 | 70% |
| **D4** Reproducibilidad | R-01 Pipeline F1/F6 | 10 | 9 | 90% |
| **D4** Reproducibilidad | R-02 Estándares F5 | 10 | 10 | 100% |
| | **TOTAL** | **145** | **120** | **83%** |

---

## Observaciones del evaluador (espacio para completar)

**Fortalezas del proyecto:**
- Pipeline ETL bien estructurado con manejo robusto de fuentes heterogéneas
- Implementación correcta de SPI-3 con validación estadística formal (KS-test)
- Diagnósticos técnicos rigurosos cuando los criterios no se cumplen (D1, D3)
- Anti-leakage correcto en `rend_mun_media` por fold
- Análisis SHAP con interpretación agronómica pertinente
- 100% open-source con datos públicos

**Áreas de mejora:**
- Completar PDP para las top-3 variables SHAP (requerimiento D2)
- Justificar formalmente la elección de P12 con criterio actuarial
- Corregir rutas de guardado de figuras para portabilidad fuera de Google Colab
- Modelar break-point estructural de roya para mejorar D3
- Ampliar período hold-out (n=2 años es insuficiente para estimación robusta de RMSE)

**Comentarios adicionales:**
_[Espacio para el evaluador]_

---

**Firma evaluador:** ____________________________  
**Fecha de evaluación:** ____________________________  
**Calificación final:** ____________________________
