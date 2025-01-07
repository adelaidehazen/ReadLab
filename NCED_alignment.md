## Aligning the fastq files with the 5B NCED reference

I got nextflow to work by first installing java and then installing nextflow where I wanted it (changing to the directory of interest

```
curl -s https://get.sdkman.io | bash
```
at this point I had to open a new terminal to continue 
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


Now I am going to try the barcoded samples and see what happens

```
nextflow run epi2me-labs/wf-alignment \
	--fastq 'barcoded' \
	--references 'reference/NCED_5B_Promoter.fa' \
  --sample_sheet 'barcodeID2.csv' \
	-profile standard



