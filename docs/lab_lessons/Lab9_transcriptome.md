### Lab 9: Transcriptome assembly


During this lab, we will acquaint ourselves with Transcriptome Assembly using SPAdes, then, we will run BUSCO and TransRate


Step 1: Launch and AMI. For this exercise, we will use a <span style="color: #ff0000;"><strong>c4.4xlarge and 50Gb HD space.</strong></span> IMPORTNT: AWS changed the default machine type for UBUNTU, we want our old machine (Ubuntu 14.04), which is still there, but you have schroll to the bottom of the page to find it.

```bash
sudo apt-get update && sudo apt-get -y upgrade && sudo apt-get -y dist-upgrade
```
Install other software

```bash
sudo apt-get -y install build-essential git python-pip python-numpy python-matplotlib
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


##### Install software


```bash  
brew install gcc spades hmmer cmake
```
##### Install BioPython


```bash
pip install biopython
```

##### Install Blast

```bash  
cd
curl -LO ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/ncbi-blast-2.5.0+-x64-linux.tar.gz
tar -zxf ncbi-blast-2.5.0+-x64-linux.tar.gz
PATH=$PATH:/home/ubuntu/ncbi-blast-2.5.0+/bin
```

##### Instal TransRate

```
cd
curl -LO https://bintray.com/artifact/download/blahah/generic/transrate-1.0.3-linux-x86_64.tar.gz
tar -zxf transrate-1.0.3-linux-x86_64.tar.gz
PATH=$PATH:/home/ubuntu/transrate-1.0.3-linux-x86_64:/home/ubuntu/transrate-1.0.3-linux-x86_64/bin
```

##### Install BUSCO

```bash
cd
git clone https://gitlab.com/ezlab/busco.git
cd busco
PATH=$PATH:$(pwd)
wget http://cegg.unige.ch/pub/BUSCO2/metazoa_odb9.tar.gz && tar -zxf metazoa_odb9.tar.gz
```

##### Add things to your permanent path
```bash
echo PATH=$PATH >> ~/.profile
echo export LD_LIBRARY_PATH=/home/linuxbrew/.linuxbrew/Cellar/salmon/0.7.2/lib >> ~/.profile
echo source /home/ubuntu/.rvm/scripts/rvm >> ~/.profile
source ~/.profile
```

##### Download and unpack the data. This is 1 million reads. Remember how many ready you have in your project dataset.

```bash
mkdir ~/reads
cd ~/reads
curl -LO https://s3.amazonaws.com/feeding/1.subsamp_1.fastq.gz
curl -LO https://s3.amazonaws.com/feeding/1.subsamp_2.fastq.gz
```


Assembly. SPAdes is probably not the best assembler for transcriptomes, but it's FAST.. And we don't have a lot of time, so here it is! We're running 2 different assemblies, one using a kmer value of 55, one using a kmer value of 25. Which one gives you the better assembly? Predictions?

```bash
mkdir ~/spades
cd ~/spades

spades.py -t 16 -m 25 -k 55 --rna --only-assembler \
--pe1-1 ~/reads/1.subsamp_1.fastq.gz \
--pe1-2 ~/reads/1.subsamp_2.fastq.gz \
-o K55

spades.py -t 16 -m 25 -k 25 --rna --only-assembler \
--pe1-1 ~/reads/1.subsamp_1.fastq.gz \
--pe1-2 ~/reads/1.subsamp_2.fastq.gz \
-o K25

```

##### Run TransRate for BOTH the K25 and K55 assemblies.. Here is the command for K=55, you figure out what the K=25 command is.

```bash

mkdir ~/evaluation && cd ~/evaluation

transrate -o K55 -t 16 \
-a ~/spades/K55/transcripts.fasta \
--left ~/reads/1.subsamp_1.fastq.gz \
--right ~/reads/1.subsamp_2.fastq.gz
```



##### Run BUSCO for BOTH the K25 and K55 assemblies.. Here is the command for K=55, you figure out what the K=25 command is.
```bash
BUSCO.py -m tran --cpu 16 -l ~/busco/metazoa_odb9 \
-o K55 -i ~/spades/K55/transcripts.fasta
```

### Seriously, what do these numbers tell you about your transcriptome? Which assembly is better?



# Terminate your instance
