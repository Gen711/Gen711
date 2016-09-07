## Lab 1: BLAST


During this lab, we will acquaint ourselves with the the software package BLAST. Your objectives are:



1. Familiarize yourself with the software, how to execute it, how to visualize results.

2. Regarding your dataset, tell me how some of these genes are related to their homologous copies.


Step 1: Launch and AMI. You will need to navigate to aws.amazon.com to do this. Follow the instructions on the `Amazon Resources` page if you cannot remember how to launch an instance.  For this exercise, we will use a c4.xlarge instance, which has 4 cores and costs 21 cents per hour to rent.

    # This is a comment, anything that follow the hashtag symbol is a comment.

	# replace the `x's` in the below command with your Public IP address

	ssh -i ~/Downloads/your.pem ubuntu@xx.xx.xx.x



The machine you are using is Linux Ubuntu: Ubuntu is an operating system you can use (I do) on your laptop or desktop. One of the nice things about this OS is the ability to update the software, easily.  The command `sudo apt-get update` checks a server for updates to existing software. `apt-get` is like the Mac app store for those of you that are Mac/iPhone users.

	sudo apt-get update

The upgrade command actually installs any of the required updates.

	sudo apt-get -y upgrade

OK, what are these commands?  `sudo` is the command that tells the computer that we have admin privileges. Try running the commands without the sudo -- it will complain that you don't have admin privileges or something like that. *Careful here, using sudo means that you can do something really bad to your own computer -- like delete everything*, so use with caution. It's not a big worry when using AWS, as this is a virtual machine- fixing your worst mistake is as easy as just terminating the instance and restarting.


So now that we have updates the software, lets see how to add new software. Same basic command, but instead of the `update` or `upgrade` command, we're using `install`. EASY!!


	sudo apt-get -y install build-essential git curl gcc make g++ python-dev unzip


ok, for this lab we are going to use BLAST, which is available as a package entitled `ncbi-blast+`

	# replace the ??? with the appropriate command that will install blast.

	sudo apt-get -y install ???

to get a feel for the different options, type `blastp -help`. Which type of blast does this correspond to? Look at the help info for blastp and tblastx. There are A LOT of options, most of which you will be able to safely ignore. One of the challenges of bioinformatics is knowing about the options and which ones you can safely ignore, and which ones are important.


### Challenge...

You have just returned from South America, where you captured a rare - never been seen before - desert mouse. You are interested in knowing how it survives, and start by trying to identify the Sodium transport genes, in particular a gene called Scn5a (https://en.wikipedia.org/wiki/Nav1.5). You've just had the animals's transcriptome sequenced, and are about to begin the search!! You *could* use web blast like you've done in the past, but there are 20166 sequences that you need to search and that would take a loooooooong time.

Download the data here:

 	curl -LO https://s3.amazonaws.com/macmanes_share/transcripts.fasta

You know that the model organism, the lab mouse, has an excellent genome and you decide to use it to help identify the sodium transport genes in the new animal. Download the data:

	curl -LO ftp://ftp.ensembl.org/pub/release-85/fasta/mus_musculus/cdna/Mus_musculus.GRCm38.cdna.all.fa.gz
	gzip -d Mus_musculus.GRCm38.cdna.all.fa.gz

#### OK, let's do some blasting..

1st step is to make the blast database

	makeblastdb -in Mus_musculus.GRCm38.cdna.all.fa -out mus -dbtype nucl

Now blast.. your results should be saved in a file called `blast.out`

	blastn -db mus -max_target_seqs 1 -query transcripts.fasta \
	-outfmt '6 qseqid evalue stitle' -evalue 1e-10 -num_threads 4 -out blast.out

look at `blast.out`. the 1st column is the ID of the desert mouse transcript, the 2nd column is the e-value (a statistic related to how good the match between query and reference sequences is - smaller numbers better. The 3rd column is the Mouse transcript match descriptor)

	less -S blast.out

Oh crap.. there are 16501 lines in that file, how are we going to find the Scn5a gene that we are looking for?? Meet `grep`.

	grep -i SCN5A blast.out

As an additional exercise, see if you sequenced your favorite gene in the new animals. Navigate to http://useast.ensembl.org/Mus_musculus/Info/Index to find the appropriate gene codes. For example..

    grep -i "solute carrier" blast.out
    grep -i aquaporin blast.out
    grep -i Rbp2 blast.out
