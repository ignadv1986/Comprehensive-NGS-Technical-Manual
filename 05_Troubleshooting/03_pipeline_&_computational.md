# Pipeline & Computational Troubleshooting (RNA-seq)

Even when sequencing QC and mapping metrics are within expected ranges, RNA-seq results can still appear inconsistent or biologically implausible during quantification and differential expression analysis. In these cases, the issue is typically not related to data quality, but to how reads are assigned, summarized, and statistically modeled downstream.

## Gene Quantification (featureCounts)

After successful alignment, read counting tools such as featureCounts may produce unexpected or biologically inconsistent gene count distributions. A common cause is **incomplete or mismatched gene annotation**. If the GTF file does not match the genome build used for alignment, a substantial fraction of reads may map correctly but fail to be assigned to any feature. This results in inflated “unassigned” fractions and artificially reduced gene counts.

Additionally, gene quantification using featureCounts can produce unexpected or biased results depending on **parameter selection**. In most cases, issues are not caused by the data itself, but by mismatches between library properties and counting settings.

- **Strand-specific counting:** One of the most common sources of incorrect gene counts is a mismatch between library strandedness and the -s parameter used in featureCounts. If the strandedness setting does not match the library preparation protocol, reads may be incorrectly assigned or discarded.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| -s | 0 (unstranded)<br>1 (forward stranded)<br>2 (reverse stranded) | Strand specification does not match library protocol | Increased unassigned reads, loss of antisense signal, distorted gene expression profiles |

- **Feature definition:** This type of error is typically detected by an unusually high proportion of unassigned reads in the featureCounts summary, often accompanied by reduced signal in genes expected to be expressed.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| -t | Exon/other feature types | Incorrect feature type selection relative to annotation | Systematic under-counting |
| -g | gene_id/other grouping keys | Mismatch with annotation structure | Fragmented or inconsistent gene-level counts |

- **Paired-end handling:** For paired-end datasets, reads must be treated as fragments rather than independent sequences. Failure to properly enable paired-end counting can distort library size estimates and gene expression values.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| -p | Enabled/disabled | Paired-end data treated as single-end  | Inflated read counts and incorrect library size estimation |
| --countReadPairs | Enabled/disabled | Fragment not counted as a single unit | Double counting of fragments and distorted expression levels |

-**Fragment filtering:** Additional parameters control which fragments are considered valid for counting. While these filters can improve specificity, overly strict or inappropriate settings can remove biologically relevant reads.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| -d/-D | Thresholds | Fragment size constraints too strict or too permissive  | Loss of valid fragments or inclusion of low-quality alignments |
| -B | On/off | Requires both ends to be mapped | Loss of valid partially mapped fragments |
| -C | On/off | Excludes chimeric fragments | Removal of potentially biologically relevant reads in some contexts |

-**Quick diagnostic guide:**

| Symptom | Likely cause | Parameter(s) to check |
| :--- | :--- | :--- |
| High proportion of unassigned reads despite good mapping | Incorrect strand specification | `-s` |
| Gene counts lower than expected across all samples | Mismatch between annotation and feature definition | `-t`, `-g` |
| Unexpectedly high library sizes / inflated counts | Paired-end reads counted incorrectly as single reads | `-p`, `--countReadPairs` |
| Strong discrepancies between featureCounts and Salmon results | Differences in read assignment or annotation structure | `-t`, `-g` |
| Loss of reads in specific regions or globally reduced counts | Overly strict fragment filtering | `-d`, `-D`, `-B`, `-C` |

## Gene Quantification (Salmon)

Even when alignment or quasi-mapping is successful, quantification with Salmon can produce unexpected or biologically inconsistent expression estimates. In most cases, these issues arise from **mismatches between transcriptome reference, library properties, and quantification parameters**, rather than from the sequencing data itself.

A frequent source of error is **inconsistent transcriptome annotation**. Salmon relies entirely on the provided transcriptome index; if it does not correspond to the same genome build or annotation used elsewhere in the pipeline, reads may be assigned incorrectly or ambiguously, leading to biased transcript and gene-level estimates.

Additionally, Salmon performs transcript-level quantification, which introduces complexities such as multi-mapping reads and isoform ambiguity.

- **Library type specification:** Incorrect library type detection or specification is one of the most common causes of biased quantification.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| `--libType` | A (auto)<br>ISR / SR / IU / etc. | Library type incorrectly specified or auto-detection fails | Misassignment of reads, strand bias, distorted expression estimates |

- **Transcriptome reference:** Salmon depends entirely on the transcriptome index for mapping and quantification.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Index input | Transcript FASTA | Mismatch with genome build or GTF used downstream | Inconsistent gene-level summarization and missing transcripts |
| Decoy sequences | With/without decoys | Missing decoy-aware index | Increased spurious mapping and inflated expression estimates |

- **Mapping and alignment mode:** Salmon supports both quasi-mapping and alignment-based modes, each with different sensitivities.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Mapping mode | Quasi-mapping / Alignment-based | Inappropriate mode for data quality or complexity | Reduced accuracy in complex transcriptomes |
| `--validateMappings` | On/off | Disabled selective alignment | Increased false-positive mappings |

- **Bias correction:** Salmon includes advanced models to correct technical biases. Disabling or misusing them can skew results.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| `--seqBias` | On/off | Sequence-specific bias not corrected | Systematic distortion in expression estimates |
| `--gcBias` | On/off | GC bias not modeled | Bias toward GC-rich or GC-poor transcripts |
| `--posBias` | On/off | Positional bias ignored | Inaccurate fragment distribution modeling |

- **Fragment and length modeling:** Salmon models fragment length distributions, which are critical for accurate quantification.

| Parameter | Possible Selections | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Fragment length priors | Auto / manual | Incorrect assumptions for fragment size | Biased transcript abundance estimates |
| Paired-end handling | Implicit | Incorrect pairing or inconsistent input | Misestimation of effective lengths |

- **Gene-level summarization:** Salmon outputs transcript-level estimates; gene-level counts require aggregation (e.g., tximport).

| Step | Tool | Issue | Consequence |
| :--- | :--- | :--- | :--- |
| Transcript → gene | tximport / custom mapping | Incorrect or inconsistent transcript-to-gene mapping | Fragmented or incorrect gene-level counts |

- **Quick diagnostic guide:**

| Symptom | Likely cause | Parameter(s) to check |
| :--- | :--- | :--- |
| Strong strand bias or unexpected antisense signal | Incorrect library type | `--libType` |
| Expression detected in unexpected transcripts | Poor transcriptome reference or missing decoys | Index, decoy setup |
| Inflated expression estimates or noisy background | Disabled selective alignment | `--validateMappings` |
| Systematic bias across GC content or transcript length | Bias correction disabled | `--seqBias`, `--gcBias`, `--posBias` |
| Discrepancies between Salmon and featureCounts | Transcript vs gene-level differences or annotation mismatch | Index, tx2gene mapping |
| Fragment length inconsistencies | Incorrect pairing or fragment modeling | Input format, library type |
