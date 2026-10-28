# Transformer espacialmente interpretable para evaluar el deterioro cognitivo a partir de EEG

Código del estudio *"Spatially Interpretable Transformer Learning for EEG-Based
Cognitive Decline Assessment Through Functional Geostatistical Biomarkers"*.
El repositorio reúne los dos notebooks de Colab que sustentan el artículo: el
análisis de geoestadística funcional que produce el prior espacial y los
biomarcadores, y el Transformer PSD-funcional que usa ese prior como sesgo de
atención espacial.

*(An English version of this README is available in `README.en.md`.)*

## Qué hace el método

El EEG en reposo se clasifica como *Normal* frente a *Deteriorado* (DCL +
demencia). La densidad espectral de potencia (PSD) de cada canal se representa
como una expansión en B-splines penalizados, de modo que el modelo trabaja sobre
un descriptor funcional en lugar de potencias por banda calculadas a mano. Un
trace-variograma conjunto, estimado con distancias de círculo máximo entre
electrodos y seleccionado por el criterio de información de Akaike corregido
(AICc), genera un sesgo de atención espacial `B_spatial` que se suma a los
puntajes de atención. El mismo análisis geoestadístico funciona después como
capa de interpretabilidad: relaciona la atención con la coherencia espacial sobre
el cuero cabelludo (test de Mantel y concordancia a nivel de canal).

## Notebooks

| Notebook | Propósito |
|----------|-----------|
| `notebooks/01_functional_geostatistics.ipynb` | Estimación del trace-variograma, selección de modelo (AICc), biomarcadores a nivel de sujeto y la matriz `B_spatial` que usa el Transformer. |
| `notebooks/02_psd_functional_transformer.ipynb` | Transformer PSD-funcional con el sesgo de atención espacial, validación cruzada agrupada por paciente, validación externa, prueba de equivalencia (TOST) y la batería de ablaciones. Se incluye ejecutado, con sus salidas, para ver los resultados sin volver a correrlo. |

Ambos notebooks están pensados para Google Colab (montan Google Drive y leen el
dataset desde ahí). Corren sin cambios en un Jupyter local una vez que `BASE_DIR`
apunta a una copia local de los datos.

## Módulos reutilizables

`src/` contiene los bloques del artículo extraídos de los notebooks:

- `src/geostatistics.py` — trace-variograma y selección de modelo (AICc), el
  descriptor funcional de la PSD (B-splines penalizados), los biomarcadores a
  nivel de sujeto y las utilidades de montaje/geometría.
- `src/transformer.py` — el Transformer PSD-funcional y su sesgo de atención
  espacial, el flujo de datos, el entrenamiento y la validación cruzada, la
  validación externa, la interpretabilidad (test de Mantel, concordancia
  atención–variograma) y las variantes de ablación.

Cada módulo conserva el cuerpo de las funciones y clases tal cual; de los nombres
que los notebooks redefinieron mientras se iteraba, se guarda la última versión.
La ejecución de principio a fin (rutas de datos, configuración, entrenamiento)
permanece en los notebooks, de los que algunas funciones toman sus constantes de
configuración.

## Datos

El estudio usa dos conjuntos de datos; aquí no se redistribuye ninguno.

- **CAUEEG** (970 sujetos; 394 CN, 329 DCL, 247 DEM). El acceso lo conceden los
  autores originales previa solicitud: tras su revisión facilitan el enlace de
  descarga y la contraseña. El conjunto, su recolección y su uso se describen en
  el artículo de datos de CAUEEG (Kim et al.). Coloca los datos descargados donde
  los notebooks los esperan y ajusta `BASE_DIR`:

  ```python
  BASE_DIR = ".../caueeg-dataset"   # tu ruta local o de Drive
  ```

- **OpenNeuro ds004504** (n = 88), usado para validación externa. Disponible
  abiertamente en https://openneuro.org/datasets/ds004504.

## Entorno

Python 3.10 o superior. Instala las dependencias con:

```bash
pip install -r requirements.txt
```

En Colab vienen preinstaladas salvo `mne`, `mne-icalabel`, `autoreject` y
`scikit-fda`, que instalan las primeras celdas del notebook. Para el Transformer
conviene un entorno con GPU.

## Cómo ejecutarlo

1. Solicita el acceso a CAUEEG, descarga los datos y define `BASE_DIR`.
2. Corre `01_functional_geostatistics.ipynb` para ajustar el variograma y
   construir `B_spatial`; el notebook guarda sus salidas en la ruta indicada.
3. Corre `02_psd_functional_transformer.ipynb` para la validación cruzada, la
   validación externa y las ablaciones.

## Cómo citar

Si usas este código, cita el artículo y este repositorio (ver `CITATION.cff`).
Cita también el artículo de datos de CAUEEG y el dataset de OpenNeuro.

## Licencia

Publicado bajo la Licencia MIT (`LICENSE`). Los conjuntos de datos mantienen sus
propias licencias y condiciones de acceso.
