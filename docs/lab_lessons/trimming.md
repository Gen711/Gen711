## Lab4: Trimming fastQ

Step 1: Launch the AMI. For this exercise, we will use a c4.2xlarge instance. **IMPORTANT DETAIL!!!!!!!!!** We need to have more hard drive space for this exercise. So when you select the machine type, don't click ``Review and Launch`` like you normally do. Instead, go to near the top of teh page and click ``4. Add Storage``. You'll see a column labelled ``Size (GiB)`` with a 8 under that.. Change that 8 to 100. Now click ``Review and Launch`` like normal. You now have a computer with a hard drive of size 100Gb.


```bash
ssh -i ~/Downloads/?????.pem ubuntu@ec2-???-???-???-???
```

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


Download data, and uncompress them.. Let's put this in a tmux window so we can get to doing other things.. remember you need paste the tmux relevant commands one at a time.


```bash

tmux new -s download

cd $HOME && mkdir reads && cd reads
curl -LO https://s3.amazonaws.com/gen711/TruSeq3-PE.fa
curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R1.PF.fastq.gz > kidney.1.fq.gz &
curl -L https://s3.amazonaws.com/Mc_Transcriptome/Thomas_McBr1_R2.PF.fastq.gz > kidney.2.fq.gz

#then this to get out of tmux

ctl-b d

```

Install Skewer, a trimming tool. Seqtk, a sequence manipulation tool, and Python..


```bash  

brew install skewer seqtk python jellyfish

```

now go back to the download tmux window to see if//wait for the download

```bash
tmux at -t download

#IMPORTANT - when it is done downloading, kill the window

ctl-d

```

Run khmer. Make sure to look at the manual.


```bash

tmux new -s kmer
mkdir $HOME/kmer_analysis && cd $HOME/kmer_analysis

#IMPORTANT DETAIL: usually pasting things in 1 at a time is fine - except here... When you see ``\`` at the end of lines, this means copy the 2 (or 3 or 4) lines together.


#do trimming at P2

trim=2
seqtk mergepe ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz \
| skewer --stdout -l 25 -m pe --mean-quality $trim --end-quality $trim -t 8 -x $HOME/reads/TruSeq3-PE.fa - \
| jellyfish count -s 1000000 -m 25 -t 8 -o $trim.counts.jf -C /dev/stdin

jellyfish histo $trim.counts.jf > $trim.counts.histo

#do trimming at P30

trim=30
seqtk mergepe ~/reads/kidney.1.fq.gz ~/reads/kidney.2.fq.gz \
| skewer --stdout -l 25 -m pe --mean-quality $trim --end-quality $trim -t 8 -x $HOME/reads/TruSeq3-PE.fa - \
| jellyfish count -s 1000000 -m 25 -t 8 -o $trim.counts.jf -C /dev/stdin

jellyfish histo $trim.counts.jf > $trim.counts.histo


 # to exit out of the tmux window, if you want to.

 ctl-b d

```


Wait for these things to be done.. Use ``top -c`` to do this.. Remember ``q`` gets you outta ``top``.

Open up a new terminal (tab) or window using the buttons command-t. You're going to download the files you created on teh AWS machine to the MAC your using in the lab.

```bash
scp -i ~/Downloads/????.pem ubuntu@ec2-??-???-???-??:~/khmer_analysis/*histo ~/Downloads/


```


Now look at the and ``.histo`` file.  which is the plot of quality containing both the mean quality as well as that for each tile. I want you to plot the distribution using R and RStudio.



OPEN RStudio - this should be installed on your Mac. These commands you'll type into RStudio, NOT the terminal.


```bash

#Import Data
p2 <- read.table("~/Downloads/2.counts.histo", quote="\"", comment.char="")
p30 <- read.table("~/Downloads/30.counts.histo", quote="\"", comment.char="")

par(mfcol=c(2,1))

plot(p2$V2[2:10] ~ 1, type='p', lwd=5,
    col='blue', frame.plot=F, xlab='25-mer frequency', ylab='kmer count',
    main='Kmer distribution in sample with different trimming thresholds')

lines(p30$V2[2:10] ~ 1, type='p', lwd=5,
    col='red')

plot(p2$V2[2:30] - p30$V2[2:30], type='p',
    xlim=c(2,20), xaxs="i", yaxs="i", frame.plot=F,
    ylim=c(0,1500000), col='red', xlab='kmer frequency',
    lwd=4, ylab='count',
    main='Diff in 25mer counts of freq 2 to 20 \n Phred2 vs. Phred30')
```

##Terminate your instance!!!
