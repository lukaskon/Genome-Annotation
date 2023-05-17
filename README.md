# Genome-Annotation


## Annotate genome using the funannotate pipeline
https://funannotate.readthedocs.io/en/latest/index.html


### 1. Convert bam files to fasta format

```
samtools fasta BM16_S10_aln_bwamem2.bam > BM16_aligned.fasta
```

### 2. Sort and rename fasta headers
```
conda activate funannotate
funannotate sort -i BM16_aligned.fasta -o BM16_aligned_sorted.fasta -b scaff
```
Using "scaff" as basename instead of "scaffold" because some headers would be too long for downstream processes.


### 3. Clean fasta
```
funannotate clean -i BM16_aligned_sorted.fasta -o BM16_aligned_sorted_clean.fasta
```

### 4. RepeatMasking (skipped clean step)
```
funannotate predict -i BM16_aligned_sorted_mask.fasta -o fun_BM16 --species "Botrytis cinerea" --strain BM16 --busco_seed_species botrytis_cinerea --cpus 24
```

