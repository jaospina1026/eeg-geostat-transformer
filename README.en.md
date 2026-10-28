# Spatially Interpretable Transformer for EEG-Based Cognitive Decline Assessment

Code for the study *"Spatially Interpretable Transformer Learning for EEG-Based
Cognitive Decline Assessment Through Functional Geostatistical Biomarkers."*
The repository holds the two Colab notebooks behind the paper: the functional
geostatistics analysis that produces the spatial prior and the biomarkers, and
the PSD-functional Transformer that uses that prior as a spatial-attention bias.

## What the method does

Resting-state EEG is classified as *Normal* vs *Impaired* (MCI + dementia). Each
channel power spectral density (PSD) is represented as a penalized B-spline
expansion, so the model works on a functional descriptor rather than hand-picked
band powers. A pooled trace-variogram, estimated with great-circle distances
between electrodes and selected by the corrected Akaike information criterion
(AICc), yields a spatial-attention bias `B_spatial` that is added to the
attention scores. The same geostatistical analysis then serves as an
interpretability layer, relating attention to spatial coherence across the scalp
(Mantel test and channel-level concordance).

## Notebooks

| Notebook | Purpose |
|----------|---------|
| `notebooks/01_functional_geostatistics.ipynb` | Trace-variogram estimation, model selection (AICc), subject-level biomarkers, and the `B_spatial` matrix used by the Transformer. |
| `notebooks/02_psd_functional_transformer.ipynb` | PSD-functional Transformer with the spatial-attention bias, patient-grouped cross-validation, external validation, equivalence testing (TOST), and the ablation battery. Shipped with its cell outputs so the reported results are visible without rerunning. |

Both notebooks are written for Google Colab (they mount Google Drive and read the
dataset from it). They run unchanged on a local Jupyter environment once the
`BASE_DIR` path is pointed at a local copy of the data.

## Data

The study uses two datasets; neither is redistributed here.

- **CAUEEG** (970 subjects; 394 CN, 329 MCI, 247 DEM). Access is granted by the
  original authors upon request. After their review they provide the download
  link and password. The dataset, its collection, and its usage are described in
  the CAUEEG data paper (Kim et al.). Place the downloaded data where the
  notebooks expect it and edit `BASE_DIR` accordingly:

  ```python
  BASE_DIR = ".../caueeg-dataset"   # set to your local or Drive path
  ```

- **OpenNeuro ds004504** (n = 88), used for external validation. Openly available
  at https://openneuro.org/datasets/ds004504.

## Environment

Python 3.10+. Install the dependencies with:

```bash
pip install -r requirements.txt
```

On Colab these are preinstalled except `mne`, `mne-icalabel`, `autoreject`, and
`scikit-fda`, which the first notebook cells install. A GPU runtime is
recommended for the Transformer notebook.

## How to run

1. Request CAUEEG access and download the data; set `BASE_DIR`.
2. Run `01_functional_geostatistics.ipynb` to fit the variogram and build
   `B_spatial`; it caches its outputs to the path given in the notebook.
3. Run `02_psd_functional_transformer.ipynb` for cross-validation, external
   validation, and the ablations.


## Reusable modules

`src/` holds the paper's building blocks extracted from the notebooks:

- `src/geostatistics.py` — trace-variogram and model selection (AICc), the
  functional PSD descriptor (penalized B-splines), the subject-level biomarkers,
  and the montage/geometry helpers.
- `src/transformer.py` — the PSD-functional Transformer and its spatial-attention
  bias, the data pipeline, training and cross-validation, external validation,
  interpretability (Mantel test, attention-variogram concordance), and the
  ablation variants.

Each module keeps the function and class bodies verbatim; the final version of
any name the notebooks redefined while iterating is the one kept. End-to-end
execution (data paths, configuration, training) stays in the notebooks, which
some functions rely on for configuration constants.

## Citation

If you use this code, please cite the paper and this repository; see
`CITATION.cff`. Please also cite the CAUEEG data paper and the OpenNeuro dataset.

## License

Released under the MIT License (`LICENSE`). The datasets keep their own licenses
and access terms.
