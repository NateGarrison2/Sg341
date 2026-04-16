# MyGenome: Sg341

## Overview
This repository is for my Applied Bioinformatics Research class CS485G/ABT480, in which we are assembling and analyzing fungal genome data using computational tools and command-line interfaces.

# Procedures for Generating MyGenome Assemblies and Submitting Data to NCBI
## Download Datasets from the Farman Lab Mac
Use `scp` command to download data from Dr. Farman's Lab Mac onto personal VM.

## Assess Sequence Quality with FASTQC
We can run FASTQC on our forward and reverse sequences for quality assessment
```
fastqc Sg341_1.fq.gz
fastqc Sg341_2.fq.gz
```
By uploading the resulting .html files to a personal computer, we can see that there are several errors, especially showing that adapter content is high. 
<details>
  <summary>Summary of FASTQC results on forward/reverse reads</summary>

My Sg341_1.fq.gz contains warning levels for:
- Per base sequence content
- Per sequence GC content
- Overrepresented sequences
and an error level for Adapter Content

![Screenshot 2026-03-03 141544](https://github.com/user-attachments/assets/cb61ad33-86b6-45c2-b4a2-59e260740efd)

<img width="1110" height="600" alt="image" src="https://github.com/user-attachments/assets/3d80069f-dd83-461d-a08c-2cc750ad3485" />

My Sg341_2.fq.gz contains warning levels for:
- Per base sequence content
- Per sequence GC content
- Overrepresented sequences
- Per tile sequence quality
and an error level for Adapter Content

![Screenshot 2026-03-03 141838](https://github.com/user-attachments/assets/77a8d770-65e8-4019-89b8-42d9957f50b4)

<img width="1110" height="600" alt="image" src="https://github.com/user-attachments/assets/81391750-d237-4938-b0e8-98d17d278f0f" />
</details>


## Trim Adaptors and Poor Quality Sequence with Trimmomatic
Now that we have a good overview of our sequence quality and any issues within our data, we can use Trimmomatic to filter and trim our reads to address these issues.
1. Run Trimmomatic
```
java -jar trimmomatic-0.38.jar PE -threads 2 -phred33 -trimlog Sg341_errorlog.txt Sg341_1.fastq Sg341_2.fastq Sg341_1_paired.fastq Sg341_1_unpaired.fastq Sg341_2_paired.fastq Sg341_2_unpaired.fastq CROP:280 SLIDINGWINDOW:20:20 MINLEN:150
```
2. Run paired and unpaired sequences through FASTQC to assess the quality of trimming
```
fastqc Sg341_1_unpaired.fastq
fastqc Sg341_1_paired.fastq
fastqc Sg341_2_unpaired.fastq
fastqc Sg341_2_paired.fastq
```
<details><summary>Paired reads</summary>
My Sg341_1_paired.fastq contains warning levels for:
- Per sequence GC content
- Sequence Length Distribution
<img width="1917" height="980" alt="image" src="https://github.com/user-attachments/assets/2fed2193-dcf5-4e50-84a7-e896cdc7ae11" />
<img width="1110" height="600" alt="image" src="https://github.com/user-attachments/assets/aeaeef45-7940-42ba-99e8-33a99b3145b1" />


My Sg341_2_paired.fastq contains warning levels for:
- Per tile sequence quality
- Per sequence GC content
- Sequence Length Distribution
- Adapter Content

<img width="1919" height="975" alt="image" src="https://github.com/user-attachments/assets/402a1730-9dfb-4697-9b90-6e1a5f8096d4" />
<img width="1110" height="600" alt="image" src="https://github.com/user-attachments/assets/2c56cba5-c62b-43b3-9274-94987d84b9e0" />
</details>

<details><summary>Unpaired reads</summary>

My Sg341_1_unpaired.fastq contains warning levels for:
- Per tile sequence quality
- Per base sequence content
- Per sequence GC content
- Sequence Length Distribution
<img width="1919" height="981" alt="image" src="https://github.com/user-attachments/assets/2fc08aa7-b3c3-4fcf-83c4-c6e8ea5c8d06" />
<img width="1919" height="974" alt="image" src="https://github.com/user-attachments/assets/be9eaa03-5a24-409d-bd68-88cbac8f25cc" />


My Sg341_2_unpaired.fastq contains warning levels for:
- Per tile sequence quality
- Per base sequence content
- Per sequence GC content
- Sequence Length Distribution
- Adapter Content
<img width="1919" height="981" alt="image" src="https://github.com/user-attachments/assets/830304f7-1349-4bd6-b0ab-a883eaab830c" />
<img width="1918" height="973" alt="image" src="https://github.com/user-attachments/assets/bcedca33-2867-4b35-a9b8-7338a032714e" />
</details>

### Assessment
Trimmomatic did a great job of removing poor quality sequence in just about everything except the Illumina Universal Adapter content within the Sg341_2_unpaired.fastq file.

## Generate an Optimized MyGenome Assembly using Velvet and SPAdes
### Velvet
1. Use `scp` to transfer trimmed sequence reads from VM to MCC shell.
2. Determine optimal k-mer value for Velvet using [Velvet Advisor](https://dna.med.monash.edu/~torsten/velvet_advisor/)
3. Run VelvetOptimiser in steps of 10 (with a range of -40 to +40 from your Velvet Advisor k-mer as first and second arguments respectively) to find a more optimal k-mer value: 
```sbatch velvetoptimiser.sh Sg341 57 137 10```
4. Run VelvetOptimiser again in steps of 2 (range of -8 to +8) to find the most optimal k-mer value based on results: 
```sbatch velvetoptimiser.sh Sg341 89 105 2```

### Results from log file of 2-step VelvetOptimiser:
- Optimal k-mer value: 99
- Genome size: 43,718,235
- Number of contigs: 3,071
- N50: 49,683

### SPAdes
1. Run spades.sh script on paired reads: `sbatch Spades-paired.sh . Sg341`
2. Use command line (grep, awk, etc.) to find # of contigs, final genome size, and N50 value for final genome assembly

### Results from SPAdes:
- Genome size: 44,486,013
- Number of contigs: 11,215
- N50: 55,362

### Bandage visualization of optimized assembly
Bandage is a graphical tool that allows a user to interrogate an assembly graph to examine entire assemblies or individual nodes and their neighbors.
I used my Graph2 file generated by Velvet.
<details><summary>Picture of full assembly:</summary>

![Graph2FullAssembly](https://github.com/user-attachments/assets/0e6d2a0d-20ff-4927-af77-5944e5c4916e)
</details>

<details><summary>Picture of randomly selected node:</summary>

![Graph2Node11007](https://github.com/user-attachments/assets/4308e11e-df87-4ab2-8afd-7c1be05ede56)
</details>

## Perform Genome Post Processing for NCBI Submission
Firstly, we run the SimpleFastaHeaders.pl script to standardize sequence header formats so it is easier to read the unique contig ID field. Secondly, we run the GenomePostProcess.sh script to rename sequence headers, identify adaptor contamination, remove contaminating sequences and cull sequences less than 200 nt in length.
```
perl SimpleFastaHeaders.pl Sg341_spades_assembly/scaffolds.fa Sg341
sbatch GenomePostProcess.sh Sg341_newheader.fasta
```
In the end, we get a finalized genome assembly named Sg341_final.fasta
## Assess Genome Quality using BUSCO
BUSCO (Benchmarking Using Single-Copy Orthologs) is a tool used to assess genome completeness by using BLAST to search for a set of single-copy genes which have "orthologs" in genomes of related species.
We run BUSCO on our fully optimized, fully cleaned genome assembly (`Sg341_final.fasta`) by running: ```sbatch BuscoSingularity.sh Sg341_final.fasta```
This outputs a `Sg341_final_busco` directory with a `short_summary` file inside with results.
## Genome Interrogation using BLAST
We need to determine which contigs in the final assembly correspond to the mitochondrial genome in preparation to submit to NCBI.
1. BLAST the `MoMitochondrion.fasta` sequence against the final genome assembly:
```singularity run --app blast2120 /share/singularity/images/ccs/conda/amd-conda1-centos8.sinf blastn -query MoMitochondrion.fasta -subject Sg341_final.fasta -evalue 1e-50 -max_target_seqs 20000 -outfmt '6 qseqid sseqid slen length qstart qend sstart send btop' -out MoMitochondrion.Sg341.BLAST```
2. Export a list of contigs which comprise of mitochondrial sequences to upload along with our genome assembly in NCBI:
```awk '$4/$3 >= 0.9 {print $2 ",mitochondrion"}' MoMitochondrion.Sg341.BLAST > Sg341_mitochondrion.csv```
3. Export the blast results that did not pass the above filter into a separate .txt file:
```awk '$4/$3 < 0.9' MoMitochondrion.Sg341.BLAST > Sg341_short_mitochondrial_hits.txt```
4. Manually interrogate the `Sg341_short_mitochondrial_hits.txt` file to look for blast alignments that got split in half that combined would cross the 90% threshold, and add those contigs to the bottom of the `Sg341_mitochondrion.csv` file.
## Perform Gene Predictions

## Visualize Genes in Genome Browser
