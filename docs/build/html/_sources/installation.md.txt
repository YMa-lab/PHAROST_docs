## Environment Setup

Make sure you have [Conda](https://docs.conda.io/en/latest/) installed. Then clone the repository and set up the environment:

```bash
git clone https://github.com/YMa-lab/PHAROST.git
cd PHAROST

# Create the conda environment
conda env create -f environment.yml

# Activate the environment
conda activate PHAROST_env
```

***Install PyTorch Geometric*** 

The PyG companion libraries are already declared in `environment.yml` and installed by `conda env create`. If you ever need to reinstall them manually:

```bash
pip install torch-scatter torch-sparse torch-cluster torch-spline-conv torch-geometric \
  -f https://data.pyg.org/whl/torch-2.8.0+cu128.html
```

***Version Compatibility***

`torch_scatter`, `torch_sparse`, `torch_cluster`, and `torch_spline_conv` ship as
precompiled C extensions that must match the installed PyTorch ABI exactly.
The pinned versions in `environment.yml` target **PyTorch 2.8 + CUDA 12.8** (note the `+pt28cu128` suffix on the wheel filenames).

If `torch` is later upgraded (e.g., to 2.10), `import torch_geometric` will segfault with no helpful error message. To verify versions are aligned:

```bash
python -c "import torch; print(torch.__version__, torch.version.cuda)"
pip list | grep -E "^torch_(scatter|sparse|cluster|spline)"
# Expect: torch 2.8.x / cuda 12.8 / companion libs all '+pt28cu128'
```

If they don't match, either downgrade torch back to 2.8.0:

```bash
pip install torch==2.8.0 torchvision torchaudio \
  --index-url https://download.pytorch.org/whl/cu128
```

or reinstall the PyG companion libs against your current torch version using the matching index from <https://data.pyg.org/whl/>.

***Optional: Jupyter Notebook Support***

If you plan to use Jupyter notebooks:

```bash
conda install -c anaconda ipykernel
python -m ipykernel install --user --name PHAROST_env
```
