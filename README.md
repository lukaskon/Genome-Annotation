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

### Genome Assembly Loop

```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=12:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=32           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=800G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name assemble_CCR7      # you can give your job a name for easier identification (same as -J)
#SBATCH -o CCR7_assemblies_slurm

########## Command Lines to Run ##########

module load GCC/10.2.0
module load SPAdes/3.15.2

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/SPAdes_assemblies/Trimmed_fastas
gunzip *.gz

for file1 in *R1_*fastq
do
file2=${file1/R1/R2}
out=${file1%%.fastq}_assembly
spades.py --careful -t 32 -m 800 -1 $file1 -2 $file2 -o ../$out &
done

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
#SBATCH -o BM16_quast_slurm

########## Command Lines to Run ##########

module load GCC/9.3.0  OpenMPI/4.0.3
module load QUAST/5.1.0rc1-Python-3.8.2

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/SPAdes_assemblies

quast.py BM16_assembly/BM16_scaffolds.fasta \
-r ../../0_DNAscripts/ReferenceGenome/Botrytis_cinerea.ASM83294v1.dna.toplevel.fa \
-g ../../0_DNAscripts/ReferenceGenome/Botrytis_cinerea.ASM83294v1.52.gff3 \
-1 BM16_S10_L002_R1_trim_UP.fastq \
-2 BM16_S10_L002_R1_trim_UP.fastq \
-t 16 --fungus --contig-thresholds 0,1000,2000,5000,10000 \


scontrol show job $SLURM_JOB_ID

```

View report.html in browser for statistics.
Check contig sizes to ensure that a cutoff of 1000 is appropriate for downstream.


## Annotate genome using the funannotate pipeline
https://funannotate.readthedocs.io/en/latest/index.html


#### Clean, sort, mask repeats

```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=05:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=32           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=50G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name CSM_BM16      # you can give your job a name for easier identification (same as -J)
#SBATCH -o BM16_CSM_slurm

########## Command Lines to Run ##########

conda activate funannotate

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/SPAdes_assemblies/BM16_assembly

funannotate clean -i BM16_scaffolds.fasta -o BM16_C.fasta -m 1000 --exhaustive #Filters out contigs under 1000bp in length
funannotate sort -i BM16_C.fasta -o BM16_CS.fasta -b scaff
funannotate mask -i BM16_CS.fasta -o BM16_CSM.fasta -s botrytis_cinerea --cpus 32

conda deactivate funannotate

scontrol show job $SLURM_JOB_ID

```

funannotate-mask.log contains
1. Number of scaffolds
2. Size of genome
3. Masked repeats (bp and %)


## Functional Annotation Tools



#### 1. InterProScan5
```
funannotate iprscan -i fun_BM16 -m local --iprscan_path /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/my_interproscan/interproscan-5.62-94.0/interproscan.sh
```
#### 2. Eggnog-mapper

Now we want to run Eggnog-mapper. You can run this on their webserver http://eggnogdb.embl.de/#/app/emapper or if you have it installed locally then funannotate annotate will run it for you.


#### 3. antiSMASH


#### 4. phobius












