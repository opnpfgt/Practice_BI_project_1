# Project1 Lab

## 0. Install all needed programs for MacOS/Windows with HomeBrew/apt get/conda/mamba (Seqkit, bwa, fastqc, Trimmomatic,
varscan, samtools, snpeff)

```Zsh
brew install seqkit
which seqkit #/opt/homebrew/bin/seqkit
brew install fastqc
which fastqc #/opt/homebrew/bin/fastqc
fastqc -h #get help
brew install Trimmomatic #/opt/homebrew/opt/trimmomatic
Trimmomatic PE #
brew install VarScan
varscan -h #help
# mpileup2snp Identify SNPs from an mpileup file
brew install bwa
bwa #to see avaliable commands
brew install samtools
brew install snpeff #didn't work
```

or much better

```Zsh
conda create -n prac_project_1
conda activate prac_project_1
```

```Zsh
conda install -c bioconda seqkit
conda install bioconda::fastqc
fastqc -h #get help
conda install bioconda::trimmomatic
trimmomatic PE 
conda install bioconda::varscan
varscan -h #help
# mpileup2snp Identify SNPs from an mpileup file
conda install bioconda::bwa
bwa #to see avaliable commands
conda install bioconda::samtools
conda install bioconda::snpeff
```


## 1. Check our files with manually and with the help of Seqkit to get summarised information about contigs and the reference
genome:


```Zsh
wc -l amp_res_1.fastq amp_res_2.fastq
  1823504 amp_res_1.fastq
  1823504 amp_res_2.fastq
  3647008 total
```

```Zsh
expr 1823504 / 4
455876
```

```Zsh
expr 1823504 / 4
455876
```

```Zsh
seqkit stats amp_res_1.fastq
file             format  type  num_seqs     sum_len  min_len  avg_len  max_len
amp_res_1.fastq  FASTQ   DNA    455,876  46,043,476      101      101      101
```

```Zsh
seqkit stats amp_res_2.fastq
file             format  type  num_seqs     sum_len  min_len  avg_len  max_len
amp_res_2.fastq  FASTQ   DNA    455,876  46,043,476      101      101      101
```

## 2. Run fastqc

```Zsh
fastqc -o . amp_res_1.fastq amp_res_2.fastq
ls #to see the generated files in rowdata
```

2 files were generated:
amp_res_2.fastq.html
amp_res_1.fastq.html

Manual statistics match seqkit stats and fastqc report.

The report shows that we have significant quality drops in "Per base sequence quality" with length more 
than 80 in both direcrions and "Per tile sequence quality" for forward reads.

## 3. Trimmomatic

Next, we performed 1 step trimming to get rid of low-quality bases from the reads using
trimmomatic with following parameters and Phred+33 quality scale:

HEADCROP:20 cutting first 20 bases in every read

TRAIL:20 cutting 20 bases at the end

SLIDINGWINDOW:10:20 cutting with sliding window method

MINLEN:20 ignore reads with length<20


```Zsh
trimmomatic PE amp_res_1.fastq amp_res_2.fastq amp_res_1_paired.fastq amp_res_1_unpaired.fastq
amp_res_2_paired.fastq amp_res_2_unpaired.fastq HEADCROP:20 TRAILING:20 SLIDINGWINDOW:10:20 MINLEN:20
```
Output
Input Read Pairs: 455876 Both Surviving: 431982 (94.76%) Forward Only Surviving: 15864 (3.48%) 
Reverse Only Surviving: 5326 (1.17%) Dropped: 2704 (0.59%)

4 files were generated:

amp_res_1_unpaired.fastq
amp_res_2_paired.fastq
amp_res_1_paired.fastq
amp_res_2_unpaired.fastq



We dropped small portion of sequences and significantly increased the quality. 
At the same time we lost just 2704 (0.59%) reads. And saved 431982 paired reads.
1) We dropped short sequnces with lengh less than 20. That's gonna help us to allign our reads properly.
2) There are no more sequnces with quality lower than 20.
3) Summary and graphics looks good.
4) Sequence legth distribution were moved to the greater values which is not so critical.

Lets try to make it better with quality thresholds 30.

```Zsh
fastqc -o . amp_res_1_paired.fastq amp_res_2_paired.fastq
```

2 files were generated:
amp_res_2_paired.fastq.html
amp_res_1_paired.fastq.html

```Zsh
trimmomatic PE amp_res_1.fastq amp_
res_2.fastq amp_res_1_paired_30.fastq amp_res_1_unpaired_30.fastq amp_res_2_paired_30.fastq amp_res_2_unpaired_30.fastq
HEADCROP:30 TRAILING:30 SLIDINGWINDOW:10:30 MINLEN:20
```

Output
Input Read Pairs: 455876 Both Surviving: 338000 (74.14%) Forward Only Surviving: 42173 (9.25%) 
Reverse Only Surviving: 29333 (6.43%) Dropped: 46370 (10.17%)

4 files were generated:

amp_res_1_unpaired_30.fastq
amp_res_2_paired_30.fastq
amp_res_1_paired_30.fastq
amp_res_2_unpaired_30.fastq

```Zsh
fastqc -o . amp_res_1_paired_30.fastq amp_res_2_paired_30.fastq
```

2 files were generated:
amp_res_2_paired_30.fastq.html
amp_res_1_paired_30.fastq.html

Obtained results show that we lost around 25% of paired reads with only 338000 (74.14%) left, which means that we can lose some SNPs.
1) Per tile sequence quality became even worse in case of forward reads.
2) Other statustics became better, but we still may lack the information we need.

Let's move further with the basic quality score around 20 (Despite the recomended minlength usually 
holded around 36)

## 4. Indexing and aligning

Index our reference sequence and align or sequences to reference genome:

```Zsh
bwa index GCF_000005845.2_ASM584v2_genomic.fna.gz
bwa mem GCF_000005845.2_ASM584v2_genomic.fna.gz amp_res_1_paired_1.fastq
amp_res_2_paired_1.fastq > alignment.sam
```

## 5. Compress SAM file - SAM to BAM format

```Zsh
samtools view -S -b alignment.sam > alignment.bam
```

-b: means convert to .bam

Get statistics

```Zsh
samtools flagstat alignment.bam
#result
864052 + 0 in total (QC-passed reads + QC-failed reads) #all passed QC test
863964 + 0 primary # Out of the total reads, 863,964 are classified as
primary alignments. Primary alignments are considered the best alignments
for a given read if multiple alignments are present.
0 + 0 secondary #Indicates there are no secondary alignments. Secondary
alignments happen when a read aligns to multiple locations.
88 + 0 supplementary #These are part of reads that align to more than one
location
0 + 0 duplicates
0 + 0 primary duplicates
860382 + 0 mapped (99.58% : N/A) # Out of all the reads, 860,382 are mapped
to the reference genome, which constitutes 99.58% of the total reads.
860294 + 0 primary mapped (99.58% : N/A)
863964 + 0 paired in sequencing # Indicates that all sequences come from
paired-end sequencing, equally divided between read1 and read2.
431982 + 0 read1
431982 + 0 read2 #This indicates there are 431,982 reads for both READ1 and
READ2, highlighting the paired-end nature of the sequencing.
857350 + 0 properly paired (99.23% : N/A) #A large majority, 857,350 of the
paired reads, are mapped in a proper pair, signifying that they are
appropriately oriented and spaced according to the sequencing protocol.
858718 + 0 with itself and mate mapped
1576 + 0 singletons (0.18% : N/A) #A small number, 1,576 of the paired
reads, are singletons, meaning only one read from the pair is mapped to the
reference genome.
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5) #Indicates that no reads
have their mate mapped to a different chromosome, including those with high
mapping quality (mapQ>=5).
```


## 6. Sorting and indexing BAM

BAM files have to be sorted and indexed.

### Sorting

```Zsh
samtools sort alignment.bam -o alignment_sorted.bam
ls #to see the generated files
```

-o: means output

Generated files:
```Zsh
GCF_000005845.2_ASM584v2_genomic.fna.gz.amb
GCF_000005845.2_ASM584v2_genomic.fna.gz.ann
GCF_000005845.2_ASM584v2_genomic.fna.gz.bwt
GCF_000005845.2_ASM584v2_genomic.fna.gz.pac
GCF_000005845.2_ASM584v2_genomic.fna.gz.sa
alignment_sorted.bam was generated. 
```

### Indexing

Then index the file

```Zsh
samtools index alignment_sorted.bam
ls #to see the generated files
```
Generated file:
```Zsh
alignment_sorted.bam.bai #file with indexes
```

## 7. Visualize it with IGV browser

Only aligning resuts gonna be shown, w/o SNPs.


## 8. Make a my.mpileup file

```Zsh
samtools mpileup -f GCF_000005845.2_ASM584v2_genomic.fna alignment_sorted.bam > my.mpileup
```

-f: needed to indicate reference genome file.

## 9. Run VarScan

```Zsh
#output in terminal
Only SNPs will be reported
Warning: No p-value threshold provided, so p-values will not be calculated
Min coverage: 8
Min reads2: 2
Min var freq: 0.5
Min avg qual: 15
P-value thresh: 0.01
Reading input from my.mpileup
4640963 bases in pileup file
7 variant positions (6 SNP, 1 indel)
1 were failed by the strand-filter
5 variant positions reported (5 SNP, 0 indel) #We caught you guuuuuys
```

Here is the table with detected SNPs

| № |     Name    | Position|Ref|Alt| Qual|
|---|-------------|---------|---|---|-----|
| 1 | NC_000913.3 | 93043   | C | G | PASS|
| 2 | NC_000913.3 | 482698  | T | A | PASS|
| 3 | NC_000913.3 | 852762  | A | G | PASS|
| 4 | NC_000913.3 | 3535147 | A | C | PASS|
| 5 | NC_000913.3 | 4390754 | G | T | PASS|

Great! We obtained 5 representative SNP positions.
2 of 7 "snips" didn't pass. 1 of them is indel.

## 10. snpEff
Let's get started with SNPeff.
snpEff 4.3t version
That's much easier to download this program via conda.

1) We need to download sequnce and annotation of our reference

```Zsh
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/GCF_000005845.2_ASM584v2_genomic.gbff.gz
```
2) Create an empty text file snpEff.config, and add there just one string:

```Zsh
echo k12.genome : ecoli_K12 > snpEff.config
```

3) Create a folder for the database and copy unzipped DB to this folder:

```Zsh
mkdir -p data/k12
gunzip GCF_000005845.2_ASM584v2_genomic.gbff.gz
cp GCF_000005845.2_ASM584v2_genomic.gbff data/k12/genes.gbk
```

4) Create database:

```Zsh
snpEff build -genbank -v k12
```

5) Annotate:
```Zsh
snpEff ann k12 VarScan_results.vcf > VarScan_results_annotated.vcf
```

## Again visualize it with IGV and load
Here we need
- sorted .bam file
- inzipped reference genome
- .gff file with annotations
- annotated .vcf file


As a result, you will obtain a vcf file with additional field "ANN" (for "annotation"), describing all the effects for each SNP.
Also you may fond snp description in IGV, you just need to click on it and tou will find aaaall the information you need.

Good luck!
