## Aligning the fastq files with the 5B NCED reference

I got nextflow to work by first installing java and then installing nextflow where I wanted it (changing to the directory of interest. This comes from: https://github.com/epi2me-labs/wf-alignment/tree/master

```
curl -s https://get.sdkman.io | bash
```
at this point I had to open a new terminal to continue 
https://www.youtube.com/watch?v=GfWkLo5vzME 
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
docker run -ti -v $(pwd):$(pwd) -w $(pwd) quay.io/biocontainers/qcat:1.0.0--py_0 qcat -f barcoded/bc1/38BT8D_5_BC1_2.fastq --kit SQK-RBK004 --output barcoded/bc1/output
```




