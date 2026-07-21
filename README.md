# Predicción de Tráfico Vehicular
**Materia:** Series Temporales  
**Autor:** Francisco Moreno
**Institución:** FIUNA  

---

## 1. Descripción del Problema

El objetivo es predecir el volumen de tráfico vehicular para la próxima hora utilizando datos históricos de un sensor IoT instalado en Paraguay. El sensor registra conteos acumulados de vehículos cada 5 minutos, y se busca anticipar la demanda de tráfico con al menos 1 hora de anticipación.

---

## 2. Dataset

| Atributo | Valor |
|---|---|
| Fuente | Sensor IoT (dispositivo dev:0080e11500452428) |
| Registros crudos | 1.359 |
| Período | ~1 semana |
| Frecuencia original | ~5 minutos |
| Frecuencia procesada | 15 minutos |
| Variable objetivo | Vehículos por intervalo de 15 min |

**Nota:** ody.counting es un acumulador — el preprocesamiento correcto es max - min por intervalo, no suma ni promedio.

---

## 3. Metodología

### Preprocesamiento (
otebooks/01_preprocesamiento.ipynb)
- Conversión de Unix timestamp a datetime
- Eliminación de duplicados y nulos
- Determinación del intervalo natural del sensor (5 minutos)
- Resample a 15 minutos con agregación max - min
- Exportación a data/datos_procesados.csv

### EDA (
otebooks/04_eda.ipynb)
- Estadísticas descriptivas
- Serie temporal completa
- Distribución de valores y detección de outliers
- Patrón de tráfico por hora del día
- Patrón de tráfico por día de semana

### Modelado (
otebooks/02_modelos.ipynb)
- Split cronológico 80/20 (el test siempre es el futuro)
- Escalado con MinMaxScaler
- Implementación de N-BEATS y Prophet

---

## 4. Modelos Implementados

### N-BEATS (Deep Learning)
Arquitectura de bloques apilados con mecanismo de backcast/forecast residual implementada en Keras. Hiperparámetros optimizados con Optimización Bayesiana (keras-tuner):
- input_length: [24, 48, 96] intervalos de 15 min
- units: [32, 64]
- 
_blocks: [1, 2, 3]
- learning_rate: [1e-2, 1e-3, 1e-4]

Callbacks: EarlyStopping (patience=10) + ReduceLROnPlateau (patience=5).  
orecast_length=4 (4 × 15min = 1 hora predicha).

### Prophet (Estadístico/ML)
Modelo de Facebook para series temporales con estacionalidad diaria y semanal. Configurado sin estacionalidad anual dado el período corto del dataset.

---

## 5. Resultados y Métricas

| Modelo | MAE | RMSE | MAPE | sMAPE | R² |
|---|---|---|---|---|---|
| N-BEATS | 19.49 | 26.17 | 31.57% | 28.22% | 0.2483 |
| Prophet | 9.52 | 12.59 | 100.84% | 54.75% | -0.0168 |

**Análisis:**
- Prophet tiene menor error absoluto (MAE, RMSE) pero R² negativo, lo que indica que no captura la tendencia de la serie mejor que simplemente predecir la media.
- N-BEATS captura mejor la estructura temporal de la serie (R² positivo) y tiene sMAPE significativamente menor.
- El MAPE elevado de Prophet se debe a predicciones cercanas a 0 en horarios de baja actividad.
- Ambos modelos se beneficiarían de más datos (mínimo 4-8 semanas).

---

## 6. Visualizaciones

| Archivo | Descripción |
|---|---|
| 
esults/eda_serie_completa.png | Serie temporal completa |
| 
esults/eda_distribucion.png | Distribución y boxplot |
| 
esults/eda_patron_hora.png | Tráfico promedio por hora |
| 
esults/eda_patron_dia.png | Tráfico promedio por día |
| 
esults/real_vs_predicho.png | Predicciones vs valores reales |
| 
esults/comparativa_modelos.png | Comparativa de métricas |
| 
esults/residuales.png | Análisis de residuales |

---

## 7. Conclusiones

- La granularidad óptima para este sensor es **15 minutos** — a 5 minutos el acumulador no varía suficiente entre lecturas, y a nivel horario se pierde información.
- N-BEATS supera a Prophet en captura de tendencia (R²) y sMAPE, siendo más adecuado para este tipo de serie.
- El principal limitante es la cantidad de datos — con 1 semana se alcanza el techo de cualquier modelo de deep learning.
- Con 4-8 semanas de datos y posible incorporación de features temporales (hora, día, finde con encoding cíclico) se estima que el MAPE podría bajar de 20%.

---

## 8. Estructura del Repositorio
proyecto_ts/
│
├── data/
│ ├── datos.csv
│ └── datos_procesados.csv
│
├── notebooks/
│ ├── 01_preprocesamiento.ipynb
│ ├── 02_modelos.ipynb
│ ├── 03_evaluacion.ipynb
│ └── 04_eda.ipynb
│
├── results/
│ ├── metricas.csv
│ └── graficos/
│
├── README.md
└── requirements.txt


---

## 9. Reproducibilidad

```bash
pip install -r requirements.txt
```

Ejecutar los notebooks en orden:
1. `01_preprocesamiento.ipynb`
2. `04_eda.ipynb`
3. `02_modelos.ipynb`
4. `03_evaluacion.ipynb`
