# scRNA_public_dataset_analysis
Public scRNA datasets download and analysis

This is the public database under EMBL-EBI: https://www.ebi.ac.uk/biostudies/arrayexpress/studies

Search for the accession number you have (e.g., E-MTAB-8906)

Click on “View Table”

Click on the first FASTAQ and the second FASTAQ for Sample 1: 13__Ccl19-EYFP_naive_LN_S17_L001

"L001" means Lane 1, and there should be Read1 and Read2

"L001 I1" is the index read file, which contains 8bp-long sample barcodes.

After downloading, we should have L_001_R1.fastq and L_001_R2.fastq files

Read1 file should contain cell barcodes and UMI, and Read2 file should contain cDNA sequences = transcript reads

Because I am using macOS and Cell Ranger needs to run on Linux, I use kallisto | bustools developed by the Pachter Lab 

In the terminal, run this to create a virtual environment

``` bash
conda create -n scrna-tools -c bioconda -c conda-forge kallisto bustools
conda activate scrna-tools
```

Download the GRC38=mm10 transcriptomes fasta file from Gencode
```
wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M25/gencode.vM25.transcripts.fa.gz
gunzip gencode.vM25.transcripts.fa.gz
```

Build kallisto index
```bash
kallisto index -i mouse_mm10_transcriptome.idx gencode.vM25.transcripts.fa

```

v2 is used because the paper used Chromium Single Cell 3’ Reagent Kit (v2  Chemistry)
Pseudoalignment
```
kallisto bus -i mouse_mm10_transcriptome.idx \
  -o output \
  -x 10xv2 \
  -t 4 \
  L001_R1.fastq L001_R2.fastq
```

```
# Create whitelist
bustools whitelist -o output/whitelist.txt output/output.bus

# Correct barcodes
bustools correct -w output/whitelist.txt -o output/corrected.bus output/output.bus

# Sort corrected bus file
mkdir -p output/tmp
bustools sort -T output/tmp -o output/sorted.bus output/corrected.bus
```

Obtain the GENCODE transcript to gene annotations
```
wget ftp://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M25/gencode.vM25.annotation.gtf.gz
awk '$3 == "transcript"' gencode.vM25.annotation.gtf | \
  awk '{ match($0, /transcript_id "([^"]+)"/); transcript = substr($0, RSTART+13, RLENGTH-14); match($0, /gene_id "([^"]+)"/); gene = substr($0, RSTART+8, RLENGTH-9); print transcript "\t" gene }' > transcripts_to_genes.txt
```

Make the transcripts_to_genes_clean by removing extra "
```
sed 's/"//g; s/^[[:space:]]*//; s/[[:space:]]*$//' transcripts_to_genes.txt > transcripts_to_genes_clean.txt
```

Extract Transcript IDs 
```
cut -d '|' -f1 output/transcripts.txt > output/transcripts_clean.txt
```

```
bustools count \
  -o output/counts \
  -g transcripts_to_genes_clean.txt \
  -e output/matrix.ec \
  -t output/transcripts_clean.txt \
  --genecounts \
  output/sorted.bus
```



