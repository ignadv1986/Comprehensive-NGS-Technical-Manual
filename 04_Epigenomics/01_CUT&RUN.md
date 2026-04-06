# CUT&RUN

CUT&RUN (Cleavage Under Targets and Release Using Nuclease) is an "in-situ" mapping strategy used to identify the binding sites of DNA-associated proteins, such as transcription factors and histone modifications. Unlike traditional ChIP-seq, which relies on random sonication of cross-linked chromatin, CUT&RUN utilizes a targeted approach within the cellular environment. Cells or nuclei are permeabilized, allowing a specific antibody to bind its target protein. A Protein A-Protein G-Micrococcal Nuclease **(pAG-MNase)** fusion protein then docks onto the antibody. Upon the addition of calcium, the nuclease is activated to precisely cleave the DNA only in the immediate vicinity of the protein-DNA complex.

These small, targeted fragments diffuse out of the nuclei and are collected for sequencing. This method results in extremely low background noise, requires significantly fewer cells (down to 100 cells or fewer in some protocols), and provides high-resolution mapping of protein-DNA interactions. Because the DNA is cleaved specifically at the site of interest, the resulting sequencing data is exceptionally clean, making it a "gold standard" for researchers working with limited sample material or requiring high-precision binding data.

## Fragment Size

While in other techniques, like RNA-seq, the size of the fragments is controlled during library prep, in CUT&RUN, fragments are generated during the protocol by the MNase. Because the nuclease cleaves specifically where the antibody is bound, the length of the resulting DNA is highly informative of the protein’s identity and its "footprint" on the genome. Generally, it is accepted that fragments <120 bp correspond to those bound by transcription factor, while 150 bp and 300 bp correspond to mono- and di-nuclesomes, respectively, indicating that the protein of interest is a histone or a histone binding protein. Fragments of 500 bp or above are usually considered background. Therefore, the SPRI step of the library prep is highly dependent on what is known about the protein of interest:

| Type of protein | Size (after adapter binding) | SPRI ratio |
|-----------------|------------------------------|------------|
| Transcription factor | 140-240 bp | 1x-1.2x |
| Histone-binding factor | ~270 bp | 0.8×-0.9× |

if the protein is a TF, then the smaller fragments are , by using a double-sided size selection: first, a low ratio (~0.6x) is used to remove the big, uninformative pieces of DNA. The beads are discarded and the supernatant is kept. Then a high ratio (1.8x - 2.0x) is be used to make sure that the smaller fragments (<120 bp) are not lost in the process. These will be bound to the beads, so in this step the supernatan is discarded.
There are many tools to calculate the fragment size distribution, but [bedtools](https://bedtools.readthedocs.io/en/latest/) or the **bamPEFragmentSize** of **[deepTools](https://deeptools.readthedocs.io/en/latest/) are usually the go-to options.

## The IgG Control

Every CUT&RUN experiment must include an IgG (non-specific antibody) control. The IgG sample should have very few mapped reads and a flat, noisy profile in the genome browser. If the IgG condition looks similar to the target antibody (same peaks, same signal), the enrichment is likely just background noise or open chromatin artifacts.

## Spike-in Normalization & Scaling Factor

Because CUT&RUN has such a low background signal, traditional normalization methods (like total read count) can be misleading. It is critical to normalize using spike-in DNA (usually E. coli or yeast DNA added during the protocol). For a deeper dive into the theory behind this, see the section on [Spike-in Correction](../02_Mapping_&_Alignment/03_post-alignment_processing.md) of this repository.

In CUT&RUN, we normalize technical variations by calculating a **Scaling Factor** for each sample:

Scaling Factor = max spike-in reads across all samples / sample spike-in reads

​This normalizes technical variations between samples, since the apike-in  DNA goes through the same processes as the samples themselves. If spike-in fails for a sample (very low counts), scaling may produce extreme inflation and should be excluded.



