## Lab 3: Alignment


During this lab, we will acquaint ourselves with alignment. Your objectives are:



1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset, tell me how some of these genes are related to their homologous copies.



Step 1: Launch and AMI. For this exercise, we will use a c4.xlarge instance. This instance has 4 cores.

```bash

ssh -i ~/Downloads/your.pem ubuntu@???-???-???-???

```

Update the software

```bash
sudo apt-get update && sudo apt-get -y upgrade
```

OK, what are these commands?  ``sudo`` is the command that tells the computer that we have admin privileges. Try running the commands without the sudo -- it will complain that you don't have admin privileges or something like that. *Careful here, using sudo means that you can do something really bad to your own computer -- like delete everything*, so use with caution. It's not a big worry when using AWS, as this is a virtual machine- fixing your worst mistake is as easy as just terminating the instance and restarting.



So now that we have updates the software, lets see how to add new software. Same basic command, but instead of the ``update`` or ``upgrade`` command, we're using ``install``. EASY!!

```bash
sudo apt-get -y install build-essential git
```

## Install LinuxBrew.
http://linuxbrew.sh/. LinuxBrew can be installed in your home directory and does not require root access. The same package manager can be used on both your Linux server and your Mac laptop.

#### Install Ruby
LinuxBrew needs Ruby to install

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
Yes, installing software is easy.. `brew install XXX`. Do you have a favorite software package. Try and install it, in addition to the other things we are installing.

```bash
brew install mafft raxml blast aria2
```


## Install BioPython
BioPython is a python package that does a lot of common tasks related to sequence manipulation.

```bash
pip install biopython
```


## Download the blast database and unzip it
We are downloading SwissProt, which has about 550,000 curated protein sequences.

Note we are using a program called `aria2c` to download. This is a something that, for huuuuge files, can significantly speed up the process. For smaller things, I'll still use `curl`

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


**For the query sequences**, I am going to pull out 2 specific sequences from the larger dataset ``dataset1.pep``. These are both HOX genes that I have identified before class. The command ``grep`` finds words/numbers/symbols in files. The `-A1` flag that I pass to the `grep` command asks that grep returns the line that matches, but also the line just below the match. We're downloading a fasta record.. think about why this makes sense.

Remember that the ``>`` sends the results of the command to a file, in this case names ``query.pep``.

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


### Now sue the script to pull out the proteins into a file. Note that this script does something pretty fancy. It uses process substitution (https://en.wikipedia.org/wiki/Process_substitution). To understand process substitution you also need to get familiar with the ideas of stdout and stdin (https://en.wikipedia.org/wiki/Standard_streams)

python filter.py uniprot_sprot.fasta \
<(grep -f cool_prots.list uniprot_sprot.fasta | awk '{print $1}' | sed 's_>__') > cool_prots.fasta

```

#### Now, make a file that contains BOTH  the query sequences AND the sequences we found by blasting.
What does `cat` do??


```bash

cat query.pep cool_prots.fasta > for_alignment.pep

```


#### Use mafft for alignment
What are the different options to `mafft`

```bash

mafft --reorder --bl 80 --localpair --thread 4 for_alignment.pep > for.tree

```

#### RAxML
What are the different options to `RaxML`


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

0. Click on "Use without an account"
1. Click on the folder icon in the top-left part of the page.
2. Paste in the code from your terminal. **FYI, this is the NEWICK tree format**, yes, named after Newick's Restaurant just down the road from us!!
3. Find the HOX9 gene - this is the outgroup sequence we will use to rood the tree. Hover over the branch and it will turn red - a box will open, click "reroot here"


# TERMINATE YOUR INSTANCE