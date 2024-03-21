# [Franco](https://github.com/altsplicer) / [***trimmomatic***]

## Overview
#This a script used for (fastq)fq and fq.gz files to trim reads using trimmomatic based on fastqc analysis.
#For a tutorial see [link](https://bioinformatics-core-shared-training.github.io/Bulk_RNAseq_Course_2021/Markdowns/S3_Trimming_Reads.html)

## Slurm script headers
UCI uses the SLURM scheduler so you must use slurm headers to specify how and where you want the job to run. 
You must also name the script SCRIPT_NAME.sub and then run the script using "Sbatch SCRIPT_NAME.sub" while your working directory is in location of the script being run. 

For a more in depth view on a SLURM job script headers see https://rcic.uci.edu/slurm/examples.html.

Make note that this script uses the free partition but you can use your free 1000 core hours of the lab's core hours by changing the header.
Otherwise your job can be killed in the free partition.
``` bash
#!/bin/bash
#SBATCH --job-name=Trimm_PE         # name of the job
#SBATCH -o /dfs3b/hertel-lab/fcarranz/project_name/Trimmed_PE/trim_qc_PE.out   # contains what would normally be printed to stdout
#SBATCH -e /dfs3b/hertel-lab/fcarranz/project_name/Trimmed_PE/trim_qc_PE.err   # file name to print standard error messages to. 
#SBATCH -p free # request cores 
#SBATCH --mail-user fcarranz@uci.edu         
#SBATCH --mail-type=ALL           # send you email of job status (b)egin, (e)rror, (a)bort, (s)uspend
``` 



## Setting up loop for trimmomatic
``` bash
#reads location (fq.gz)
DATA_DIR=/dfs3b/hertel-lab/fcarranz/project_name/raw_paired_end_reads

#data directory
DIR=/dfs3b/hertel-lab/fcarranz/project_name

## Where the Trimmed data should go
#trimmed data
TRIM_DATA_PE=${DIR}/PE_trimmed_data

#Trimmomatic package location
TRIMMOMATIC_DIR=/opt/rcic/Modules/modulefiles/GENOMICS/trimmomatic/0.39/trimmomatic-0.39.jar 

#make directory if it is not already made.
mkdir -p ${TRIM_DATA_PE}
```
## TRIMMOMATIC for paired end samples

Here we are performing a loop that will use each file whoese context matches the loop in the indicated directory.
Each file will be processed with the program trimmomatic, "\" symbol indicates that more options for the program are on the next line.
``` bash
#example of input fastq file names
#HBR and UHR refer to a test condition
#1 and 2 refers to replicates 
#R1 read1 (paired end)
#R2 read2 (paired end)
#HBR_1_R1.fq.gz
#HBR_1_R2.fq.gz
#HBR_2_R1.fq.gz
#HBR_2_R2.fq.gz
#HBR_3_R1.fq.gz
#HBR_3_R2.fq.gz
#UHR_1_R1.fq.gz
#UHR_1_R2.fq.gz
#UHR_2_R1.fq.gz
#UHR_2_R2.fq.gz
#UHR_3_R1.fq.gz
#UHR_3_R2.fq.gz

for SAMPLE in HBR UHR;
do
    for REPLICATE in 1 2 3;
    do
        # Build the name of the files.
		#R1=Read 1 file name and location
        R1=${DATA_DIR}/${SAMPLE}_${REPLICATE}_R1.fq.gz
		
		#R2=Read 2 file name and location
        R2=${DATA_DIR}/${SAMPLE}_${REPLICATE}_R2.fq.gz
		
		#OUTPUT= directory where trimmomatic output file should go and be named
        OUTPUT=${TRIM_DATA_PE}/${SAMPLE}_${REPLICATE}.fq.gz
		
		#Trimmer options
        TRIMMER="HEADCROP:1 LEADING:3 TRAILING:1 SLIDINGWINDOW:4:15 MINLEN:36"
        
		#trimmomatic execution
        java -jar ${TRIMMOMATIC_DIR} \
        PE \
        -threads 8 \
        ${R1} ${R2} \
        -baseout ${OUTPUT} \
        ${TRIMMER} 
     done
done
```
## Trimmer setting:
PE: input files are paired end reads.

-threads: # of threads to be used.

-baseout: where the output files should be placed.

HEADCROP: Cut the specified number of bases from the start of the read.

LEADING: Cut bases off the start of a read, if below a threshold quality (below quality 3).

TRAILING: Cut bases off the end of a read, if below a threshold quality (below quality 1).

SLIDINGWINDOW: Scan the read with a 4-base wide sliding window (windowSize), cutting when the average quality per base drops below 15 (requiredQuality).

MINLEN: Drop reads which are less than 36 bases long after these steps

# Please note if any of these commands are not sent in as slurm job but done in the terminal then you need to be in a interactive node, NOT THE LOGIN NODE! You will get in trouble with UCI's HPC staff. 
