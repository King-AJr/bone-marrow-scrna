# Single-Cell RNA-Seq Analysis of Bone Marrow Sample Using Standard Scanpy Workflow

This project performs a full exploratory analysis of **bone marrow single-cell RNA sequencing (scRNA-seq)** data using **Scanpy**, **AnnData**, and **Decoupler**, covering the key stages of a typical scRNA-seq pipeline: data loading, QC, normalization, dimensionality reduction, clustering, and biological interpretation.

## 📁 Files in this Project

* **`bone-marrow-sc-rna.ipynb`** — the main notebook that contains all code, plots, and commentary.
* **`bone_marrow.h5ad`** — the starting bone marrow dataset (cells × genes).
* (Optionally) **`result.txt`** — a mapping file from Ensembl gene IDs to gene names (only needed if converting IDs / gene names).

---

## ✅ What I Did: Step-by-Step

### 1. Load the data

* I loaded the `.h5ad` file using Scanpy / AnnData.
* The dataset contained ~14,783 cells and ~17,374 genes.
* I inspected both the cell metadata (`obs`) and gene metadata (`var`) to understand what information was present.

### 2. Clean up names

* To avoid any problems later, I made sure that **all gene names and cell barcodes** are unique by running `var_names_make_unique()` and `obs_names_make_unique()`.
* This ensures downstream functions don’t fail because of duplicate row/column names.

### 3. Quality Control (QC) examinations

* I attempted to mark mitochondrial (MT), ribosomal (RPS / RPL), and hemoglobin (HB) genes by checking gene names.
* In this dataset, gene identifiers were mostly Ensembl IDs — so those flags remained zero (no matching genes).
* Then, I looked at distributions of the number of genes detected per cell and total UMI counts:

  * Most cells expressed ~1,000–2,000 genes, suggesting decent complexity per cell.
  * There was a tail of cells expressing up to ~5,000 genes — these could be doublets or particularly active cells.
  * UMI counts per cell broadly ranged around 5,000–15,000, with some cells having much higher counts — again, possibly doublets or very active cells.

### 4. Doublet detection & normalization

* I ran a doublet detection algorithm (Scrublet) to flag potentially problematic cells.
* I saved a copy of the raw counts, normalized the data to equal total counts per cell, and applied log-transformation (`log1p`) to stabilize variance across cells.

### 5. Feature selection — Highly Variable Genes (HVGs)

* I selected the top 1,000 most variable genes across all cells — these are likely to drive meaningful biological variation (lineage differences, activation states, etc.).
* These HVGs were used for dimensionality reduction and clustering steps.

### 6. Dimensionality reduction & visualization

* I performed PCA (principal component analysis) to compress the data from thousands of genes down to principal components that capture major variation.
* Then I built a neighbor graph among cells based on PCA — effectively representing which cells are transcriptionally similar.
* I ran UMAP to embed cells in 2D for visualization. This gave an overview of the cellular landscape in the bone marrow sample.

### 7. Clustering

* Using the neighbor graph, I applied the Leiden community detection algorithm at multiple resolutions. This produced clusters representing putative biological groups (cell types or states).

### 8. Cell-type annotation using marker signatures (Decoupler + PanglaoDB)

* I fetched canonical marker gene sets from PanglaoDB.
* I re-indexed the gene names in `adata.var` to use the gene symbol column for compatibility.
* I ran the `dc.mt.ulm()` method to compute a “cell-type activity score” for each cell based on the marker sets.
* I converted the resulting matrix (`score_ulm`) into a DataFrame and computed the average score per cluster (according to Leiden clustering).
* For each cluster, I assigned the cell type corresponding to the top-scoring signature.
* This resulted in a cluster-to-cell-type mapping such as:

  ```text
  '0': 'Neutrophils',
  '1': 'Gamma delta T cells',
  '2': 'Nuocytes',
  '3': 'NK cells',
  '4': 'Gamma delta T cells',
  '5': 'Nuocytes',
  '6': 'B cells naive',
  '7': 'Platelets',
  '8': 'Plasma cells',
  '9': 'Monocytes'
  ```
* I plotted these cell-type assignments on UMAP — giving a first-pass “annotated” bone marrow cellular landscape.

---

## 🧬 Biological Interpretation (So Far)

From this analysis, I reconstructed a plausible bone marrow cellular landscape comprising:

* **Myeloid lineages**: Neutrophils, Monocytes, Macrophages
* **Lymphoid lineages**: NK cells, γδ T cells, naïve αβ T cells
* **B-cell lineage**: Naïve B → B cells → Plasma cells
* **Platelets / megakaryocyte lineage** (or platelet precursors)

The UMAP and clustering suggest not only distinct mature cell types, but also possible **developmental trajectories** (e.g. B-cell maturation, monocyte → macrophage lineage).
This structure aligns well with known bone marrow biology.

---

## 1. What cell types did I identify?

From the UMAP and marker-based annotation, I identified the following cell types: Neutrophils, Macrophages, Monocytes, Gamma delta T cells (γδ T cells), NK cells, Naive CD4⁺ T cells, Naive B cells, Mature/activated B cells, Plasma cells, and Platelets.

Here’s the text with **bullets** for question 2 (the biological roles) while keeping the rest as you wrote it:

---

## 2. Biological role of each identified cell type

* **Neutrophils** — These are rapid-response innate immune cells that rush to sites of infection or tissue damage, engulf and kill microbes by phagocytosis, and contribute to early inflammatory responses. 
* **Macrophages** — Derived from monocytes, macrophages are long-lived phagocytes that ingest pathogens, clear debris and dead cells, recycle damaged tissue, present antigens to T cells to trigger adaptive immune responses, and release cytokines to coordinate inflammation or tissue repair. 
* **Monocytes** — These circulate in blood and can migrate into tissues where they differentiate into macrophages (or dendritic cells), thereby acting as a reservoir of phagocytic and antigen-presenting cells that contribute to immune surveillance and response.
* **γδ T cells (Gamma delta T cells)** — As unconventional T lymphocytes, γδ T cells can respond quickly to stress, infection, or tissue damage, bridging innate and adaptive immunity, often recognizing antigens in a non-MHC-restricted manner and contributing to early immune defense.
* **NK cells (Natural Killer cells)** — These are cytotoxic lymphocytes of the innate immune system that can detect and kill virus-infected or transformed (e.g. cancer) cells without prior sensitization, through release of cytotoxic granules, and also secrete cytokines to modulate immune responses. 
* **Naive CD4⁺ T cells** — These are helper T lymphocytes that have not yet encountered their specific antigen; upon activation by antigen-presenting cells, they proliferate and differentiate into effector subsets that coordinate adaptive immune responses, including helping B cells and cytotoxic cells. 
* **Naive B cells** — Antigen-inexperienced B lymphocytes that circulate or reside in lymphoid organs; upon encountering antigen (and receiving necessary helper signals), they can activate, proliferate, and differentiate into antibody-producing cells or memory B cells. 
* **Mature / Activated B cells** — Once antigen-experienced, these B cells participate in adaptive immunity by presenting antigen (to T cells), undergoing class-switching, proliferating, and eventually differentiating into plasma cells that secrete antibodies.
* **Plasma cells** — Terminally differentiated B cells that secrete large quantities of antibodies (immunoglobulins) specific to encountered antigens; these antibodies can neutralize pathogens, opsonize them for phagocytosis, or trigger complement-mediated destruction.
* **Platelets** — Small, anucleate fragments derived from megakaryocytes, primarily involved in hemostasis by forming clots at sites of vascular injury, but they can also influence immune responses by interacting with leukocytes and modulating inflammation or immune cell recruitment.


## 3. Is the tissue source really bone marrow? Justify your answer

No, the data argue strongly that the tissue is not genuine bone marrow, or at least not a representative healthy bone-marrow aspirate. In a normal bone marrow sample, there should be a complex mixture of stem cells, progenitors, and various maturing cell lineages, not just mature immune cells. Specifically, we expect to see pluripotent hematopoietic stem cells (HSCs), progenitor cells, erythroid precursors (for red blood cells), megakaryocytes or their precursors (which give rise to platelets), and other lineages, in addition to mature leukocytes and platelets. By contrast, our data show only mature leukocytes (neutrophils, various lymphoid cells such as γδ T, NK, B cells, plasma cells, monocytes/macrophages) and platelets, without any sign of stem/progenitor cells, erythroid cells, megakaryocyte lineage, or stromal/niche-associated cells, that stark absence of marrow-specific lineages is incompatible with a representative bone marrow sample. Therefore, based on the relative cell-type composition, that is what should be present in a bone marrow sample versus what is present, this does not match a typical healthy marrow.

## 4. Based on the relative abundance of cell types, is the patient likely healthy or infected/activated?

The composition is much more consistent with an activated or immune-active peripheral blood sample rather than healthy bone marrow, which suggests the patient is likely in a state of immune activation (e.g. infection or inflammation). Peripheral blood circulation normally contains mature leukocytes (neutrophils, monocytes, NK/T/B lymphocytes, etc.) and platelets under basal or activated conditions. In immune activation (such as infection or inflammation), innate and adaptive immune cells, including lymphocytes like NK cells and possibly unconventional subsets (e.g. γδ T cells), can expand or be enriched in blood. While I don’t have a reference that maps exactly the combination I observe (γδ T / NK / plasma cell + neutrophil/monocyte + platelets) to a defined clinical state, it is biologically plausible that “activated peripheral blood” could show enrichment for such circulating immune populations. By contrast, a healthy bone marrow sample should show not only mature cells but also ongoing hematopoiesis: progenitors, precursors, maturing erythroid, myeloid, and megakaryocytic lineages, and other elements of the marrow microenvironment. Because none of those are present in our data, there is no evidence for active multilineage hematopoiesis or normal bone-marrow architecture, which argues strongly against the sample being a healthy bone marrow. Given the exclusive presence of mature circulating-cell types and platelets, the most parsimonious interpretation is that I have a **peripheral blood sample, likely under immune activation** (infection, inflammation, or other immune challenge), rather than a healthy bone marrow sample.

---
