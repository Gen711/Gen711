### Final Project

By now, everybody should have picked a dataset to assembly for their project. For those of you assembling a transcriptome, you should follow the [Oyster River Protocol versio 2](http://oyster-river-protocol.readthedocs.io/en/v2/index.html). Specifically, you should do steps 2-7, as well as a final BUSCO and TransRate evaluation on the final assembly `Highexp.fasta`.

##### Hardware

I have made a Amazon machine with all the software already installed, so you will only have to download the reads to that computer, and start working. I think this should make things a lot easier for you! To access the machine:

1. Go to the AWS console like you usually do.
2. On the left side bar, you will see AMIs, click this. You'll be launching an AMI that I've made for you.
3. Once on the AMI page, type `UNH_Gen711` and change `Owned by me` to `Public Images`, then hit enter. The image should appear.
4. At this point, launching the machine is basically how you have done it all semester.
5. If you are assembling less than 30M reads, you will probably want a machine like the c4.4xlarge, which has 16 cores. If you are assembling more that 30M reads, try the c4.8xlarge with 36 cores. Make sure to change the code in all places to use all the cores.     

##### Workflow

0. If people want to work on a computer in my lab, that is fine.
1. You have gotten used to terminating your instances when not in use - while you are working on this project, you will stop your machine, rather than terminate. Stopping is really like pausing. YOu can relaunch that machine at a later date, ad pick up where you left off.. IMPORTANT, only stop your instance between steps.
    - For instance, you do error correction, but you can't start the next step until the next day. Stop your machine, then restart it later.
2. Most people will probably want to start things, and let them run, then check a few hours later or the next morning. This is fine. Come talk with me if you are confused.

##### Oral Reporting: For all assemblies

0. Power point slide
1. Report the BUSCO stats (Complete, Duplicated, Fragmented, Missing)
2. Report the TransRate Numbers (n seqs, p good mapping, p bases uncovered, p contigs lowcovered, TRANSRATE ASSEMBLY SCORE, TRANSRATE OPTIMAL SCORE, p good contigs)

##### Written Reporting

1. Describe, what you did, and your briefly results. See [this paper](http://biorxiv.org/content/biorxiv/early/2016/11/11/085365.full.pdf) for an example. This is a couple of pages at most.  
