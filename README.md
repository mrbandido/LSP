# Reconocimiento de LSP mediante Aprendizaje Profundo con Datos Inerciales

Desarrollo de un modelo de reconocimiento de palabras de la Lengua de Señas Peruana (LSP) mediante aprendizaje profundo, utilizando datos inerciales capturados con guantes hápticos SenseGlove DK3.

Proyecto realizado como parte del Seminario de Investigación I — Universidad de Lima, contribuyendo al proyecto institucional **SignoPeru**.

## Autor

- **Patrick Fabricio Cornejo Huanca** — Ingeniería de Sistemas, Universidad de Lima
- Asesor: Ing. Edwin Escobedo Cárdenas

## Descripción

El sistema captura datos de orientación (quaterniones), flexión de dedos e inercia (IMU) de un par de guantes SenseGlove DK3 de forma bimanual, vía UDP, sin depender del SDK propietario oficial. Con estos datos se construyó un dataset y se entrenaron y compararon tres arquitecturas de deep learning para el reconocimiento de señas del alfabeto dactilológico de la LSP.

## Flujo de uso del proyecto

Este repositorio contiene el **dataset**. El flujo completo del proyecto es:

```
1. Dataset (este repo)
        │
        ▼
2. Entrenamiento de modelos en Google Colab
   (BiLSTM, CNN-1D, Transformer → exporta modelos .h5)
        │
        ▼
3. Interfaz principal de inferencia
   (carga los modelos entrenados y reconoce señas LSP en tiempo real
    a partir de los guantes SenseGlove DK3)
```

La interfaz de inferencia se documentará/subirá por separado; este README se enfocará en el dataset y cómo prepararlo para el paso 2.

## Estructura del repositorio

```
├── dataset-20260701T175232Z-3-001-20260712T010934Z-2-001.zip   # Dataset completo (comprimido)
└── README.md
```

## Dataset

El dataset se encuentra comprimido en:
`dataset-20260701T175232Z-3-001-20260712T010934Z-2-001.zip`

Para usarlo, descomprímelo en la raíz del proyecto:

```bash
unzip dataset-20260701T175232Z-3-001-20260712T010934Z-2-001.zip
```

### Contenido
- **791 archivos CSV**, organizados en carpetas por clase (letra o palabra), ej. `a/`, `b/`, `g/`, etc.
- Cada archivo sigue el formato `letra_repXX.csv` (una repetición de la seña).
- **~45MB** sin comprimir / **~14MB** comprimido.
- **27 clases dactilológicas** (alfabeto completo menos J y Z, que requieren trayectoria espacial no capturable solo con IMU) más frases/palabras comunes.
- **10 sujetos** distintos, con un mínimo de 5 repeticiones por seña (recomendación de asesoría).

### Columnas
Cada fila del CSV corresponde a un frame sincronizado con **157 columnas**, incluyendo:
- Timestamp
- Quaterniones de palma y 5 dedos por mano (LH y RH)
- Valores de flexión (`bend`) por dedo
- Datos de 8 sensores IMU × 6 ejes por mano

## Cómo usar el dataset (paso 2: entrenamiento en Colab)

1. Descomprime `dataset-20260701T175232Z-3-001-20260712T010934Z-2-001.zip` en tu entorno de trabajo (local o subido directo a Colab/Drive).
2. Sube la carpeta descomprimida a Google Drive (o carga el zip directo en Colab y descomprime ahí con `!unzip`).
3. En el notebook de entrenamiento (`LSP_Reconocimiento.ipynb`), monta el Drive y apunta la ruta del dataset a la carpeta descomprimida.
4. El notebook hace el preprocesamiento (padding al percentil-90, `StandardScaler`, `LabelEncoder` + one-hot, split estratificado 70/15/15) y entrena las tres arquitecturas (BiLSTM, CNN-1D, Transformer).
5. Los modelos entrenados se exportan como `.h5`, listos para cargarse en la interfaz de inferencia (repositorio/carpeta separada).

## Metodología

Investigación de tipo aplicada, enfoque cuantitativo, alcance exploratorio-descriptivo, alineada a los ODS 4, 9 y 10.

1. **Fase 1 — Captura de datos**: interfaz Python bimanual sobre UDP (puertos 53453/53454), con calibración automática por mano y renderizado 3D en tiempo real.
2. **Fase 2 — Construcción del dataset**: recolección sincronizada en CSV, con 10 sujetos y mínimo 5 repeticiones por clase.
3. **Fase 3 — Entrenamiento y comparación de modelos**: BiLSTM, CNN-1D y Transformer Encoder, entrenados en Google Colab con TensorFlow/Keras (split estratificado 70/15/15, EarlyStopping y ReduceLROnPlateau).

## Resultados

| Modelo | F1-score |
|---|---|
| CNN-1D | 0.96 |
| BiLSTM | 0.93 |
| Transformer Encoder | 0.40 |

## Limitaciones y trabajo futuro

- Las letras **J y Z** quedaron fuera del alcance por requerir trayectoria espacial; se contempla integrar **MediaPipe** (visión por computadora) en la Investigación II para cubrir señas espaciales y de contacto.
- El pipeline de inferencia en tiempo real fue entrenado principalmente con datos de mano derecha; se recomienda ampliar el dataset con mano izquierda para mejorar la generalización.

## Tecnologías

- Python (captura UDP, interfaz con Tkinter)
- TensorFlow / Keras (BiLSTM, CNN-1D, Transformer)
- Google Colab (entrenamiento)
- SenseGlove DK3 (hardware de captura)
