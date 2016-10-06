## MOCK EXAM 1


To help you prepare for the exam practical, let's do a mock exam. These are the types of questions I will ask you during the exam. I will give you points for each section, with the idea that I have to help you get past a particular section, you lose those points, but if you can do the rest of the steps, you can still get credit for that.

&nbsp;

1. Launch a ``c4.2xlarge`` instance.  _______ 5 points

&nbsp;

2. Update your machine and install the software (`build-essential` and `git`) using ``apt-get`` just like we did in lab 2,3,4,5. _________ 5 points

&nbsp;




3. Install `LinuxBrew` and `skewer` and `jellyfish` and `aria2` and `seqtk` like lab 4. ___________________ 10 points.

&nbsp;




4. Download the datasets located here: `https://s3.amazonaws.com/gen711/TruSeq3-PE.fa`, `https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R1.PF.fastq.gz` and `https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R2.PF.fastq.gz`.  _________________ 5 points

&nbsp;

5. Trim and count kmers from the dataset like we did in Lab 4 using a Phred score of 28.  ____________________ 10 points

&nbsp;

6. Count the unique kmers left using the command `head -1 28.counts.histo` and put the number you get here _______________ 15 points (64484989 is answer)
