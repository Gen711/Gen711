## Lab 3: Alignment


During this lab, we will acquaint ourselves with alignment. Your objectives are:



1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset, tell me how some of these genes are related to their homologous copies.



Step 1: Launch and AMI. For this exercise, we will use a c4.xlarge instance.

::

	ssh -i ~/Downloads/your.pem ubuntu@???-???-???-???



The machine you are using is Linux Ubuntu: Ubuntu is an operating system you can use (I do) on your laptop or desktop. One of the nice things about this OS is the ability to update the software, easily.  The command ``sudo apt-get update`` checks a server for updates to existing software.


```bash
sudo apt-get update && sudo apt-get -y upgrade
```

OK, what are these commands?  ``sudo`` is the command that tells the computer that we have admin privileges. Try running the commands without the sudo -- it will complain that you don't have admin privileges or something like that. *Careful here, using sudo means that you can do something really bad to your own computer -- like delete everything*, so use with caution. It's not a big worry when using AWS, as this is a virtual machine- fixing your worst mistake is as easy as just terminating the instance and restarting.



So now that we have updates the software, lets see how to add new software. Same basic command, but instead of the ``update`` or ``upgrade`` command, we're using ``install``. EASY!!

```bash
sudo apt-get -y install build-essential git
```
## Install BioPython

```bash
pip install biopython
```

## Install LinuxBrew

#### Install Ruby

```bash
cd
wget https://keybase.io/mpapis/key.asc
gpg --import key.asc
\curl -sSL https://get.rvm.io | bash -s stable --ruby
source /home/ubuntu/.rvm/scripts/rvm
```

#### Install LinuxBrew itself

```bash
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

## Install mafft and RAxML and BLAST

```bash
brew install mafft raxml blast aria2
```

## Download the database and unzip it


```bash
aria2c ftp://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.fasta.gz
gzip -d uniprot_sprot.fasta.gz
```


## Download the query sequences

```bash
aria2c https://s3.amazonaws.com/gen711/dataset1.pep
```

## Make blast database and blast

```bash
makeblastdb -in uniprot_sprot.fasta -out uniprot -dbtype prot
```


**For the query sequences**, I am going to pull out 2 specific sequences from the larger dataset ``dataset1.pep``. These are both HOX genes that I have identified ahead of time. The command ``grep`` finds words/numbers/symbols in files. Remember that the ``>`` sends the results of the command to a file, in this case names ``query.pep``.

```bash
grep -A1 'ENSPTRP00000032491\|ENSPTRP00000032494' dataset1.pep > query.pep
```


Now that I have a couple of query sequences, we are ready to blast.

```bash
  blastp -evalue 8e-8 -num_threads 4 -db uniprot -query query.pep -max_target_seqs 3 -outfmt "6 qseqid score pident evalue stitle"
```


You will see the results in a table with 4 columns. Use `blastp -help` to see what the results mean. Test out some of the blast options. Try changing the matrix size using ``-matrix``, to see how these changes affect the results.


#### Make a file that contains the sequences for the blast hits you just discovered


```bash
### You need a little script to help you pull out the sequences

curl -LO https://raw.githubusercontent.com/macmanes-lab/general/master/filter.py

### Now open a file, and make a list of the names of the proteins you just identified using blast

nano cool_prots.list

### paste this list, and save and quit nano

HXA2_HUMAN
HXA2_BOVIN
HXA2_PAPAN
HXA3_HUMAN
HXA3_MOUSE
HXA3_BOVIN
HXA9_HUMAN


### Now sue the sctipr to pull out the proteins into a file

python filter.py uniprot_sprot.fasta \
<(grep -f cool_prots.list uniprot_sprot.fasta | awk '{print $1}' | sed 's_>__') > cool_prots.fasta

```

#### Now, make a file that contains BOTH  the query sequences AND the sequences we found by blasting.

```bash

cat query.pep cool_prots.fasta > for_alignment.pep

```


#### Use mafft for alignment


```bash

mafft --reorder --bl 80 --localpair --thread 4 for_alignment.pep > for.tree

```

#### RAxML

Make a phylogeny

```bash

raxmlHPC-PTHREADS -f a -m PROTCATBLOSUM62 -T 4 -x 34 -N 100 -n tree -s for.tree -p 35


```

### Copy phylogeny and view online.

```bash

	less RAxML_bipartitionsBranchLabels.tree

	#copy this info.

```

Visualize tree on website: http://www.evolgenius.info/evolview/

0. Click on "Use without and account"
1. Click on the folder icon in the top-left part of the page.
2. Paste in the code from your terminal. **FYI, this is the NEWICK tree format**, yes, named after Newick's Restaurant just down the road from us!!
3. Find the HOX9 gene - this is the outgroup sequence we will use to rood the tree. Hover over the branch and it will turn red - a box will open, click "reroot here"


# TERMINATE YOUR INSTANCE
