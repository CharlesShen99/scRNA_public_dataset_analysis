# scRNA_public_dataset_analysis
Public scRNA datasets download and analysis

This is the public database under EMBL-EBI: https://www.ebi.ac.uk/biostudies/arrayexpress/studies

Search for the accession number you have (e.g., E-MTAB-8906)

Click on “View Table”

Click on the first FASTAQ and the second FASTAQ for Sample 1: 13__Ccl19-EYFP_naive_LN_S17_L001

"L001" means Lane 1, and there should be Read1 and Read2

After downloading, we should have L_001_R1.fastq and L_001_R2.fastq files

Read1 file should contain cell barcodes and UMI, and Read2 file should contain cDNA sequences = transcript reads

Because I am using macOS and Cell Ranger needs to run on Linux, I use kallisto | bustools developed by the Pachter Lab 

In the terminal, run this to create a virtual environment

``` R
conda create -n scrna-tools -c bioconda -c conda-forge kallisto bustools
conda activate scrna-tools
```

Download the GRC38=mm10 transcriptomes fasta file from Gencode
```
wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M25/gencode.vM25.transcripts.fa.gz
gunzip gencode.vM25.transcripts.fa.gz
```

Build kallisto index
```R
kallisto index -i mouse_transcriptome.idx gencode.vM25.transcripts.fa

```


