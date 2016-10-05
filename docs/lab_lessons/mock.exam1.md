## MOCK EXAM 1


To help you prepare for the exam practical, let's do a mock exam. These are the types of questions I will ask you during the exam. I will give you points for each section, with the idea that I have to help you get past a particular section, you lose those points, but if you can do the rest of the steps, you can still get credit for that.



1. Launch a ``c4.2xlarge`` instance.  _______ 5 points



2. Update your machine and install the software using ``apt-get`` just like we did in lab 2,3,4,5. _________ 5 points


3. Install `LinuxBrew` and `skewer` and `jellyfish` like lab 4. ___________________ 10 points.


4. Download the datasets located here: `https://s3.amazonaws.com/gen711/TruSeq3-PE.fa`, `https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R1.PF.fastq.gz` and `https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R2.PF.fastq.gz`.  _________________ 5 points



5. Trim the dataset like we did in Lab 4 using a Phred score of 28.  ____________________ 10 points



6. Count the remaining reads using the command `zgrep -c @HWI skewer-trimmed-pair1.fastq` and put the number you get here _______________ 15 points