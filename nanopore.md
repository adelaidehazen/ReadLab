## Learning to Process Nanopore Outputs

`````
#!/bin/bash -l
#SBATCH -p v100                                             
#SBATCH --gres=gpu:v100:1
#SBATCH --time=1:00:00
#SBATCH --ntasks=5
#SBATCH --mem=40g
#SBATCH --tmp=32g
#SBATCH --job-name=Basecall_class

mkdir basecalled

module load guppy/3.2.4

#the command to call the Nanopore basecaller
/common/software/install/migrated/guppy/4.2.2-gpu/bin/guppy_basecaller -i fast5  -r -s basecalled \
  --config dna_r9.4.1_450bps_hac_prom.cfg  --device CUDA:0
`````
This is the main text that I used for today's activity. 
Guppy is the module which converts the fast5 files to fastq files. 

I learned to use vim
`````
vim 'filename'.sh
`````

I learned to log into the msi
`````
ssh hazen039@mangi.msi.umn.edu
`````

Scratch folder files are DELETED after 30 days  
`````
chmod 777
`````
gives permision to everyone for everything

`````
squeue -u hazen039
`````
this prints out the status of what is running

##Logging back in on 10.6.24 to work through the lab

vim commands: 
`````
#save and exit
:wq 
#insert mode
i
#command mode
[esc]
'''''

Today I made it through the lab until the last step where I got the error "samtools: error while loading shared libraries: libcrypto.so.10: cannot open shared object file: No such file or directory"
