# Tutorial

This tutorial demonstrates how to run PHAROST on spatial transcriptomics data.

## 1. Prepare data

```python
import scanpy as sc

adata = sc.read_h5ad("path/to/data.h5ad")
```

## 2. Load pretrained model

```python
from pharost.model import PHAROST

model = PHAROST.load("path/to/pretrained_model.pt")
```

## 3. Run prediction

```python
pred = model.predict(adata)
```

## 4. Visualization

```python
import scanpy as sc

adata.obs["drug_response"] = pred
sc.pl.spatial(adata, color="drug_response")
```

## 5. Custom training (optional)

```python
model = PHAROST()
model.train(train_data)
```

## Notes

- GPU is recommended for large datasets
- Input data should be normalized
