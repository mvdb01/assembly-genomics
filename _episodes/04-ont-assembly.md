---
title: "de novo Long Read Assembly"
teaching: 10
exercises: 0
questions:
- "How to do a de novo long-read genome assembly?"
objectives:
- ""
keypoints:
- ""
---

# *De Novo* assembly (Nanopore)

Recent new single molecule technologies like PacBio and Oxford Nanopore are promising. Here we will assemble data from Nick Loman lab: [http://lab.loman.net/2016/07/30/nanopore-r9-data-release/](http://lab.loman.net/2016/07/30/nanopore-r9-data-release/), sequenced on a MinION SpotON flowcell R9.2. [https://nanoporetech.com/products/minion](https://nanoporetech.com/products/minion)

# Inspecting read length

First inspect the data by plotting the read length distribution. We will use a python script, `getHistogram_fastX.py` to make the plot.

Let's be sure we are in the asm_workshop directory. 

~~~
$ cd ~/asm_workshop
~~~
{: .bash}

And create an output folder:

~~~
$ mkdir ~/asm_workshop/results/ont_read_lengths
~~~
{: .bash}

Use the `-h` flag to see how to run the script

~~~
$  getHistogram_fastX.py -h
~~~
{: .bash}

Now apply this on the Nanopore reads: `~/asm_workshop/data/ont/E_coli_K12_1D_R9.2_SpotON_2.pass_50k.fasta`

~~~
$  getHistogram_fastX.py -i ~/asm_workshop/data/ont/E_coli_K12_1D_R9.2_SpotON_2.pass_50k.fasta \
                        -o ~/asm_workshop/results/ont_read_lengths/ont_reads \
                        -f fasta -s ont_reads -x 20000
~~~
{: .bash}

getHistogram_fastX.py create a png file which we have to download.

~~~
$ mkdir ~/Desktop/ont
$ scp YOUR-NETID@student-linux.tudelft.nl:~/asm_workshop/results/ont_read_length/ont_reads.png ~/Desktop/ont/
~~~
{: .bash}

Compare the read length distribution to the illumina reads.

# Miniasm assembler



