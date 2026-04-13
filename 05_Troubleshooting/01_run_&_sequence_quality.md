# Sequencing Run & Quality Issues

This section covers issues detected immediately after FASTQ file generation, during primary quality assessment.

**Note:** Before FASTQ file generation, the sequencer performs a **primary analysis** of the data generated during the run, providing **.interop** format files. These files can be inspected using vendor-specific tools, allowing for detailed diagnosis of sequencing instrument failures (e.g. cluster generation, fluidics, imaging issues). Such metrics are outside the scope of this repository and therefore won't be covered here.

## Low Quality Scores

The first step when assessing the quality of the FASTQ files is looking at the Phred scores. Ideally, all samples should have a similar per sequence quality score, and a stable per base sequence quality thoughout the reads.

A low per base sequence quality, and therefore a low per sample quality score, in one of the samples is usually a sign of poor sample/library preparation. However, if this issue is present across all samples, a sequencing/run-level issue cannot be discarded and the primary analysis performed by the sequencer needs to be checked. In these cases, because there is a widespread below-threshold Q-score, aggresive trimming will not solve the issue and it is therefore heavily discouraged. 


