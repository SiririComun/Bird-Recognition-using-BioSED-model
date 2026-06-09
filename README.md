# Bioacoustic Sound Event Detection (BioSED)

Este proyecto implementa un flujo completo de **Bioacoustic Sound Event Detection (BioSED)** para 10 especies de aves. El pipeline ya está centralizado en el notebook **[Bird_id_with_audio_pipeline.ipynb](Bird_id_with_audio_pipeline.ipynb)**, donde se realiza la extracción de datos desde Xeno-canto, el preprocesamiento de audio, el etiquetado temporal a nivel de frame y el entrenamiento de un modelo **CRNN** en PyTorch.

## Objetivo
Entrenar un modelo capaz de predecir, para cada frame de un clip de audio, la probabilidad de presencia de cada una de las 10 especies. La evaluación se realiza con **Average Precision (AP)** por clase y **mean Average Precision (mAP)** global.

## Notebook principal
El flujo de trabajo está dividido en siete celdas principales dentro de [Bird_id_with_audio_pipeline.ipynb](Bird_id_with_audio_pipeline.ipynb):
1. Configuración del entorno, semillas y dispositivos.
2. Carga de audio y extracción de **Log-Mel Spectrograms**.
3. `Dataset` personalizado, construcción de targets frame-level y `DataLoader`.
4. Arquitectura **CRNN** con CNN 2D + BiGRU + salida multi-etiqueta.
5. Entrenamiento con **BCE Loss** y optimizador **Adam**.
6. Postprocesamiento de predicciones con umbral y reconstrucción de eventos temporales.
7. Evaluación con **mAP** y visualización de predicciones.

## Estructura de carpetas
La organización actual del proyecto es la siguiente, excluyendo la carpeta ignorada:

```text
Fis_comp_2/
├── Bird_id_with_audio_pipeline.ipynb
├── README.md
├── requirements.txt
├── best_model.pth
├── artifacts/
├── dataset_aves/
│   ├── df_etiquetas_fuertes.csv
│   ├── Campylorhynchus_griseus/
│   ├── Crotophaga_ani/
│   ├── Pitangus_sulphuratus/
│   ├── Pygochelidon_cyanoleuca/
│   ├── Thraupis_episcopus/
│   ├── Thraupis_palmarum/
│   ├── Troglodytes_aedon/
│   ├── Turdus_ignobilis/
│   ├── Tyrannus_melancholicus/
│   └── Zonotrichia_capensis/
└── venv/
```

## Dataset
El dataset está organizado en `dataset_aves/` y contiene 500 clips de audio, todos de 10 segundos, perfectamente balanceados entre 10 especies: 50 audios por especie.

Las etiquetas fuertes están en `dataset_aves/df_etiquetas_fuertes.csv`, con columnas para:
- archivo original
- especie detectada
- inicio y fin del evento en segundos
- confianza de la anotación

## Arquitectura del modelo
La red utilizada es una **CRNN** diseñada para bioacústica:
- una **CNN 2D** extrae patrones locales del espectrograma,
- el eje de frecuencia se comprime mediante pooling,
- una **BiGRU** modela la evolución temporal,
- una capa final con **sigmoid** genera probabilidades por frame y por especie.

## Preprocesamiento de audio
El notebook trabaja con los siguientes parámetros base:
- sample rate: `32000 Hz`
- `n_fft = 1024`
- `hop_length = 512`
- `n_mels = 64`
- conversión a mono antes de extraer el espectrograma

## Dependencias
Instala el entorno con:

```bash
pip install -r requirements.txt
```

El archivo `requirements.txt` incluye las dependencias mínimas para ejecutar el notebook, entre ellas `torch`, `torchaudio`, `pandas`, `numpy`, `matplotlib` y `scikit-learn`.

## Uso
1. Activa tu entorno virtual.
2. Instala las dependencias.
3. Abre [Bird_id_with_audio_pipeline.ipynb](Bird_id_with_audio_pipeline.ipynb).
4. Ejecuta las celdas en orden para preparar datos, entrenar el modelo y evaluar resultados.

## Salidas esperadas
Al finalizar el entrenamiento, el notebook produce:
- curvas de pérdida de entrenamiento y validación,
- predicciones frame-level por especie,
- métricas AP por clase,
- valor final de mAP,
- una visualización de ejemplo para inspección temporal.

## Notas
- El notebook está pensado para ejecutarse en CPU o GPU, según disponibilidad.
- Los archivos `.mp3` del dataset se leen directamente desde `dataset_aves/`.
- Si quieres cambiar la resolución temporal o la representación acústica, ajusta `sr`, `n_fft`, `hop_length` y `n_mels` dentro del notebook.