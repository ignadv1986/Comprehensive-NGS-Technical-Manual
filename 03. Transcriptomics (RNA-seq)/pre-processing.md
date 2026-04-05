# RNA-seq

RNA-seq (RNA Sequencing) is the standard method for analyzing the **transcriptome**—the complete set of RNA transcripts in a cell or population of cells at a specific moment. Unlike the genome, which is relatively static, the transcriptome is highly dynamic, changing in response to environmental factors, drugs, or disease states.

The primary goal of RNA-seq is typically **Differential Expression Analysis**: identifying which genes are turned "up" or "down" between two conditions (e.g., Healthy vs. Diseased).

## RNA Quality Check: RNA Integrity Number (RIN)

Before any library preparation begins, the quality of the total RNA extracted from the cells must be assessed. The gold standard for this is the **RIN (RNA Integrity Number)**.

The RIN is a standardized numerical value from 1 (completely degraded) to 10 (perfectly intact). It was developed by Agilent Technologies to remove the subjectivity of a scientist "looking" at a gel and guessing the quality. This number is provided by automated electrophoresis devices, such as the bioanalyzer or the TapeStation.
The calculation is based on the entire electrophoretic trace of the RNA sample, but it focuses on three main features:

- **The 28S and 18S peaks:** In eukaryotes, these are the two largest ribosomal RNA species. In a healthy cell, the 28S:18S ratio should be approximately 2.0. As RNA degrades, these peaks shrink.
- **The degradation window:** This is the area between the 5S/small RNA peak and the 18S peak. As the long rRNA molecules break into smaller pieces, this valley starts to fill up with noise.
- **The Pre-Region:** High-quality RNA has a very clean baseline before the 18S peak. High signal here indicates significant fragmentation.

## RNA Enrichment Strategies

Total RNA is dominated by **ribosomal RNA (rRNA)**, which accounts for >90% of the molecules in a cell but provides almost no useful biological information for most studies, so an enrichment step where the RNA of interest is selected is required. The goal of the RNA-seq experiment determines the enrichment strategy; in the most common approaches, the analysis focuses on fully-mature mRNA. In this scenario, mRNAs are enriched through the use of **oligo(dT)-coated magnetics beads**, taking advantage of the fact that most mature eukaryotic mRNAs are polyadenylated, but rRNAs are not.

However, sometimes the analysis needs to include **non-polyadenylated RNAs**, such as prokaryotic RNA, histone RNA, and some long non-coding RNAs (lncRNAs). In these cases, the use of oligo(dT) would result in the loss of not only the rRNA, but also of the RNA of interest, so a **depletion** strategy is used instead: biotinylated DNA probes that are perfectly complementary to the 18S, 28S, 5S, and 5.8S ribosomal RNA sequences are added. These probes bind to the rRNA, and then Streptavidin beads are used to pull the rRNA-probe complexes out of the solution.
Additionally, in degraded or FFPE samples the poly-A tail is often separated from the rest of the gene, so a ribo-depletion step is also required. If oligo(dT) are used, they will bind to the fragmented poly-A tails, and the nanodrop would show a normal concentration of RNA, but the library will have almost no data.

It is important to note that the depletion approach is more expensive and requires higher quality input RNA. Because the mRNA is not concentrated, a deeper sequencing is needed to get the same level of gene coverage.

**Note:** even with standard library prep, checking for overrepresented sequences (like rRNA) and filtering them with tools such as SortMeRNA or BBduk in case they appear as overrepresented sequences in the fastQ step is a good safeguard to improve mapping efficiency and quantification accuracy.

## RNA fragmentation

As most NGS protocols, RNA-seq starts with a fragmentation step, to generate smaller fragments that can be processed by the sequencer after cDNA generation. The goal for most Illumina libraries is a fragment size of ~200–300 bp However, because RNA is single-stranded and much more fragile than DNA, sonication is not recommended. It generates local heat and cavitation that can damage the RNA, and it might contain traces of RNase. The go-to method for RNA is **chemical fragmentation**, based on a combination of heat (94°C) and a buffer containing bivalent metal ions (like Zn or Mg). At high temperatures, the metal ions act as Lewis acids. They coordinate with the oxygen atoms in the phosphodiester backbone of the RNA, making the backbone susceptible to hydrolysis (breaking the bond using a water molecule).

| RIN | Quality | Strategy | Fragmentation | Note |
|-----|---------|----------|---------------|------|
| 8-10 | Excellent | Poly-A/Ribo-depletion | Standard heat-Mg | Ideal for high-coverage mRNA-seq |
| 6-7 | Acceptable | Recommended ribo-depletion | Standard heat-Mg | Watch for "3' bias" if using Poly-A selection |
| 2-5 | Highly degraded | Ribosomal depletion mandatory | Skip or heavily reduce | Use DV200 to assess if sample is even sequenceable |

DV200: percentage of RNA fragments that are longer than 200 nucleotides. If the DV200 is >30%, the sample is usually "salvageable" for a specialized Ribo-depletion library.

## cDNA Generation

The most critical challenge of RNA-seq is that Illumina sequencers (and most other platforms) cannot sequence RNA directly; they require a stable, double-stranded DNA template. Therefore, the first step in any RNA-seq protocol is converting RNA into complementary DNA (cDNA). Since the RNA is fragmented in a previous step, the poly-A can no longer be used to prime the reverse transcriptase (RT) activity. The standard priming method consists on the use of **random hexameres**, 6-nucleotide random sequences that bind all along the RNA fragments, allowing the reverse transcriptase to initiate synthesis regardless of the original 3' or 5' position.

After the initial synthesis by the RT, an RNA-DNA hybrid is formed. 
