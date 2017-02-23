# SingleCellRNAseqTools
A collection of scripts for processing single-cell RNA-seq read data, including protocols that use single-cell library prep kits to sequence barcoded pools of thousands of cells.
##GetGeneCountsFromThreePrimeWellOverloadMIT.py
For some experiments, there is an interest in understanding expression variation among cell populations assigned to various conditions, but the amount of RNA obtained from those populations is too small to serve as input to standard bulk RNA-seq library preparation protocols. In such cases, and where there is less interest in cell-to-cell variation, one can use single-cell library preparation methods to build libraries from pools of cells. MIT's BioMicroCenter currently uses the protocol of Soumillion et al. (http://biorxiv.org/content/early/2014/03/05/003236) to generate such heaviliy 3'-end biased single-end RNA-seq libraries. Fastq files generated by this protocol produce reads with headers that contain (a) an index (identifying a set of plate rows) and a well id within those rows that together correspond to a specific sequenced pool of cells, and (b) a UMI. Under the assumption that a UMI should only map to the same genmoic location once, GetGeneCountsFromThreePrimeWellOverloadMIT.py takes a bwa-generated bam file and a bed file of gene boundaries for the reference genome, and extracts mRNA counts per gene. It requires samtools and bedtools to be in the path, either through a loaded module or via modifying the PATH environmental variable.

The first step is to map single-end reads with bwa. Once you have created a bwa genome index for the reference genome, a SLURM job submission script to Harvard's Odyssey cluster with two cmd line arguments (path to indexed reference fasta and fastq read file) would look something like this:

    #!/bin/bash
    #SBATCH -N 1
    #SBATCH -n 8
    #SBATCH -t 04:00:00
    #SBATCH -p serial_requeue
    #SBATCH --mem=16000
    #SBATCH -o bwa_%A.out
    #SBATCH -e bwa_%.err

    source new-modules.sh
    module purge
    module load bwa/0.7.9a-fasrc01

    # $1 = full path to reference fasta
    # $2 = fastq file

    ref=$1
    fastq=$2

    prefix=`echo $fastq |sed 's/.fq.gz//g'`
    bwa aln -t 8 -l 24 $ref $fastq > ${prefix}.sai
    bwa samse $ref ${prefix}.sai $fastq > ${prefix}.sam
    
    # NOTE: the amount of time required for the job, as specified by the -t argument, will vary depending upon the size of the file and the reference genome

**This code is a work in progress under active development, currently lacking many important bells and whistles!**
