# Pipeline & Computational Troubleshooting (ATAC-seq)

Even when sequencing quality, alignment, and standard QC metrics are within expected ranges, ATAC-seq results can still appear inconsistent or biologically implausible during downstream analysis. In these situations, the issue is rarely the raw data itself, but how reads are translated into signal.

This troubleshooting guide focuses on standard ATAC-seq pipelines aimed at differential accessibility analysis and (optionally) motif enrichment. In this context, Tn5 insertion offset correction (read shifting) is not a critical requirement for most downstream analyses. While it can improve resolution at base-pair level (e.g., transcription factor footprinting), most common issues in ATAC-seq data arise from library complexity, filtering, normalization, or experimental design rather than the absence of Tn5 shifting.

## TSS Enrichment Score

The TSS enrichment score is often the first indication that something is wrong despite otherwise acceptable QC metrics, as it directly reflects how well the signal has been processed and aligned to regulatory features.

A flat or weak TSS enrichment profile can be a consequence of the following:

- **BAM filtering:** BAM files from different samples need to be properly filtered (especially by removing reads mapping to chrM) to prevent accumulation of background reads that might mask enrichment at TSS.
- **Over-Digitization:** If the Tn5 concentration is too high for the number of nuclei, it will force its way into the edges of nucleosomes, leading to a broad plateau instead of a sharp spike in the TSS enrichment plot.
- **Low Non-Redundant Fraction (NFR):** If the TSS score is low and the duplication rate is high (low NRF), that is a signal of an under-sampled library. This often happens if the transposition was successful but the DNA recovery (PCR/clean-up) was poor, leading to a small amount of reads being repetaedly sequenced. Because the TSS score relies on a ratio of signal-to-background, a lack of unique diverse reads can actually make the TSS signal look artificially weak or statistically insignificant.

### Quick diagnostics guide

| Symptom | Likely Cause | What to Check|
| :--- | :--- | :--- |
| Flat or low TSS enrichment profile | BAM filtering issues | Check for presence of chrM reads, PCR duplicates, or suboptimal filtering strategy in the final BAM |
| Broad plateau instead of sharp spike | Over-transposition (Over-digitization) | Check Tn5-to-nuclei ratio. Excessive enzyme forces cuts into nucleosome edges, blurring the TSS signal |
| Weak enrichment + High duplication | Low Library Complexity (NRF) | Check the Non-Redundant Fraction. Signal is likely masked by under-sampling due to poor DNA recovery post-transposition |
| High variability between samples | Inconsistent Preprocessing | Verify that uniform filtering (e.g., same MAPQ thresholds and duplicate removal) was applied across the entire cohort |

## Early Visualization in Genome Browsers

After BAM file processing and in parallel to TSS enrichment score inspection, it is good practice to study the signal in a genome browser such as IGV before proceeding to peak calling and downstream analysis.

Unlike CUT&RUN, ATAC-seq does not produce discrete protein binding sites but rather a continuous accessibility landscape. Therefore, interpretation relies on recognizing characteristic patterns of open chromatin rather than isolated peaks. A clean BAM file derived from a good ATAC-seq experiment should show:

- Sharp enrichment at promoters and known regulatory regions
- Clear depletion in heterochromatic regions
- Consistent signal patterns across replicates
- Well-defined peaks with low background signal

**Note:** Even though BAM file inspection would theoretically be performed before coverage files generation, many pipelines skip this step due to the heavy computational power required to open and zoom in into BAM files, which are usually in the Gb range. However, if errors are spotted in downstream steps (coverage track generation, peak calling, etc), visualizing BAM files in a genome browser might clarify if the problem occured during the alignment process or in subsequent steps.

### Uniform signal across the whole genome

If signal appears broadly distributed with little contrast:

- **Presence of background reads:** BAM files not properly filtered are likely to show higher levels of background and therefore masked peaks.
- **Over-tagmentation:** Uncontrolled enzymatic activity can introduce widespread low-level accessibility, reducing contrast between open and closed chromatin.

### Lack of enrichment at expected regions

If promoters or known accessible loci show little to no signal:

- **Insufficient signal-to-noise ratio:** Poor library complexity or high background can obscure real accessibility.
- **Biologically real low accesibility:** The signal may truly be low in this condition, but it should be consistently low across all replicates if that is the case.
- **Lack of Tn5 offset correction:** This can slightly diffuse signal around cut sites, but on its own rarely explains a complete absence of enrichment.

### Signal present but poorly defined

If enrichment exists but lacks sharp peaks:

- **Over-tagmentation:** Excessive enzymatic activity can broaden accessible regions artificially.
- **Residual background reads:** Elevated background reduces contrast and makes enrichment regions appear less distinct.
- **Fragment size heterogeneity:** Mixed nucleosomal and subnucleosomal fragments can reduce signal clarity. While major abnormalities are usually visible in fragment analyzer profiles, more subtle imbalances often only become apparent after alignment and visualization in genome browsers.
  
### Enrichment in unexpected regions

If strong signal appears outside known regulatory elements:

- **Genome build mismatch:** The genome build used for visualization must match the one used in the mapping step, or the peaks will seem to appear in unexpected regions.
- **Incorrect filtering:** If not removed, reads may accumulate in repetitive or blacklist regions. These regions can produce convincing but non-reproducible peaks.
- **Multimapping reads retained:** Low-complexity regions may show artificial enrichment if not properly filtered.

### Quick diagnostics guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Uniform signal across genome (low contrast) | Background contamination or over-tagmentation | BAM filtering, chrM removal, fragment distribution |
| Lack of enrichment at promoters or known loci | Low signal-to-noise or biological effect | Library complexity, replicate consistency |
| Signal present but poorly defined | Background signal or fragment heterogeneity | Fragment size distribution, BAM quality |
| High signal in heterochromatic regions | Poor filtering or alignment artifacts | MAPQ filtering, blacklist regions |
| Inconsistent patterns between replicates | Technical variability | Replicate concordance, library quality |

## Coverage File Generation

After BAM processing, normalized coverage tracks (usually BigWig files generated with `deepTools bamCoverage`) should be inspected before peak calling. These tracks provide a direct view of chromatin accessibility and often reveal issues that are not apparent from alignment or QC metrics alone.

### Uniform signal across the whole genome

If coverage appears broadly distributed with little contrast between regions:

- **Incomplete filtering of reads:** Incomplete BAM filtering increases baseline signal and masks true accessibility.
- **Over-tagmentation:** Excessive enzymatic activity can generate widespread low-level accessibility signal.
- **Scaling issues:** Lack of proper normalization with CPM or RPKM may make high-depth samples appear uniformly elevated. This behavior is controlled by the `--normalizeUsing` parameter in bamCoverage, and inconsistent settings across samples can introduce artificial differences in signal intensity.

### Signal intensity differs strongly between samples

If one sample appears globally stronger or weaker than others:

- **Missing or inconsistent normalization:** All samples should be normalized using the same method, or library sizes and other factors can affect the scaling and lead to result misinterpretation.
- **chrM contamination:** High mitochondrial content can compress nuclear signal in affected samples, so it should be equally removed in all samples.

### Lack of enrichment at promoters or known regulatory regions

If expected accessible regions show weak or no signal:

- **Biologically real ow accesibility:** Biologically true low accesibility can occur due to specific treatments or vary between cell lines, but it should be consistent across replicates.
- **Missing Tn5 correction:** Signal becomes blurred and less concentrated at true cut sites, sometimes (but very rarely) making it difficult to detect enrichment at such sites.

### Signal present but poorly defined (blurred or wide peaks)

If enrichment exists but lacks sharp boundaries:

- **Incorrect bin size:** Large bins oversmooth signal; very small bins introduce noise. The default for deepTools bamCoverage is 50 bp, but sometimes this can prevent detection of narrow peak information. Bin size can be modified in bamCoverage with `--binSize` and 1-10 bp ranges can be used to detect these peaks 
- **Fragment handling issues:** Treating paired-end data as single-end reduces resolution.
- **Residual background reads:** Elevated background reduces contrast and sharpness.
- **Lack of Tn5 shifting:** This may contribute slightly to peak broadening, but is rarely the primary cause.

### Enrichment in unexpected regions

If strong signal appears outside known regulatory elements:

- **Genome build mismatch:** Coverage does not align with annotation or browser reference
- **Incomplete filtering:** Reads accumulating in blacklist or repetitive regions
- **Multimapping reads retained:** Artificial signal in low-complexity regions

### Quick diagnostics guide

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Uniform signal across genome (low contrast) | Incomplete filtering or over-tagmentation | BAM filtering (chrM, duplicates), fragment size distribution |
| Uniformly elevated signal in one sample | Missing or incorrect normalization | CPM/RPKM normalization, library size |
| Strong differences in signal intensity between samples | Inconsistent normalization or chrM contamination | Normalization method consistency, mitochondrial read fraction |
| Weak or no enrichment at promoters / regulatory regions | Low signal-to-noise or biological effect | Library complexity, replicate consistency |
| Signal present but blurred or poorly defined | Background signal or binning effects | `--binSize`, fragment distribution, BAM quality |
| Peaks overly broad | Oversmoothing or fragment composition | Bin size, fragment size profile |
| Noisy or fragmented signal | Very small bin size or low coverage | `--binSize`, sequencing depth |
| Enrichment in unexpected regions | Genome mismatch or poor filtering | Genome build, blacklist removal |
| Strong signal in repetitive regions | Multimapping reads retained | Alignment settings, MAPQ filtering |
| Coverage does not match BAM visualization | Inconsistent processing parameters | bamCoverage settings vs BAM preprocessing |

## Peak Calling

After alignment and preprocessing, MACS3 peak calling may still produce biologically inconsistent or unstable peak sets even when QC metrics look acceptable. In ATAC-seq, this is often due to mismatch between fragment representation, signal structure, and MACS3 assumptions, rather than the peak caller itself.

For ATAC-seq pipelines aimed at differential accessibility analysis, it is strongly recommended to run MACS3 in paired-end mode (`-f BAMPE`).
In this mode, MACS3 uses the full fragment span inferred from read pairs rather than modeling fragment size from single-end reads. This better reflects the biology of ATAC-seq data and typically results in more accurate and stable peak calls.

When inspecting the narrowPeak files generated in the peak calling step, a reliable peak set should show strong overlap with promoter regions, sharp peak boundaries, and consistency across replicates. Additionally, it should ressemble what can be seen in the coverage files.

### Peak set does not match coverage signal

- **Incorrect fragment handling:** Running MACS3 in single-end mode (`-f BAM`) forces fragment size estimation, which can artificially broaden peaks and distort signal structure.
- **Parameter choice:** Parameters commonly used when analysing single-end reads, such as `--nomodel`, `--shift` or `--extsize`, can artificially shift or stretch signal, leading to peaks that do not align with coverage tracks.

### Over-fragmented peak set

If peaks appear artificially split into many small regions:

- **Overly stringent significance threshold:** Very strict cutoffs (`--qvalue` too low; e.g., ≤ 0.001 depending on dataset quality) can break continuous accessible regions into multiple smaller peaks by only retaining the highest local signal.
- **Peak calling performed independently with borderline signal:** When signal is near the detection limit, small local fluctuations may pass the threshold in some regions but not others, leading to fragmented peak structure.

### Over-merged or excessively broad peaks

If peaks span large genomic regions with poorly defined boundaries:

- **Improper fragment handling (not using BAMPE):** Leads to inaccurate estimation of fragment size and broader peak calls.
- **Overly permissive significance threshold:** Permissive cutoffs  (`--qvalue` too high; e.g., ≥ 0.1) allow weak signal to be included, causing nearby regions of modest enrichment to merge into large peaks.
- **Broad peak mode used inappropriately:** The `--broad` mode of MACS3 is designed for histone marks, not typical ATAC-seq accessibility. Using it can artificially merge distinct accessible regions.

### Strong differences in peak sets between replicates

If replicate peak sets differ substantially despite similar coverage profiles:

- **Inconsistent parameter usage:** Differences in `--qvalue`, duplicate handling, or fragment interpretation (BAM vs BAMPE) can produce divergent peak sets.
- **Threshold sensitivity:** Peaks close to the cutoff may appear in one replicate but not another, especially in moderate or noisy datasets.

### Quick diagnostics guide 

| Symptom | Likely cause | What to check |
| :--- | :--- | :--- |
| Peaks do not match coverage tracks | Incorrect fragment handling or shifting/stretching | BAMPE mode, `--shift`, `--extsize`, `--nomodel` |
| Over-fragmented peaks | Excessive stringency | `--qvalue` (too low), peak distribution |
| Overly broad or merged peaks | Lenient thresholds or broad mode | `--qvalue` (too high), `--broad`, BAMPE mode |
| Large differences between replicates | Parameter inconsistency or threshold sensitivity | `--qvalue`, parameter consistency |





