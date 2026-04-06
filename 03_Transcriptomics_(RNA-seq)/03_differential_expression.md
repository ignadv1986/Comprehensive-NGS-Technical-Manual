# Differential Expression

## DESeq2

For the identification of differentially expressed genes, the R/Bioconductor package [DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html) is the industry standard. It works on a **DESeq2 object**, constructed from a count matrix (usually provided by featureCounts) with the number of reads, with the genes as rows and the samples as columns, and a metadata matrix containing sample information (replicate, condition…).

Example of a count matrix:

| Gene | Sample1 | Sample2 | Sample3 | Sample4 | Sample5 | Sample6 |
|------|---------|----------|---------|---------|----------|---------|
| GeneA | 123 | 98 | 112 | 304 | 350 | 335 |
| GeneB | 7 | 15 | 4 | 6 | 7 | 12 |
| GeneC | 300 | 250 | 289 | 150 | 123 | 98 |

Example of a metadata matrix 

| Sample | Condition | Replicate |
|--------|-----------|-----------|
| Sample1 | Control | 1 |
| Sample2 | Control | 2 |
| Sample3 | Control | 3 |
| Sample4 | Treated | 1 |
| Sample5 | Treated | 2 |
| Sample6 | Treated | 3 |

When analyzing differential expression, DESeq2 does the following three things:

- **Normalization:** It calculates a "scaling factor" for each sample. It doesn't just divide by the total number of reads (which is what TPM or RPKM does); it uses a median-of-ratios method. This ensures that a single, massively over-expressed gene doesn't skew the normalization for every other gene in the sample.
- **Dispersion estimation:** RNA-seq data is "overdispersed," meaning the variance grows faster than the mean. DESeq2 uses a **Negative Binomial distribution** to model this biological noise, "shrinking" the dispersion estimates of individual genes toward a global trend to make the analysis more robust against outliers.
- **Statistical testing:** It performs a statistical test (**Wald test**)to determine if the change in expression between the conditions is greater than what would be expected by random chance.

It outputs a table containing:

- **basemean:** The average expression across all samples.
- **log2FoldChange (LFC):** The magnitude of the change (e.g., a value of 1 means a 2-fold increase; a value of -1 represents a 2-fold decrease).
- **The adjusted p-value (padj):** Corrects for multiple testing, controlling for false discovery rate (FDR), using the Benjamini-Hochberg correction. Let´s say we have a p-value of 0.01. That means we have a 1% possibility of our result being a false positive. Padj drastically reduces this number, and in consequence a padj < 0.05 is considered the threshold for significance.

DESeq2 results are usually plotted as volcano plots, with the shrunk LFC (see below) in the x-axis, and the -log10(padj) on the y axis. This transformation turns tiny p-values into large positive numbers, placing the most significant genes at the top of the plot

## LFC shrinkage

For genes with low counts or high variability, LFC estimates can be unstable and exaggerated in magnitude due to noise. For example, a gene moving from 1 read to 5 reads represents a "5-fold increase" that is likely statistically meaningless. The R/Bioconductor package [apeglm](https://bioconductor.org/packages/release/bioc/html/apeglm.html) shrinks LFCs per gene, instead of using a fixed prior like other methods, to get more stable, interpretable, and realistic estimates of gene expression changes — reducing noise without erasing true biological effects.
