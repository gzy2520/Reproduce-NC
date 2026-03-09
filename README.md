# B-cell 多组学单细胞测序复现项目 (Nature Communications, 2021)

本项目旨在复现 B 细胞发育过程中的多组学数据分析流水线，涵盖从原始数据下载、CellRanger 定量到下游 Seurat 深度分析的全过程 。

**参考论文：** [Single-cell multi-omics reveals the genetic programs of B cell development (Nature Communications, 2021)](https://www.nature.com/articles/s41467-021-27232-5)

---

## 目录结构

项目通过 `00_config.env` 统一管理路径，建议保持以下结构 ：

```text
NC/
├── Bcell/                  # 数据存储主目录 
│   ├── fastq/              # 原始序列 (下设 RNA/ADT/HTO 子目录) 
│   ├── meta/               # 存放 libraries.csv 和 feature_reference.csv
│   ├── refdata/            # 10x 官方参考基因组 
│   └── output/             # CellRanger 运行结果输出
├── Ranalyze/               # R 语言下游分析目录
│   └── raw/                # 存放从 GEO 下载的矩阵数据 (GSE168158)
[cite_start]└── software/               # 软件安装目录 (cellranger-9.0.1) 

```

---

## 环境要求

1. **基础环境**：Linux 系统，安装有 `wget`, `fastp`, `tar`。
2. **CellRanger**：版本 9.0.1。
3. **R 语言**：需安装 `Seurat`, `SCTransform`, `patchwork`, `biomaRt`, `clusterProfiler`, `msigdbr` 等包。

---

## 运行指南

### 第一阶段：数据准备与上游处理 (Shell)

1. **下载数据与参考基因组**
执行脚本下载 FASTQ 文件（来自 ENA）及 mm10 参考基因组。下载后建议使用 `md5sum` 校验文件完整性。
```bash
bash 01_download.sh

```


2. **生成配置文件**
自动创建 CellRanger 运行所需的 `libraries.csv` 和特征定义文件。
```bash
bash 02_make_csv.sh

```


3. **运行 CellRanger 定量**
脚本会自动处理 FASTQ 命名规范（重命名为 `_S1_L001_R1_001` 格式）并启动 `cellranger count`。
```bash
bash 03_run_cellranger.sh

```



---

### 第二阶段：下游生物信息分析 (R)

本项目提供两种分析路径，请根据数据来源选择：

* **路径 A：处理 GEO 已发表矩阵 (推荐)**
如果直接使用 GEO ([GSE168158](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE168158)) 的原始计数矩阵：
```bash
Rscript Control_a.R

```


* **路径 B：处理自行运行的 CellRanger 结果**
如果使用第一阶段生成的 `filtered_feature_bc_matrix`：
```bash
Rscript Control.R

```


* **统一绘图与高级分析**
对处理好的 `bcell.rds` 进行细胞类型映射、分群高亮、差异基因火山图及 GSEA 分析。
```bash
Rscript clear.R

```



---

## 主要分析内容

* **细胞群鉴定**：涵盖从 Prepro B 到 Mature B 的 14 种亚群映射。
* **多维投影**：生成 UMAP、tSNE 及基于 Cell Cycle 的 PCA 投影。
* **多组学验证**：同时展示 RNA 水平与 ADT（抗体捕获）水平的特征表达。
* **功能富集**：针对 PreBCR 依赖阶段进行 Myc Targets 及代谢通路的 GSEA 分析。

---
