# PHAROST API Reference

Public API surface of the `pharost` package. Import as:

```python
import pharost
```

The package exposes three layers:

- **Top-level** (`pharost.*`): training, inference, model loading.
- **`pharost.analysis.*`**: downstream analyses on predicted drug-response scores.
- **`pharost.model.*`**: lower-level neural network components (for advanced use).

---
## Model Modules

### `pharost.train`

```python
pharost.train(
    p_bulk_gene_exp,
    p_bulk_label,
    p_adata,
    out_dir,
    n_epochs,
    batch_size,
    seed=42,
    device='cuda',
    LR=5e-5,
    GAT_hidden_dim=(512, 64),
    lmmd_weight=0.3,
    coral_weight=0.7,
    spatial_preprocess=False,
    p_cell_info=None,
    p_sc_label=None,
)
```

End-to-end training. Loads bulk + spatial data, trains the GAT-MLP transfer
model with LMMD + CORAL domain-adaptation losses, and saves predicted
probabilities to `out_dir`.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `p_bulk_gene_exp` | `str` | Path to bulk RNA-Seq expression CSV (cells × genes). |
| `p_bulk_label` | `str` | Path to bulk drug-response label CSV. |
| `p_adata` | `str` | Path to spatial AnnData (`.h5` or `.h5ad`). |
| `out_dir` | `str` | Output directory; predictions and per-run log written here. |
| `n_epochs` | `int` | Number of training epochs. |
| `batch_size` | `int` | Number of METIS partitions for the spatial graph. |
| `seed` | `int` | `default : 42`, Random seed for full reproducibility. |
| `device` | `str` | `'cuda'` or `'cpu'`. |
| `LR` | `float` | `default : 5e-5`,AdamW learning rate. |
| `GAT_hidden_dim` | `tuple[int, int]` |`default: (512, 64)`, GAT hidden dimensions `(num_hidden, out_dim)`. |
| `lmmd_weight` | `float` | `default: 0.3`, LMMD loss weight. |
| `coral_weight` | `float` | `default: 0.7`, CORAL loss weight. |
| `spatial_preprocess` | `bool` | If True, run scanpy normalize/log/scale on spatial data. |
| `p_cell_info` | `str`, optional | CSV with `x_centroid`/`y_centroid` columns; used if `adata.obsm['spatial']` is missing. |
| `p_sc_label` | `str`, optional | scRNA-seq label path (passed to inner trainer). |

**Returns**: trained `TransferNN` model.

**Outputs**: writes the following to `out_dir`:
- `pharost_weights_adapted.pth` — pickled trained model.
- `predicted_probabilities.csv` — spatial-cell predicted probabilities.
- `predicted_probabilities_bulk.csv` — bulk predicted probabilities.

---

### `pharost.load_model`

```python
pharost.load_model(path, device='cuda')
```

Load a saved PHAROST model from disk.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `path` | `str` | Path to a saved `.pth` file produced by `pharost.train()`. |
| `device` | `str` | Device to map the model onto. |

**Returns**: `TransferNN` model.

---

### `pharost.predict`

```python
pharost.predict(
    model,
    adata,
    batch_size,
    device='cuda',
    spatial_preprocess=False,
    spa_identity=False,
)
```

Run inference on spatial cells using a trained model.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `model` | `TransferNN` | Model from `pharost.train()` or `pharost.load_model()`. |
| `adata` | `AnnData` | Spatial transcriptomics object with `adata.obsm['spatial']`. |
| `batch_size` | `int` | Number of METIS partitions, same as in the training. |
| `device` | `str` | `'cuda'` or `'cpu'`. |
| `spatial_preprocess` | `bool` | Apply scanpy preprocessing before inference. |
| `spa_identity` | `bool` | Use identity matrix for spatial graph. |

**Returns**: `list[np.ndarray]` — per-cell predicted probability vectors,
ordered to match `adata.obs`.

---
## Analysis Modules `pharost.analysis`

Downstream analyses. All functions assume `adata.obs[drug]` already contains
predicted probabilities (use `load_response_prediction` to populate them).

### `pharost.analysis.load_response_prediction`

```python
pharost.analysis.load_response_prediction(
    adata,
    drugs,
    path_template,
    add_label=False,
    label_threshold=0.5,
)
```

Load per-drug prediction CSVs into `adata.obs`.

**Parameters**

| Parameter | Type | Description |
|---|---|---|
| `adata` | `AnnData \| str` | AnnData object, or path to `.h5ad` file. |
| `drugs` | `list[str]` | Drug names to load. |
| `path_template` | `str \| callable` | Per-drug path: format string with `{drug}` placeholder, or `callable(drug) -> str`. |
| `add_label` | `bool` | If True, also write `adata.obs[f"{drug}_label"]` as `"Sensitive"`/`"Resistant"`. |
| `label_threshold` | `float` | Threshold for the Sensitive/Resistant cutoff. |

**Returns**: `AnnData` (in-place modified, also returned).

**Example**

```python
adata = pharost.analysis.load_response_prediction(
    adata,
    drugs=['LAPATINIB', 'AFATINIB'],
    path_template=lambda d: f'BC_result/{d}/predicted_probabilities.csv',
)
```

---

### `pharost.analysis.plot_response_celltype_prop`

```python
pharost.analysis.plot_response_celltype_prop(
    adata,
    target_drugs,
    cell_type_col,
    save=False,
    file_format='pdf',
    sample_id=None,
    save_dir='Drug_Celltype',
    palette='tab10',
)
```

Bar plot of the proportion of "sensitive" cells (probability > 0.5) in each
cell type, faceted by drug. Cell types with fewer than 5 cells are dropped.

---

### `pharost.analysis.drug_celltype_jsd`

```python
pharost.analysis.drug_celltype_jsd(
    adata,
    drug,
    cell_type_col='celltype',
    cmap=None,
    output_dir='Drug_Celltype_JSD',
    file_format='pdf',
    dpi=300,
    bins=50,
    hist_range=(0, 1),
    min_cells=10,
    pseudocount=1e-8,
    row_cluster=True,
    col_cluster=True,
    figsize=(6, 6),
    vmax=None,
    annot=False,
    verbose=False,
)
```

For one drug: histogram-based **Jensen–Shannon divergence** between
cell-type-specific response distributions, drawn as a clustermap.

**Returns**: `pd.DataFrame` — square JSD matrix indexed by cell type, or
`None` if fewer than 2 eligible cell types.

---

### `pharost.analysis.drug_drug_jsd_grid`

```python
pharost.analysis.drug_drug_jsd_grid(
    adata,
    drug_list,
    celltypes,
    cell_type_col='celltype',
    cmap=None,
    output_path='Drug_Drug_JSD/all_celltypes_grid_jsd_hist.pdf',
    dpi=300,
    bins=50,
    hist_range=(0, 1),
    min_cells=10,
    pseudocount=1e-8,
    max_cols=3,
    figsize_per_plot=4,
    annot=False,
    vmin=0,
    vmax=0.45,
    verbose=False,
)
```

For each cell type: pairwise drug–drug **Jensen–Shannon divergence** within
that cell type, plotted as a grid of heatmaps.

**Returns**: `dict[str, pd.DataFrame]` — cell-type → drug-drug JSD matrix.

---

### `pharost.analysis.drug_gene_correlation`

```python
pharost.analysis.drug_gene_correlation(
    adata,
    target_drugs,
    n_top_genes=15,
    cmap=None,
    plot=True,
    annot=False,
    save=False,
    save_dir='Gene_Drug_Corr',
    file_format='png',
    verbose=False,
)
```

Spearman correlation between gene expression and drug response across all
cells. Computes per-drug top-`n_top_genes` and plots a clustered heatmap of
the union.

**Returns**: long-form `pd.DataFrame` with columns `Gene`, `Drug`, `Correlation`.

---

### `pharost.analysis.plot_single_drug_gene_correlation`

```python
pharost.analysis.plot_single_drug_gene_correlation(
    adata,
    gene,
    drug,
    save=False,
    cmap=None,
    file_format='png',
    save_dir='Gene_Drug_Corr',
    verbose=False,
)
```

2×2 Spearman correlation heatmap between one gene and one drug
(predicted-class label thresholded at 0.5).

---

### `pharost.analysis.bivariate_gene_drug`

```python
pharost.analysis.bivariate_gene_drug(
    adata,
    gene,
    drug,
    bins=3,
    spot_size=0.01,
    dpi=500,
    save=False,
    save_dir='Gene_Drug_Corr',
    file_format='png',
    verbose=False,
)
```

Bivariate spatial scatter: each cell colored by its (gene_expression,
drug_response) bin, with a 2D legend grid.

---

### `pharost.analysis.bivariate_gene_gene`

```python
pharost.analysis.bivariate_gene_gene(
    adata,
    gene1,
    gene2,
    bins=3,
    spot_size=0.01,
    dpi=500,
    save=False,
    save_dir='Gene_Drug_Corr',
    file_format='png',
    verbose=False,
)
```

Bivariate spatial scatter for two genes (same coloring scheme as
`bivariate_gene_drug`).

---

### `pharost.analysis.sensitive_resistant_DE`

```python
pharost.analysis.sensitive_resistant_DE(
    adata,
    drug,
    method='wilcoxon',
    save=False,
    plot=False,
    n_top_genes=5,
    save_dir='Sensitive_vs_Resistant_DE',
    format='png',
)
```

Differential expression between predicted-sensitive and predicted-resistant
cells for one drug, using `scanpy.tl.rank_genes_groups`.

---

### `pharost.analysis.gsea_analysis`

```python
pharost.analysis.gsea_analysis(
    adata,
    target_drugs,
    gene_set_path,
    n_top_gene_set=50,
    save_dir='Sensitive_vs_Resistant_DE',
    permutation_num=1000,
    threads=4,
    seed=0,
    skip_de_analysis=False,
    visualize=False,
    format='png',
    verbose=False,
)
```

GSEA on sensitive-vs-resistant rankings using `gseapy.prerank`. Set
`skip_de_analysis=True` to reuse a prior DE result.