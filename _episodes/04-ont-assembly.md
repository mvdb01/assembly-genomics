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

Miniasm is a very fast OLC-based de novo assembler for noisy long reads like PacBio and Oxford Nanopore data. [https://github.com/lh3/miniasm](https://github.com/lh3/miniasm)

Miniasm needs an all-vs-all read self-mapping (typically by minimap2) as input.

The first step is to run minimap for the all-vs-all mapping.

Run minimap2 without options for command usage.

~~~
$  minimap2
~~~
{: .bash}

For all-vs-all mapping we use the raw ont reads as target `and` as query. As preset setting we have to use `-x ava-ont` for ONT read overlap and redirect the output to a `PAF` file (Pairwise mApping Format), which is a text format describing the approximate mapping positions between two sets of sequences.

~~~
$  minimap2 -x ava-ont \
            ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            > ~/asm_workshop/results/asm_ont/ont_overlaps.paf
~~~
{: .bash}

The all-vs-all alignments (`ont_overlaps.paf`) will be used as input for `miniasm` that will build a assembly graph in GFA format.

Besides the all-vs-all `PAF` file we also have to provide the raw ONT reads.

~~~
$ miniasm -f ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            ~/asm_workshop/results/asm_ont/ont_overlaps.paf \
            > ~/asm_workshop/results/asm_ont/ont_assembly.gfa
~~~
{: .bash}

# Error correcting (polishing) long read based assemblies

Contigs genereated from erroneous long read based *De Novo* assemblies needs to be error corrected to improve the consensus sequence. Here we will use Minipolish, which will use Racon for polishing, to error correct in three steps the miniasm assembly. 

## Step1: initial Racon polish with constituent reads

Miniasm's assembled contigs are made up of pieces of long reads and therefore has a high error rate – probably around 90% or so, depending on the input reads.

The miniasm GFA file indicates specifically which reads contributed to each contig.

Therefore, the first thing Minipolish does is to run Racon on each contig independently, only using the reads which were used to create that contig. This step typically quite fast because it does not involve high read depths, and it can bring the percent identity up to the high 90s.

## Step 2: full Racon polish rounds

Now that the assembly is in better shape, Minipolish does full Racon-polishing rounds – aligning the full read set to the whole assembly and getting a Racon consensus.

## Step 3: contig read depth

Minipolish finishes by doing one more read-to-assembly alignment, this time not to polish but to calculate read depths. These depths are added to the GFA line for each contig (e.g. dp:f:77.179) and they will be recognised if the graph is loaded in Bandage.

~~~
$ minipolish ~/asm_workshop/data/ont/DRR198814_44x.fastq.gz \
            ~/asm_workshop/results/asm_ont/ont_assembly.gfa \
            > ~/asm_workshop/results/asm_ont/ont_polished.gfa
~~~
{: .bash}


