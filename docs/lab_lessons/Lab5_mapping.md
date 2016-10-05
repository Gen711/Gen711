## Lab 5: Read Mapping


During this lab, we will acquaint ourselves with de novo transcriptome assembly using Trinity. You will:

1. Install software and download data

2. Use sra-toolkit to extract fastQ reads

3. Map reads to dataset

4. look at mapping quality


The BWA manual: http://bio-bwa.sourceforge.net/



Step 1: Launch and AMI. For this exercise, we will use a c4.2xlarge machine. Add 100Gb storage.


Update Software


```bash
sudo apt-get update && sudo apt-get -y upgrade
```

Install other software

```bash
sudo apt-get -y install build-essential git
```


Install LinuxBrew

```bash

cd
wget https://keybase.io/mpapis/key.asc
gpg --import key.asc
\curl -sSL https://get.rvm.io | bash -s stable --ruby
source /home/ubuntu/.rvm/scripts/rvm


sudo mkdir /home/linuxbrew
sudo chown $USER:$USER /home/linuxbrew
git clone https://github.com/Linuxbrew/brew.git /home/linuxbrew/.linuxbrew
echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"' >> ~/.profile
echo 'export MANPATH="/home/linuxbrew/.linuxbrew/share/man:$MANPATH"' >> ~/.profile
echo 'export INFOPATH="/home/linuxbrew/.linuxbrew/share/info:$INFOPATH"' >> ~/.profile
source ~/.profile
brew tap homebrew/science
brew update
brew doctor
```

Install software

```bash
brew install bwa samtools aria2 salmon sratoolkit
```


Download data

```bash
mkdir $HOME/data
cd $HOME/data
aria2c http://datadryad.org/bitstream/handle/10255/dryad.72141/brain.final.fasta
aria2c ftp://ftp-trace.ncbi.nlm.nih.gov/sra/sra-instant/reads/ByRun/sra/SRR/SRR157/SRR1575395/SRR1575395.sra

```



```bash
cd $HOME/data
fastq-dump --split-files --split-spot SRR1575395.sra
```

Map reads!! (17 minutes). You're mapping to a mouse brain transcriptome reference.

```bash
mkdir $HOME/mapping
cd $HOME/mapping


tmux new -s mapping


bwa index -p index $HOME/data/brain.final.fasta
time bwa mem -t8 index $HOME/data/SRR1575395_1.fastq $HOME/data/SRR1575395_2.fastq | samtools view -1 -o brain.bam -@ 4 -T $HOME/data/brain.final.fasta -

```

Look at BAM file. Can you see the columns that we talked about in class?


```bash
#Take a quick general look.

samtools view brain.bam | head
```


look at mapping stats. Figure out what this means.

```bash
samtools flagstat brain.bam
```

Map with salmon

```
salmon index --type quasi --threads 8 --index brain.idx -t $HOME/data/brain.final.fasta
salmon quant --gcBias --seqBias -p 8 -i brain.idx -l a -1 $HOME/data/SRR1575395_1.fastq -2  $HOME/data/SRR1575395_2.fastq -o salmon_brain
```


Lastly, look at the Salmon output using the command `more salmon_brain/quant.sf`. Pick a random transcript (column 1), and note the number of fragments that map to it (this is in col. 5). Let's see how concordant this number is with the BWA output.

To count the number of reads mapping to a specific transcript, we need to search the BAM/SAM file for the name. Try the code:

```bash
samtools view brain.bam | awk '{print $3}' | grep -c NAME_OF_YOUR_TRANSCRIPT
```

Make sure you know what this command is doing, rather than just copy/pasting..

# TERMINATE YOUR INSTANCE
