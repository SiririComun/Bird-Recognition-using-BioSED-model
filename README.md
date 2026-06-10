# BioSED: Bird Recognition Using Bioacoustic Sound Event Detection

Proyecto final de Fisica Computacional 2: Aprendizaje Estadistico.

El objetivo es construir un sistema capaz de detectar, a partir de audio, cual de las 10 especies de aves objetivo esta vocalizando y en que intervalo temporal ocurre el canto. El enfoque principal es Bioacoustic Sound Event Detection: el modelo predice probabilidades por frame temporal y por especie.

## Estado Del Proyecto

El repositorio esta en proceso de refactorizacion hacia una estructura autocontenida en notebooks. La decision de diseno es mantener el proyecto como entrega academica reproducible, no como paquete de produccion.

La version inicial del pipeline vive en:

```text
Bird_id_with_audio_pipeline.ipynb
```

Ese notebook contiene descarga de datos, pseudo-etiquetado, preprocesamiento, entrenamiento, evaluacion e inferencia en un solo flujo. La refactorizacion separara esas responsabilidades en notebooks mas pequenos y autocontenidos.

## Estructura Objetivo

```text
notebooks/
|-- 01_dataset_y_etiquetado.ipynb
|-- 02_EDA_dataset_y_mels.ipynb
|-- 03_entrenamiento_CRNN_BioSED.ipynb
|-- 04_evaluacion_modelo.ipynb
`-- 05_dashboard_demo.ipynb

artifacts/
|-- biosed_crnn_checkpoint.pth
|-- figures/
|-- metrics/
`-- splits/

dataset_aves/
|-- df_metadata_audios.csv
|-- df_etiquetas_fuertes.csv
`-- carpetas_por_especie/
```

## Flujo Esperado

1. `01_dataset_y_etiquetado.ipynb`: descarga audios desde Xeno-canto, valida archivos y genera pseudo-etiquetas temporales con BirdNET.
2. `02_EDA_dataset_y_mels.ipynb`: analiza el dataset, las etiquetas, las senales de audio y los espectrogramas Log-Mel.
3. `03_entrenamiento_CRNN_BioSED.ipynb`: entrena el modelo CRNN de deteccion temporal.
4. `04_evaluacion_modelo.ipynb`: evalua el checkpoint con mAP, AP por clase, curvas precision-recall, matriz de confusion y analisis de errores.
5. `05_dashboard_demo.ipynb`: carga el checkpoint y permite probar el modelo con audios subidos o grabados durante la exposicion.

Los notebooks deben ser autocontenidos: cada uno define sus imports, configuracion, funciones necesarias y verificaciones de entrada.

## Dataset

El dataset local esperado se almacena en `dataset_aves/`. Los audios crudos no se versionan porque pueden ser pesados y se pueden reconstruir o copiar localmente.

Los CSV pequenos de metadatos y pseudo-etiquetas si pueden versionarse:

```text
dataset_aves/df_metadata_audios.csv
dataset_aves/df_etiquetas_fuertes.csv
```

Esto permite reproducir analisis, splits y evaluaciones sin subir los archivos de audio al repositorio.

Las etiquetas temporales se tratan como pseudo-etiquetas porque son generadas automaticamente con BirdNET. Esto debe tenerse en cuenta al interpretar las metricas: la evaluacion mide acuerdo con pseudo-ground-truth, no necesariamente con anotacion humana perfecta.

## Politica De Versionamiento

Se versiona:

- notebooks finales del proyecto;
- `README.md`, `requirements.txt`, `.gitignore` y archivos pequenos de configuracion;
- CSVs pequenos de metadatos y pseudo-etiquetas;
- checkpoint oficial si su tamano sigue siendo razonable;
- metricas, splits y figuras finales si son utiles para reproducibilidad o presentacion.

No se versiona:

- audios crudos o derivados pesados;
- entornos virtuales;
- caches de Python/Jupyter;
- notebooks temporales o de descarte;
- archivos locales de planificacion.

## Modelo

El enfoque base usa una CRNN:

```text
audio -> Log-Mel Spectrogram -> CNN 2D -> BiGRU -> sigmoid por frame y clase
```

La salida esperada tiene forma:

```text
P(especie c activa en frame t | audio)
```

Por eso el problema se formula como clasificacion multi-label frame-level y se entrena con Binary Cross Entropy.

## Instalacion

Se recomienda crear un entorno virtual y luego instalar dependencias:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Para algunos formatos de audio puede ser necesario tener `ffmpeg` instalado en el sistema.

## Artefactos

Los checkpoints actuales estan en `artifacts/`. Durante la refactorizacion se elegira un checkpoint oficial, idealmente:

```text
artifacts/biosed_crnn_checkpoint.pth
```

Ese checkpoint debe incluir pesos, clases, configuracion de audio, seed y metadatos suficientes para inferencia y evaluacion.

## Notas De Reproducibilidad

- `dataset_aves/` puede contener CSVs pequenos versionados, pero no audios.
- Los audios crudos no deben subirse al repositorio.
- Los notebooks 03 y 04 deben poder ejecutarse si ya existen datos, splits y checkpoint.
- El notebook 05 debe depender solo del checkpoint y del audio de entrada.
