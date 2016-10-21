## Lab 8: Bacterial Genome Assembly



During this lab, we will acquaint ourselves with Genome Assembly using SPAdes.

<del>1. Install software and download data</del>

<del>2. Error correct, quality and adapter trim data sets.</del>

3. Assemble



The SPAdes manuscript:


Step 1: Launch and AMI. For this exercise, we will use a <span style="color: #ff0000;"><strong>c4.2xlarge and 50Gb HD space.</strong></span> Remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)

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
brew tap Gen711/homebrew-science
brew update
brew doctor


```

Install a Python plotting package



```bash  

brew install gcc openmpi spades abyss bwa bowtie2 samtools bedtools

```

```bash
pip install matplotlib
```

```bash  

brew install quast

```


Download and unpack the data

```bash
mkdir ~/reads
cd ~/reads
wget https://s3.amazonaws.com/gen711/ecoli_data.tar.gz
tar -zxf ecoli_data.tar.gz
```


Assembly. Try this with different data combos (with mate pair data, without, with minION data and without, etc). Remember to name your assemblies something different using the `-o` flag. Spades has a built-in error correction tool (remove `--only-assembler`). Does 'double error correction seem to make a difference?'.

```bash

mkdir ~/spades
cd ~/spades

spades.py -t 8 -m 15 --only-assembler --mp1-rf -k 95 \
--pe1-1 ~/reads/ecoli_pe.1.fq \
--pe1-2 ~/reads/ecoli_pe.2.fq \
--mp1-1 ~/reads/nextera.1.fq \
--mp1-2 ~/reads/nextera.2.fq \
--pacbio ~/reads/minion.data.fasta \
-o Ecoli_all_data
```



Assembling with ABySS


```bash
mkdir ~/abyss
cd ~/abyss

abyss-pe np=8 k=95 name=ecoli lib='pe1' mp='mp1' long='minion' \
pe1='~/reads/ecoli_pe.1.fq ~/reads/ecoli_pe.2.fq' \
mp1='~/reads/nextera.1.fq ~/reads/nextera.2.fq' \
minion='~/reads/minion.data.fasta' mp1_l=30
```

quast

```bash

mkdir ~/quast
cd ~/quast

curl -LO ftp://ftp.ensemblgenomes.org/pub/release-32/bacteria//fasta/bacteria_91_collection/escherichia_coli/dna/Escherichia_coli.HUSEC2011CHR1.dna_rm.toplevel.fa.gz

quast ~/spades/Ecoli_all_data/scaffolds.fasta ~/abyss/ecoli-long-scaffs.fa \
        -R  Escherichia_coli.HUSEC2011CHR1.dna_rm.toplevel.fa.gz \
        -o quast_output --threads 8
```

open a new terminal tab, download the data and view

```
scp -ri your.pem ubuntu@??.???.??.??:/home/ubuntu/quast/quast_output ~/Downloads
```

### Which assembly is better??

# Terminate your instance
