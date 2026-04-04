# Sequencing Data QC

## Sequence Depth VS Coverage

**Sequencing depth**, also known as read depth or depth of coverage, refers to the number of times a specific base (nucleotide) in the DNA is read during the sequencing process. In other words, it's the average number of times a given position in the genome is sequenced. A higher sequencing depth provides more confidence in the accuracy of the base calls at that position and helps to reduce sequencing errors and noise.
For example, if a specific nucleotide is sequenced 30 times, the sequencing depth at that position is 30x. This is the gold standard for whole genome sequencing (WGS) because, statistically, it ensures that >99.9% of the genome is covered by at least a few reads, minimizing the risk of missing a heterozygous variant.

Before the experiment is performed, the flow cell that will be used needs to be decided based on the aimed sequencing depth and the genome size:

**Total data (Gb) = Genome size (Gb) x Desired sequencing depth (x)**

For example, a full sequencing of a human genome (around 3.2 Gb) at 30x sequencing depth, requires 96 Gb of raw data.
Choosing the right flow cell for the experiment can save money and time. High-output flow cells are more expensive and take more time to image, but they might not be ideal if the total data required by the experiment isn’t very high.

Once the run is over, the actual sequencing depth achieved can be calculated through the following formula:

**Sequencing depth = Total number of reads x fragment length (bp) / Total genome size (bp)**

If a bacterial genome of 5 Mb is sequenced, and we obtain 2000000 reads of 150 bp, then the sequencing depth would be 60x

**Coverage** or **breadth of coverage** is closely related to sequencing depth but provides a broader perspective. Coverage is the proportion or percentage of a genome that has been sequenced at a certain depth. It gives an idea of how much of the entire genome has been effectively read and is usually expressed as a multiple of the genome's size, expressed as a percentage. For example, “95% coverage” means that 95% of the intended region has been sequenced at least once or a certain amount of times. It is calculated by bioinformatics tools (bedtools or samtools) after the mapping step, with the following formula:

**Coverage (%) = (Number of bases with at least one read / Total genome size) x 100**

The higher the sequencing depth, the lower the possibility that some positions won’t be sequenced or, in other words, the higher the coverage
| Coverage | Sequencing Depth |
|----|----|
| 95%	| ~3x to 5x average |
| 99%	| ~8x to 10x average |
| 99.99% | ~20x to 30x average |

One important term is **uniformity of coverage**. This is a measure of the variability of coverage across the genome. While the overall coverage might be the desired one, this is an average across all positions, so some bases might not be reaching it while others have an inflated coverage, skewing the %. This can be problematic, especially or WGS, where we want flat coverage. If a position shows a much lower coverage, then the detection of mutations with high confidence won’t be possible. For ATAC-seq or CUT&RUN, we actually want bumpy coverage (peaks where the protein binds and valleys everywhere else).

The most common cause of low uniformity of coverage is **PCR bias**. Some regions are easier to amplify by the polymerase, including those with lower GC content, leading to an overrepresentation of these fragments.

The uniformity of coverage is usually calculated by tools like Picard or Mosdepth with the following formula:

**Coefficient of Variation (CV) = Standard Deviation of Depth / Average Sequencing Depth**. The lower this metric (0.1-0.2), the more uniform the data.

