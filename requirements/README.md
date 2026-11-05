# Requirements

| File | Used by | Install |
|---|---|---|
| `analysis.txt` | `04_temporal_smoothing`, `05_evaluation_and_figures` | `pip install -r requirements/analysis.txt` |
| `preprocessing.txt` | `00_prepare_2d_inputs` | `pip install -r requirements/preprocessing.txt` |

Notebooks `01_tcpformer_inference`, `02_posemamba_inference` and `03_mvgformer_inference` have no requirements file on purpose:
each one installs its own dependencies in its setup section (the PyTorch build, the compiled CUDA operators and the pinned
packages of each original repository depend on the runtime), so they should be run in Google Colab, the environment of the paper
(Python 3.12, PyTorch 2.10, CUDA 12.8, Tesla T4).

The versions in the files above are lower bounds, not an exact freeze. The reference environment is the Colab runtime described in the main README.
