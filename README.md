# Unsupervised discovery of tissue architechture with graphs (UTAG)

[![Biorxiv badge](https://zenodo.org/badge/doi/10.1101/2022.03.15.484534.svg)](https://doi.org/10.1101/2022.03.15.484534) ⬅️ read the preprint here
 
This package implements segmentation of multiplexed imaging data into microanatomical domains.
Multiplexed imaging data types are typically imaging mass cytometry (IMC), co-detection by indexing (CODEX), multiplexed ion beam imaging by time of flight (MIBI-TOF), cyclic immunofluorescence (CyCIF), and others.
The package also provides functions for the downstream analysis of the detected micro-anatomical structure.


## Getting Started

### Install from github

```bash
pip install git+https://github.com/ElementoLab/utag.git@main
```
Installation should take less than 10 seconds.

#### Requirements
There are no specific hardware requirements.

Software requirements:
- UTAG has only been tested on Linux machines.
- Python 3.7+ (tested on 3.8.2)
- Python packages (automatically installed by `pip`):
  - numpy
  - pandas
  - anndata
  - scanpy
  - parc
  - squidpy
  - scipy
  - matplotlib
  - tqdm
  - networkx
  - parmap
  - scikit-learn
Specific versions of Python packages have been pinned to the [setup.cfg](setup.cfg) file.

## Basic Usage Principles

The UTAG process can be run with a single function call `utag.utag`.
The input is a [AnnData](https://anndata.readthedocs.io/) object which should have the position of cells (typically centroids) in the `spatial` slot of `adata.obsm`.
The function will output domain classes for each cell stored in the `obs` slot of the returned AnnData object.


### Running an example/demo dataset

Please refer to the [notebook directory](documentation/), and to the notebook on [running UTAG on healthy lung data](https://github.com/ElementoLab/utag/blob/main/documentation/IMC%20Healthy%20Lung.ipynb) for a reproducible example.

### Running on your data

To run the method on a single image:
```python
from utag import utag
utag_results = utag(
    adata,
    slide_key=None,
    max_dist=20,
    normalization_mode='l1_norm',
    apply_clustering=True,
    clustering_method = 'leiden', 
    resolutions = [0.05, 0.1, 0.3]
)
```

To run the method on multiple images/slides in batch mode:
```python
from utag import utag
utag_results = utag(
    adata,
    slide_key="roi",
    max_dist=20,
    normalization_mode='l1_norm',
    apply_clustering=True,
    clustering_method = 'leiden', 
    resolutions = [0.05, 0.1, 0.3]
)
```

To visually inspect the results of the method:
```python
import scanpy as sc
for roi in utag_results.obs['roi'].unique():
    result = utag_results[utag_results.obs['roi'] == roi].copy()
    sc.pl.spatial(result, color = 'UTAG Label_leiden_0.1', spot_size = 10)
```

## Key Parameters

| Input Parameter | Description |
| ---------- |----------|
| `adata` | (`anndata.AnnData`) n_cells x n_features. `AnnData` of cells with spatial coordinates stored in `adata.obsm['spatial']` as `numpy.ndarray`. |
| `max_dist` | (`float`, default = 20.0) Threshold euclidean distance to determine whether a pair of cell is adjacent in graph structure. Recommended values are between 10 to 100 depending on magnification. |
| `slide_key` | (`str`, optional, default = 'Slide') Key required for running UTAG across multiple images. Unique image identifiers should be placed under `adata.obs`. Use `None` to run UTAG on a single slide. |
| `save_key` | (`str`, default = 'UTAG Label') Key to be added to adata object holding the UTAG clusters. Depending on the values of `clustering_method` and `resolutions`, the final keys will be of the form: {save_key}\_{method}\_{resolution}". |
| `normalization_mode` |  (`str`, default = 'l1_norm') Method to normalize adjacency matrix. 'l1_norm' will behave as mean-aggregation during message passing. Default is 'l1_norm'. Any other value will not perform normalization, leading to a sum-aggregation. |
| `apply_clustering` |  (bool, default = True) Whether to cluster the message passed matrix. |
| `clustering_method` |  (Sequence[str], default = ['leiden', 'parc']) Which clustering method(s) to use for clustering of the message passed matrix. |
| `resolutions` |  (Sequence[float], default = [0.05, 0.1, 0.3, 1.0]) Resolutions the methods in `clustering_method` should be run at. |
| `parallel` | Whether the message passing part of the method should be parallelized. This is done using the `parmap` package and the `multiprocessing` module from the standard library. |

For more detailed usage of the package and downstream analysis, please refer to [IMC Healthy Lung.ipynb](https://github.com/ElementoLab/utag/blob/main/documentation/IMC%20Healthy%20Lung.ipynb) in the documentation folder.
