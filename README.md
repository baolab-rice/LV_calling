# LV_caller

A downstream analysis pipeline focusing on large gene modification (large variants) after CRISPR/Cas9 editing. The pipeline takes the output from longread_umi[[1]](#1) pipeline using SMRTseq data and generates a series of files and figures for large gene modification analysis.

Note: This pipeline has not been tested with broad cases, please let the author (mingming.cao@rice.edu) know if there's any issue or suggestion. Thanks in advance!

Updated date: 2022-7-6

**Table of contents**
- [Schematics](#schematics)
- [Installation](#installation)
- [Quick start](#quick-start)
- [Usage](#usage)
- [Process](#Process)
- [Output Format](#output)
- [References](#references)

## Schematics
![Schematics](Schematic.png)

## Installation 

Note that our pipeline is used for downstream analysis after longread_umi pipeline. If you need to learn how the initial steps work, please refer to https://github.com/SorenKarst/longread_umi

1. Requirements & Dependencies
`python` >= 3.6.0
`minimap2` >= 2.17 [[2]](#2)
`BLAT` version 35 [[3]](#3)

Be sure to export the path to these programs.

2. Install python packages
```bash
conda install numpy scikit-learn pandas scipy
conda install -c conda-forge matplotlib
```
These packages are applied in distribution generation.

3. Download the scripts
```bash
git clone https://github.com/baolab-rice/LV_calling.git
```
or
Download zip file from our github page.

4. The SMRTseq_data_processing .py in the scrips folder is the main script to be used. 

## Quick-start

Note: For an efficient running, we use two references. One with flag 'g' is the amplicon product reference; Another with flag 'G' is the genome reference for large insertion mapping.

In the script folder, you could run with the following line. (Note:rxn is for identifying the folder from longread_umi, usr may manually find it.)

 ```bash
 python3 SMRTseq_data_processing.py \
         -d <longread_umi output directory> \
         -o <An existed folder to store output data> \
         -st 2 \
         -os 2 \
         -m \
         -g <amp ref.fasta> \
         -ld \
         -ld_ps <cutsite_pos> \
         -li \
         -ld_c \
         -ld_cps <deletion_size_tolerance + 'd' + position_tolerance + 'l'>   
         -rxn <3>
 ```

## Usage

```bash
usage: SMRTseq_data_processing.py [-h] -d DIRECTORY -o OUTPUT [-st {1,2}] [-os {1,2}] [-r READS] [-m] [-g REFERENCE] [-ld] [-ld_ps LARGE_DELETION_PARAMETERS] [-li]
                                  [-G GENOME] [-ld_c] [-ld_cps LARGE_DELETION_CLUSTERING_PARAMETERS] [-a]

You may use 'seqtk seq -a <fastq> > <fasta> to convert file if raw reads need to be processed.'

For longumi_read downstraem analysis: Version 3 this script is for extracting data from longumi_read pipeline output. 

optional arguments:
  -h, --help            show this help message and exit
  -d DIRECTORY, --directory DIRECTORY
                        Directory of outout folder with longumi_read pipeline, the output folder should
                        contain a raconx subfolder.
  -o OUTPUT, --output OUTPUT
                        Output directory. Output the completly organized file.
  -st {1,2}, --stats {1,2}
                        Output the stats filem (default=2): -st 1: UMI_ID Read_IDs
                        Consensus_read_sequence; -st 2: UMI_ID Read_count Consensus_read_sequence
  -os {1,2}, --output_style {1,2}
                        If also involve raw reads, can also produce a file contaning all reads. -os 1:
                        UMI_ID Read_ID Read_sequence Centroid_ID. -os 2: UMI Consensus_seq Reads_seqs
  -R READS, --Reads READS
                        Unbarcoded reads.
  -m, --mapping         Mapping all filetered read to reference amp using minimap2. For the large
                        deletion analysis option, could ONLY use minimap2.
  -g REFERENCE, --reference REFERENCE
                        Alignment reference amp. If use HDR mode, the expected HDR amplicon sequence is
                        required.
  -ld, --large_deletion
                        Large deletion calling, devide LDs as small INDELs or unmodified, 50bp-200bp,
                        and >=200bp.
  -ld_ps LARGE_DELETION_PARAMETERS, --large_deletion_parameters LARGE_DELETION_PARAMETERS
                        Large deletion analysis parameters:
  -li, --large_insertion
                        Large insertion (>=50bp) calling.
  -G GENOME, --genome GENOME
                        Reference Genome.
  -ld_c, --large_deletion_clustering
                        Large deletion clustering based on deletion size and deletion start site.
  -ld_cps LARGE_DELETION_CLUSTERING_PARAMETERS, --large_deletion_clustering_parameters LARGE_DELETION_CLUSTERING_PARAMETERS
                        Large deletion clustering parameters: FORMAT:
                        deletion_size_tolenrance+d+deletion_position_tolerance+l Default: 10d10l

```

## Process
[Gathering umibins list...] \
[Reading from umibins...] \
[Polishing umis...] \
[Reading from raw fasta file...] \
[Mapping reads with reads IDs...] \
[Writing into files...] \
[Generating fasta file for consensus seqs...] \
[Alignment using minimap2...] \
[Large deletions calling...] \
[Large insertions calling...] \
[Large deletions and large insertions rearranging...] \
[Clustering large deletions (>=200bp)...] \
[Generating distribution figure for large deletions (>=200bp)...] \
[Generating distribution figure for clustered large deletions (>=200bp)...] \
[Removing temp files...] \
[Mapping large insertions (>=50bp) to reference genome...] \
[Generating stats...] \
Program finished. 

## Output
**1. result.txt**

Rearrangement of UMIs and reads with sequences.

Output option 1

| UMI_ID   | Read_ID | Read_sequence | Centroid_ID |
|----------|---------|---------------|-------------|
| umi1     | ReadID1 | seq1          | ID1         |
| umi2     | ReadID2 | seq2          | ID2         |

Output option 2

| UMI_ID                   |
|--------------------------|
| Consensus sequence1: seq |
| Read1: seq               | 
| Read2: seq               |
| Consensus sequence2: seq |
| Read1: seq               | 
| Read2: seq               |

**2. result_stats.txt**

Inital stats with UMI_ID and read binned to that UMI with ID or counts.

Output option1

| UMI_ID   | Read_IDs | Consensus_read_sequence |
|----------|----------|-------------------------|
| umi1     | ID1,ID2..| seq1                    |
| umi2     | IDn,IDm..| seq2                    |

Output option2

| UMI_ID   | Read_count | Consensus_read_sequence |
|----------|------------|-------------------------|
| umi1     | 20         | seq1                    |
| umi2     | 18         | seq2                    |

**3. result_invalidated_umis.txt**

UMIs those consensus reads could not be mapped to amplicon/gene reference.

| UMI      |
|----------|
| umi1     |
| umi2     |

**4. result_LD200.txt result_LD50to200.txt**

Large deletions grouped by size, applying >=200bp and 50bp-200bp.

| UMI	     | Start	| End	 | Deletion_length	| If_cover_cutsite	| Read	      | Alternative_deletion         |
|----------|-------|------|-----------------|------------------|------------|------------------------------|
| umi10005	| 2853	 | 3175	| 322            	| no               | CATACGA... | None                         |
| umi10010	| 2580 	| 2849	| 269	            | yes              | CGGCATG... | [(2580, 2849), (3564, 4197)] |

**5. result_small_INDELs_and_unmodified.txt**

Reads with small INDELs or reads without any modification.

| UMI   | Read       | 
|-------|------------|
| umi1  | seq        | 
| umi2  | seq        | 

This file can be converted to .fasta using:

```
cat ./outputs/result_small_INDELs_and_unmod.txt | sed 's/umi/>umi/g' | grep "umi" | sed 's/\t/\n/g' > SU.fasta
```


**6. result_LI_with_LD200_besthit.txt result_LI_with_LD50_besthit.txt result_LI_with_other_besthit.txt**

Large insertions mapped to reference genomem, grouped with LI and LD together or with sINDELs/unmodified reads.

| UMI	    | Chr	  | Match	| Mismatch	| Strand	| Start	   | End      |
|---------|-------|-------|----------|--------|----------|----------|
| umi5296 |	chr19	| 475	  | 38	      | +	     | 27732180	| 27739479 |
| umi122	 | chr11	| 580	  | 5	       | +	     | 5248622	 | 5249208  |

**7. result_LD200_cluster.txt**

Clustered large deletions.

| Cluster_size | Start	| End	 | Length	| 
|--------------|-------|------|--------|
| 1            |	2660 	| 2849 | 109	   | 
| 6     	      | 2828	 | 3031	| 203	   | 

**8. result_output_stats.txt**

Final output stats.

| Ref_CCS_reads:             |
|----------------------------|
| 74424                      |
| UMI_consensus:             |
| 7158                       |
| Invalidated_UMI_consensus: |
| 4                          |
| Ref_UMI_consensus:         |
| 7154                       |
| LD200:                     |
| 1112                       |
| LD50-200:                  |
| 420                        |
| smallINDEL:                |
| 3711                       |
| unmodified:                |
| 1802                       |
| LI_with_LD200:             |
| 16                         |
| LI_with_LD50:              |
| 32                         |
| LI_other:                  |
| 61                         |
| clusters in LD200:         |
| 495                        |

**9. result_LD200.svg result_LD200_cluster.svg**

The distribution plotting.

![distribution](Distribution.svg)

## Citation
Park SH, Cao M, Pan Y, et al. Comprehensive analysis and accurate quantification of unintended large gene modifications induced by CRISPR-Cas9 gene editing. Sci Adv. 2022;8(42):eabo7676. doi:10.1126/sciadv.abo7676

## References
<a id="1">[1]</a> 
SM Karst, RM Ziels, RH Kirkegaard, EA Sørensen, D. McDonald, Q Zhu, R Knight, & M Albertsen. (2020). Enabling high-accuracy long-read amplicon sequences using unique molecular identifiers with Nanopore or PacBio sequencing. bioRxiv, 6459039. https://github.com/SorenKarst/longread_umi

<a id="1">[2]</a> 
Li, H. (2018). Minimap2: pairwise alignment for nucleotide sequences. Bioinformatics, 34:3094-3100. doi:10.1093/bioinformatics/bty191 https://github.com/lh3/minimap2

<a id="1">[3]</a> 
BLAT: Kent WJ. BLAT - the BLAST-like alignment tool. Genome Res. 2002 Apr;12(4):656-64.
