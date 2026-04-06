# CUT&RUN

CUT&RUN (Cleavage Under Targets and Release Using Nuclease) is an "in-situ" mapping strategy used to identify the binding sites of DNA-associated proteins, such as transcription factors and histone modifications. Unlike traditional ChIP-seq, which relies on random sonication of cross-linked chromatin, CUT&RUN utilizes a targeted approach within the cellular environment. Cells or nuclei are permeabilized, allowing a specific antibody to bind its target protein. A Protein A-Protein G-Micrococcal Nuclease **(pAG-MNase)** fusion protein then docks onto the antibody. Upon the addition of calcium, the nuclease is activated to precisely cleave the DNA only in the immediate vicinity of the protein-DNA complex.

These small, targeted fragments diffuse out of the nuclei and are collected for sequencing. This method results in extremely low background noise, requires significantly fewer cells (down to 100 cells or fewer in some protocols), and provides high-resolution mapping of protein-DNA interactions. Because the DNA is cleaved specifically at the site of interest, the resulting sequencing data is exceptionally clean, making it a "gold standard" for researchers working with limited sample material or requiring high-precision binding data.

## Fragment Size

While in other techniques, like RNA-seq, the size of the fragments is controlled during library prep, in CUT&RUN, fragments are generated during the protocol by the MNase. Because the nuclease cleaves specifically where the antibody is bound, the length of the resulting DNA is highly informative of the protein’s identity and its "footprint" on the genome. Generally, it is accepted that fragments <120 bp correspond to those bound by transcription factor, while 150 bp and 300 bp correspond to mono- and di-nuclesomes, respectively, indicating that the protein of interest is a histone or a histone binding protein. Fragments of 500 bp or above are usually considered background.
There are many tools to calculate the fragment size distribution, but [bedtools](https://bedtools.readthedocs.io/en/latest/) or the **bamPEFragmentSize** of **[deepTools](https://deeptools.readthedocs.io/en/latest/) are usually the go-to options.

## Spike-in Normalization & Scaling Factor

