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

#SBATCH --time=13:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=32           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=800G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name assemble_CCR7      # you can give your job a name for easier identification (same as -J)
#SBATCH -o CCR7_assemblies_slurm

########## Command Lines to Run ##########

module load GCC/10.2.0
module load SPAdes/3.15.2

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/SPAdes_assemblies/Trimmed_fastas
for infile in *_R1_trim_UP.fastq
do
base=$(basename ${infile} _R1_trim_UP.fastq)
spades.py -o ../${base}_assembly --careful -t 32 -m 800 -1 ${base}_R1_trim_UP.fastq -2 ${base}_R2_trim_UP.fastq
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



## Annotate
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



### Run Busco v5 separate from funannotate if wanted (need to use local writable Augustus, change config)

```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=02:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=24           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=50G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name Buscoloop      # you can give your job a name for easier identification (same as -J)
#SBATCH -o Busco_loop_slurm

########## Command Lines to Run ##########

module purge
module load GCC/10.2.0  OpenMPI/4.0.5
module load BUSCO/5.3.0
export AUGUSTUS_CONFIG_PATH=/mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Augustus-master/config

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/SPAdes_assemblies

for infile in AI7* W18* B5* BU9* I9* R23* Y1*

do

base=$(basename ${infile} _assembly)

cd ${base}_assembly

busco -i ${base}_CSM.fasta \
-m genome \
-o ../Helotiales_busco/${base}_busco_helotiales \
-l /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/busco_downloads/lineages/helotiales_odb10 \
--augustus --augustus_species botrytis_cinerea \
--cpu 24

cd ..

done

conda deactivate
```






busco -i ${base}_CSM.fasta -m genome -o ${base}_busco -l /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/busco_downloads/lineages/helotiales_odb10 --augustus --augustus_species botrytis_cinerea --cpu 24







## Functional Annotation Tools



#### 1. InterProScan5
Installed InterProScan locally, then created symbolic links to the pf* files within the conda funannotate bin directory.
```
conda activate funannotate
module purge
module load Java/19.0.2
./interproscan.sh -i test_all_appl.fasta -f tsv -dp
```
Current error: 
Error: File format problem in trying to open HMM file data/gene3d/4.3.0/gene3d_main.hmm.
Opened data/gene3d/4.3.0/gene3d_main.hmm.h3m, a pressed HMM file; but format of its .h3i file unrecognized

Use Interproscan directly instead of through funanotate. Code recommended by creator in a comment on issues thread: https://github.com/nextgenusfs/funannotate/issues/841
```
#interproscan.sh -i /pathto/predict_results/genome.proteins.fasta -f XML -goterms -pa
```
```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=03:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=12           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=100G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name AF13_test_IPR      # you can give your job a name for easier identification (same as -J)
#SBATCH -o IPR_test5_slurm

########## Command Lines to Run #########

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_Annotate

for infile in AF13T*

do

base=$(basename ${infile} _fun)

../my_interproscan/interproscan-5.62-94.0/interproscan.sh -i ${base}_fun/predict_results/Botrytis_cinerea_${base}.proteins.fa \
--cpu 12 -f XML -goterms -pa \
-o ${base}_fun/predict_results/${base}_ipr.xml

done

scontrol show job $SLURM_JOB_ID


```

#### 2. Eggnog-mapper

Used interproscan instead


#### 3. antiSMASH

```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=2:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=24           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=100G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name Antismash_loop      # you can give your job a name for easier identification (same as -J)
#SBATCH -o Antismash_BT15_slurm

########## Command Lines to Run #########

module purge
conda activate antismash

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_Annotate
for infile in *_fun
do
base=$(basename ${infile} _fun)
cd ${base}_fun/predict_results
antismash Botrytis_cinerea_${base}.gbk --taxon fungi --pfam2go --genefinding-gff3 Botrytis_cinerea_${base}.gff3 --output-basename ${base}_smash --cpu 24
cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_Annotate
done

scontrol show job $SLURM_JOB_ID

```
Counts can be found in slurm titled "Antismash_Count_slurm"


#### 4. phobius

Need to obtain binary package from creator. The package is sent via email, but email blocks binary attachments. No response from creator.

#### 5. SignalP
```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=24:00:00             # limit of wall clock time - how long the jo
b will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) t
hat you require (same as -n)
#SBATCH --cpus-per-task=16           # number of CPUs (or cores) per task (same
as -c)
#SBATCH --mem=100G                    # memory required per node - amount of mem
ory (in bytes)
#SBATCH --job-name SignalP6      # you can give your job a name for easier id
entification (same as -J)
#SBATCH -o SignalP_slurm

########## Command Lines to Run #########

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_Annotate

signalp6 --version

for infile in *_fun

do

base=$(basename ${infile} _fun)

cd ${base}_fun/predict_results
mkdir signalp

signalp6 --fastafile Botrytis_cinerea_${base}.proteins.fa \
--output_dir /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_An
notate/${base}_fun/predict_results/signalp \
--format txt --mode fast

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_Annotate

done

scontrol show job $SLURM_JOB_ID
```
##### Count of proteins most likely to contain "standard" secretory signals (>50% likelihood)
```
cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_Annotate

echo "Count of secreted proteins"
echo
echo "Sec/SPI: "standard" secretory signal peptides transported by the Sec trans
locon and cleaved by Signal Peptidase I (Lep)"
echo

for infile in *_fun

do

base=$(basename ${infile} _fun)

cd ${base}_fun/predict_results/signalp

echo ${base}

awk '$4 ~ /\.[98765]/ { print $0 }' prediction_results.txt|wc -l

echo

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_Annotate

done

```


## Annotate with funannotate

```
#!/bin/bash --login
########### Define Resources Needed with SBATCH Lines ##########

#SBATCH --time=01:00:00             # limit of wall clock time - how long the job will run (same as -t)
#SBATCH --ntasks=1                  # number of tasks - how many tasks (nodes) that you require (same as -n)
#SBATCH --cpus-per-task=24           # number of CPUs (or cores) per task (same as -c)
#SBATCH --mem=80G                    # memory required per node - amount of memory (in bytes)
#SBATCH --job-name Manu_annotate      # you can give your job a name for easier identification (same as -J)
#SBATCH -o B5_copy_annotatesignlp2_slurm

########## Command Lines to Run ##########

module purge
conda activate funannotate

cd /mnt/research/Hausbeck_group/Lukasko/BotrytisDNASeq/CCR7/Predict_Annotate

for infile in B5_copy*

do

base=$(basename ${infile} _fun)

cd ${base}_fun

funannotate annotate -i predict_results \
--species botrytis_cinerea \
--iprscan predict_results/B5_ipr.xml \
--antismash predict_results/B5_smash/B5_smash.gbk \
--busco_db helotiales_odb10 \
--isolate B5 \
--cpus 24 \
--force

cd ../

done

conda deactivate

scontrol show job $SLURM_JOB_ID
```
*SignalP not currently working, use command above for right now.




