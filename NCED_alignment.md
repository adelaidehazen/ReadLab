## Aligning the fastq files with the 5B NCED reference

I got nextflow to work by first installing java and then installing nextflow where I wanted it (changing to the directory of interest. This comes from: https://github.com/epi2me-labs/wf-alignment/tree/master

```
curl -s https://get.sdkman.io | bash
```
at this point I had to open a new terminal to continue 
https://www.youtube.com/watch?v=GfWkLo5vzME 
https://github.com/boulderrinnlab/CLASS_2023/blob/master/CLASSES/03_Nextflow/01_installing_nextflow.Rmd
```
sdk install java 17.0.10-tem
java -version
cd nced_align
curl -s https://get.nextflow.io | bash
sudo nano /etc/paths.d/nextflow
/Users/adelaide/desktop/installs/

```

Now I am continuing with the workflow
A nested folder "fastq" contains all the input files
A nested folder "reference" contains the reference file

```
nextflow run epi2me-labs/wf-alignment \
	--fastq 'fastq/38BT8D_4_BC4.fastq' \
	--references 'reference/NCED_5B_Promoter.fa' \
	-profile standard
```
I am storing all of the outputs in seperate folders named "BARCODE_output"

## Trying to seperate out barcodes
Now I am going to try the barcoded samples and see what happens

```
nextflow run epi2me-labs/wf-alignment \
	--fastq 'barcoded' \
	--references 'reference/NCED_5B_Promoter.fa' \
  	--sample_sheet 'barcodeID2.csv' \
	-profile standard
```
So, it seems that the barcoding does not search for barcodes within the sequences I feed it, but rather they already need to be sorted by barcode

I am going to try something called qcat: https://github.com/nanoporetech/qcat 

```
docker run -ti -v `pwd`:`pwd` -w `pwd` quay.io/biocontainers/qcat:1.0.0--py_0 qcat -f barcoded/bc1/38BT8D_5_BC1_2.fastq -b barcoded/bc1/output

```
This ran, but it separated a file containing 2 barcodes into 5 different barcodes and then most of the reads were lumped into "none"
Copilot gave me this pointer: 
```
qcat -f <fastq_file> -b <output_folder> -b <barcode1> -b <barcode2> ...
```
so I am going to run this
```
docker run -ti -v `pwd`:`pwd` -w `pwd` quay.io/biocontainers/qcat:1.0.0--py_0 qcat -f barcoded/bc1/38BT8D_5_BC1_2.fastq --barcode barcoded/bc1/output --barcode CACAAAGACACCGACAACTTTCTT --barcode ACAGACGACTACAAACGGAATCGA

```
This did the exact same thing but put it into a folder named after one of my DNA sequences. 
Copilot is telling me I need to include the barcode kit name for the specific sequences like this: (replacing the kit name with the specific kit we used for barcodes)
```
docker run -ti -v $(pwd):$(pwd) -w $(pwd) quay.io/biocontainers/qcat:1.0.0--py_0 qcat -f barcoded/bc1/38BT8D_5_BC1_2.fastq --kit PBC096 --output barcoded/bc1/output
```
Turns out, the barcodes we used are not exactly as the kit is, but there seems to be an easy fix according to copilot. I will make a custom barcode file: a txt file with each line as a specific barcode. 
```
docker run -ti -v `pwd`:`pwd` -w `pwd` quay.io/biocontainers/qcat:1.0.0--py_0 qcat -f 38BT8D_5_BC1_2.fastq -b barcodes --output output
```

## 5B and 5D Alignment

To do this, I will first create a new fasta file which is a concatenation of the two copies of the NCED promoter. Then I will run the original pipeline using the new concatenated pipeline. the B genome is the first sequence and the D genome is after

```
nextflow run epi2me-labs/wf-alignment \
	--fastq 'fastq/38BT8D_1_BC1.fastq' \
	--references 'reference/NCED_5Dand5B_Promoter.fa' \
	-profile standard
```
When I ran this, my reads seemed to be much more accurately mapped and there was a way higher 
I am going to run just the 5D promoter next to see what happens
```
nextflow run epi2me-labs/wf-alignment \
	--fastq 'fastq/38BT8D_1_BC1.fastq' \
	--references 'reference/NCED_5D_Promoter.fa' \
	-profile standard
```

## Mapping Unmapped Reads, Using Andy's Github Script: 

First I used FileZilla to add the Fastq files to be mapped, and the .fa file for the NCED promoter regions. Then I ran all these lines: 
```
ssh hazen039@mangi.msi.umn.edu

#!/bin/bash -l
#SBATCH -p v100                                             
#SBATCH --gres=gpu:v100:1
#SBATCH --time=1:00:00
#SBATCH --ntasks=5
#SBATCH --mem=40g
#SBATCH --tmp=32g
#SBATCH --job-name=Basecall_class

mkdir basecalled
module load samtools/1.14
module load minimap2/2.17
minimap2 NCED_5Dand5B_Promoter.fa  * 38BT8D_1_BC1.fastq -ax map-ont > BC1.sam
samtools view -bS BC1.sam > BC1.bam
samtools sort BC1.bam -o BC1_sorted.bam
module load samtools
samtools view BC1.sam -f 4 > BC1_unmapped.txt

```
This worked! Turns out, reads are mapping to the 3B chromosome...
Now I am going to repeat this for all the barcodes to see if they are all doing the same thing 




