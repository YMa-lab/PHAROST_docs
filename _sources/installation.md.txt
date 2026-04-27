# Installation

## 1. Clone the repository

```bash
git clone https://github.com/YMa-lab/PHAROST.git
cd PHAROST
```

## 2. Create conda environment

```bash
conda env create -f environment.yml
conda activate PHAROST
```

## 3. Install PyTorch Geometric

```bash
pip install torch-scatter torch-sparse torch-cluster torch-spline-conv torch-geometric \
  -f https://data.pyg.org/whl/torch-2.8.0+cu128.html
```

## 4. (Optional) Jupyter Notebook

```bash
conda install -c anaconda ipykernel
python -m ipykernel install --user --name PHAROST --display-name "Python (PHAROST)"
```

## 5. Verify installation of pytorch packages

```python
import torch
import scanpy as sc
import torch_geometric

print("Installation successful!")
```
