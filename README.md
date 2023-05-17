# Genome-Assembly-Annotation

## Genome Assembly
```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=12:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=32           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=800G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name assemble_BM16      # you can give your job a name for easier identification (same as -J)
#SBATCH -o BM17_assemble_slurm

########## Command Lines to Run ##########

module load GCC/10.2.0
module load SPAdes/3.15.2

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/SPAdes_assemblies
spades.py -o BM16_assembly --careful -t 32 -m 800 -1 BM16_S10_L002_R1_trim_UP.fastq -2 BM16_S10_L002_R2_trim_UP.fastq


scontrol show job $SLURM_JOB_ID
```

## Assembly statistics

```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=06:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=16           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=100G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name quast_BM16      # you can give your job a name for easier identification (same as -J)
#SBATCH -o BM17_quast_slurm

########## Command Lines to Run ##########

module load GCC/9.3.0  OpenMPI/4.0.3
module load QUAST/5.1.0rc1-Python-3.8.2

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/SPAdes_assemblies quast.py BM16.assembly.fa \
-r ../../0_DNAscripts/ReferenceGenome/Botrytis_cinerea.ASM83294v1.dna.toplevel.fa \
-g ../../0_DNAscripts/ReferenceGenome/Botrytis_cinerea.ASM83294v1.52.gff3 \
-t 16 --fungus --contig-thresholds \
-o /Quast_metrics/BM16_quast


scontrol show job $SLURM_JOB_ID

```

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


## Function annotation



### 1. InterProScan5
```
funannotate iprscan -i fun_BM16 -m local --iprscan_path /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/my_interproscan/interproscan-5.62-94.0/interproscan.sh
```
### 2. Eggnog-mapper

Now we want to run Eggnog-mapper. You can run this on their webserver http://eggnogdb.embl.de/#/app/emapper or if you have it installed locally then funannotate annotate will run it for you.


### 3. antiSMASH


### 4. phobius












