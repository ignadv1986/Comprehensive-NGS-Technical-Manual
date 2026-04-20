# NGS-From-Bench-to-Bioinformatics

A technical reference manual covering the full Next Generation Sequencing workflow, from sequencing chemistry and library preparation through to computational analysis and troubleshooting. Unlike most resources that treat wet lab and bioinformatics as separate domains, this manual follows the data continuously from the flow cell to biological interpretation, with the goal of making each step understandable in the context of the ones before and after it.
The troubleshooting sections are written from practical experience: each issue is traced back to its most likely origin in the experimental or computational workflow, with concrete diagnostic criteria and actionable recommendations.
This is a living document and will be updated over time.

Contents
01 — NGS Foundations & Pre-processing
FileDescription01_sequencing_physics.mdSequencing by synthesis, bridge amplification, optical chemistry, patterned flow cells, and common run-level failure modes02_experimental_design.mdSequencing depth, coverage, read configuration, multiplexing strategy, and practical design trade-offs03_library_preparation.mdEnd repair, adapter ligation, SPRI size selection, PCR amplification, quantification, and pooling04_sequencing_QC.mdFastQC metrics, Phred scores, trimming strategy, and MultiQC
02 — Mapping & Alignment
FileDescription01_mapping_principles_&_QC.mdReference genomes, genome indexing, seed-and-extend alignment, SAM/BAM format, MAPQ, and mapping QC02_aligners.mdSTAR, HISAT2, BWA-MEM2, Bowtie2, Salmon, and kallisto — when and why to use each03_post-alignment_processing.mdBAM sorting, duplicate marking, blacklist filtering, spike-in normalization, and final indexing
03 — Transcriptomics (RNA-seq)
FileDescription01_pre-processing.mdRNA quality assessment (RIN), enrichment strategies, fragmentation, cDNA generation, strandedness, and UMIs02_analysis_workflow.mdRead quantification with featureCounts, differential expression with DESeq2, LFC shrinkage, and functional enrichment
04 — Epigenomics
FileDescription01_ChIP-seq_vs_CUT&RUN.mdChIP-seq principles, limitations, and a direct comparison with CUT&RUN02_CUT&RUN_sample_prep.mdpAG-MNase mechanism, fragment size biology, IgG control, SPRI strategy, PCR amplification, and quantification03_CUT&RUN_analysis.mdSpike-in normalization, coverage file generation, peak calling with SEACR and MACS3, replicate handling, and biological interpretation04_ATAC-seq_sample_prep.mdTn5 transposase mechanism, tagmentation, fragment size distribution, the 9 bp shift, and SPRI selection05_ATAC-seq_analysis.mdAlignment strategy, mitochondrial read handling, TSS enrichment, coverage files, peak calling with MACS3, DiffBind, and differential accessibility
05 — Troubleshooting
FileDescription01_fragment_size_distribution.mdAdapter dimers, broad distributions, high molecular weight contamination, and loss of expected fragment populations02_run_&_sequence_quality.mdLow Phred scores, GC content anomalies, adapter contamination, duplication patterns, and per-base N content03_alignment_&_data_quality.mdLow mapping rates, multi-mapping, and integrated interpretation of alignment metrics04_pipeline_&_computational(RNA-seq).mdfeatureCounts and Salmon quantification issues, DESeq2 model failures, and batch effect diagnosis05_pipeline_&_computational(CUT&RUN).mdGenome browser interpretation, coverage file generation, SEACR peak calling artifacts, and replicate inconsistency06_pipeline_&_computational(ATAC-seq).mdTSS enrichment failures, coverage artifacts, MACS3 peak calling issues, featureCounts misconfiguration, and DESeq2 edge cases

License
This work is licensed under the MIT License. You are free to use, share, and adapt the material with attribution.

For questions or collaborations, visit my GitHub profile.
