Week 7: DigiNorm
--


During this lab, we will acquaint ourselves with digital normalization. You will:

1. Install software and download data

2. Quality and adapter trim data sets.

3. Apply digital normalization to the dataset.

4. Count and compare kmers and kmer distributions in the normalized and un-normalized dataset.

5. Plot in RStudio.

-

The JellyFish manual: <a href="ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf">ftp://ftp.genome.umd.edu/pub/jellyfish/JellyfishUserGuide.pdf</a>

The Khmer manual: <a href="http://khmer.readthedocs.org/en/v1.1/">http://khmer.readthedocs.org/en/v1.1/</a>

---

Step 1: Launch and AMI. For this exercise, we will use a c3.xlarge instance. Remember to change the permission of your key code `chmod 400 ~/Downloads/????.pem` (change ????.pem to whatever you named it)

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
brew tap Gen711/homebrew-science
brew update
brew doctor


```


Download data, and uncompress them.


```bash
cd $HOME && mkdir reads && cd reads
curl -LO https://s3.amazonaws.com/gen711/TruSeq3-PE.fa
curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R1.PF.fastq.gz > kidney.1.fq.gz &
curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R2.PF.fastq.gz > kidney.2.fq.gz

```

Install Skewer, a trimming tool. Seqtk, a sequence manipulation tool, and Python..


```bash  

brew install skewer seqtk python jellyfish

```

Install khmer

```bash
pip install khmer
```


Trim low quality bases and adapters from dataset. These files will form the basis of all out subsequent analyses.


```bash
trim=2
norm=30

#paste the below lines together as 1 command

seqtk mergepe ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz \
	| skewer --stdout -l 25 -m pe --mean-quality $trim --end-quality $trim -t 8 -x $HOME/reads/TruSeq3-PE.fa - \
	| jellyfish count -s 1000000 -m 25 -t 8 -o /dev/stdout -C /dev/stdin \
 	| jellyfish histo /dev/stdin -o trimmed.yes.normalize.histo

#and

#paste the below lines together as 1 command

seqtk mergepe $HOME/reads/kidney.1.fq.gz $HOME/reads/kidney.2.fq.gz \
  | skewer --stdout -l 25 -m pe --mean-quality $trim --end-quality $trim -t 8 -x $HOME/reads/TruSeq3-PE.fa - \
  | normalize-by-median.py --max-memory-usage 4e9 -C 30 -o - - \
  | jellyfish count -s 1000000 -m 25 -t 8 -o /dev/stdout -C /dev/stdin \
  | jellyfish histo /dev/stdin -o trimmed.no.normalize.histo

```

> Open up a new terminal window using the buttons command-t


	scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??.compute-1.amazonaws.com:/mnt/jelly/*histo ~/Downloads/


> Look at the `.histo` file, which is a kmer distribution. I want you to plot the distribution using R and RStudio.

-

> OPEN RSTUDIO


    #Import all 2 histogram datasets: this is the code for importing 1 of them..

    khmer <- read.table("~/Downloads/trimmed.yes.normalize.histo", quote="\"")
    trim <- read.table("~/Downloads/trimmed.no.normalize.histo", quote="\"")

    #What does this plot show you??

    barplot(c(trim$V2[1],khmer$V2[1]),
        names=c('Non-normalized', 'C50 Normalized'),
        main='Number of unique kmers')

    # plot differences between non-unique kmers

    plot(khmer$V2[10:300] - trim$V2[10:300], type='l',
        xlim=c(10,300), xaxs="i", yaxs="i", frame.plot=F,
        ylim=c(-10000,60000), col='red', xlab='kmer frequency',
        lwd=4, ylab='count',
        main='Diff in 25mer counts of \n normalized vs. un-normalized datasets')
    abline(h=0)



---

<a href="http://genomebio.org/Gen711/wp-content/uploads/2014/10/khmer_norm1.jpeg"><img class="aligncenter size-large wp-image-240" src="http://genomebio.org/Gen711/wp-content/uploads/2014/10/khmer_norm1-1024x700.jpeg" alt="khmer_norm" width="1024" height="700" /></a>

-

-

> What do the analyses of kmer counts tell you?
