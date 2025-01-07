## Aligning the fastq files with the 4B NCED reference

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
```

Now I am continuing with the workflow
mkdir -p $Desktop/nced_align/coding
mv nextflow $Desktop/nced_align 

