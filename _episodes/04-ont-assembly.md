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

Recent new single molecule technologies like PacBio and Oxford Nanopore are promising. Here we will assemble data from NCBI Bioproject [https://www.ncbi.nlm.nih.gov/bioproject/PRJDB9013](https://www.ncbi.nlm.nih.gov/bioproject/PRJDB9013), which was used for `Benchmarks of de novo assemblers for bacterial genomes`. The data we will use is *E. coli* str. K12 substr. MG1655 with identifier `DRR198814` (https://www.ncbi.nlm.nih.gov/sra/DRX189231[accn]) which was sequenced on a MinION from Oxford Nanopore. [https://nanoporetech.com/products/minion](https://nanoporetech.com/products/minion)

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



