Sand Lizard (*Lacerta agilis*) Genomics — Disertation project log
================
Ty White
Last updated: 2026-11-20

------------------------------------------------------------------------

## October 2026

### 9 October 2026 — Meeting & Technical Setup

- **Meeting:** Discussed initial steps for preparing the project and
  signed up for the upcoming bioinformatics bootcamp.
- **Technical Setup:** Created an account on Supercomputing Wales (SCW)
  and installed Windows Subsystem for Linux (WSL).
- **Next Steps:** Work through the Oxford University Bash introduction.

### 12 October 2026 — Self-Study

- **Progress:** Completed the *“Introduction to shell”* and *“File
  directories”* sections of the Oxford Bash introduction.
- **Next Steps:** Continue onto the next sections of the Bash
  introduction.

### 14 October 2026 — Self-Study

- **Progress:** Completed the *“Creating things”* and *“Pipes and
  filters”* sections of the Bash introduction.
- **Next Steps:** Complete the *“Shell scripts”* and *“Loops”* sections.

### 16 October 2026 — Self-Study

- **Progress:** Completed the *“Shell scripts”* section of the Bash
  introduction.
- **Next Steps:** Complete the *“Loops”* section.

### 21 October 2026 — Self-Study

- **Progress:** Completed the *“Loops”* section, finishing the Oxford
  Bash introduction.
- **Next Steps:** Attend the next supervisor meeting to discuss project
  progression.

### 28 October 2026 — Meeting

- **Progress:** Reviewed progress made during Bash self-study and
  discussed preparation for the bioinformatics bootcamp.
- **Next Steps:** Attend the bioinformatics bootcamp.

------------------------------------------------------------------------

## November 2026

### 2–6 November 2026 — Training

- **Progress:** Attended the Bioinformatics Bootcamp (covering
  command-line workflows, Slurm job submission, and genomics tools).
- **Next Steps:** Attend the post-bootcamp supervisor meeting.

### 11 November 2026 — Meeting

- **Progress:** Received the *Lacerta agilis* BAM files and discussed
  initial analytical steps.
- **Next Steps:** Calculate D-statistics to test for
  introgression/admixture.

### 12 November 2026 — Self-Study

- **Progress:** Ran D-statistic and block Jackknife analyses; results
  showed no significant evidence of admixture.

``` bash
module purge
module load angsd/0.940-GCC-12.3.0

angsd -doAbbababa 1 /
-doCounts 1 /
-minQ 30 /
-minMapQ 30 /
-b /scratch/SCWF00043/people/b.tyw24bzh_ty/allbam.txt /
-anc /scratch/SCWF00043/BEARCAVE0/refgenomes/rLacAgi1/GCA_009819535.1_rLacAgi1.pri_genomic.fa /
-out dstat -nThreads 10
```

- **Next Steps:** Present and discuss these D-statistic results at the
  next meeting.

### 16 November 2026 — Meeting & Self-Study

- **Meeting:** Discussed the non-significant D-statistic results and
  agreed on next steps: running an autosomal D-statistic and building a
  Maximum Likelihood (ML) phylogeny.
- **Self-Study:** Performed the autosomal D-statistic analysis, which
  also yielded no significant evidence for admixture.

``` bash
module purge
module load angsd/0.940-GCC-12.3.0


angsd -doAbbababa 1 /
-doCounts 1 /
-minQ 30 /
-minMapQ 30 /
-b /scratch/SCWF00043/people/b.tyw24bzh_ty/allbam.txt /
-rf /scratch/SCWF00043/BEARCAVE0/refgenomes/rLacAgi1/autosomes.txt /
-anc /scratch/SCWF00043/BEARCAVE0/refgenomes/rLacAgi1/GCA_009819535.1_rLacAgi1.pri_genomic.fa/
-out firstdstat -nThreads 10
```

- **Next Steps:** Generate consensus FASTA sequences and align genomes
  across samples.

### 17 November 2026 — Self-Study

- **Progress:** Generated consensus FASTA sequences across all 19
  *Lacerta agilis* samples using `ANGSD` (`-doFasta 2`) and merged them
  into a multi-FASTA alignment.

``` bash
module purge
module load angsd/0.940-GCC-12.3.0

while read -r bam; do
    sample=$(basename "$bam" .bam) #extracts prefix for output file

#Generate each concensous fasta
    angsd -doFasta 2 -doCounts 1 -minQ 30 -minMapQ 30 -setMinDepthInd 3 \
          -basesPerLine 100000000 -i "$bam" -out "$sample"
done < allbam.txt
```

``` bash
#combine each fasta file while giving them unique headers
for f in *.fa.gz; do
    sample=$(basename "$f" .fa.gz)
    zcat "$f" | sed "s/^>/>${sample}_/"
done > all_samples.fa
```

- **Next Steps:** Strip invariant and missing-data sites using
  `snp-sites` and infer a Maximum Likelihood tree using `iqtree3`.

### 18 November 2026 - Self-Study

- **progress:** code for stripping sites and maximum likelihood tree
  create

``` bash
module purge
module load snp-sites/2.5.1-GCCcore-12.3.0

snp-sites -c clean_merged_equal.fasta > finalla.fa

module purge
module load IQ-TREE/3.0.1-gompi-2024a

iqtree3 -s finalla.fa \
        -o d9e_wwLA5398_agi_agilis_14.55205 \
        -T 4 \
        -B 1000 \
        -m GTR+G+ASC \
        --redo
```

- **problem:** snp-sites not working

- **reason** repeated headers per sample

- **fix:** extract sample ID, remove all headers for each consensous
  fasta and write a single FASTA header per file

``` bash
for f in *.fa.gz; do
    sample=$(basename "$f" .fa.gz)
    echo ">${sample}" >> clean_merged.fasta
    zcat "$f" | grep -v "^>" >> clean_merged.fasta
done
```

- **problem:** snp-sites not working

- **error message:**Alignment clean_merged.fasta contains sequences of
  unequal length. Expected length is 1391406172 but got 1391398340 in
  sequence on5+Lacerta_agilis_Denmark_DM111_agi_agilis_4.76924

- **fix:** remove the 7832 bp line

``` bash
awk 'length($0) != 7832' clean_merged.fasta > clean_merged_equal.fasta
```

- **problem:** IQtree not working
- **reasons:** IQtree automatically converted the “+” in the FASTA
  header to “\_” for -o so outgroup was not found
- **fix:** change the “+” in -o to a “\_”

``` bash
#final working script

module purge
module load snp-sites/2.5.1-GCCcore-12.3.0

snp-sites -c clean_merged_equal.fasta > finalla.fa

module purge
module load IQ-TREE/3.0.1-gompi-2024a

iqtree3 -s finalla.fa \
        -o d9e_wwLA5398_agi_agilis_14.55205 \
        -T 4 \
        -B 1000 \
        -m GTR+G+ASC \
        --redo
```

### 20 November 2026 - Self-Study

- **progress:** Produced maximum likehood tree

![](C:\Users\chees\Documents\Disertation\Maxliketree.svg)
