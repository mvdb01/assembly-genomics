---
title: "de novo Short Read Paired-End and Mate-Paired Assembly"
teaching: 10
exercises: 0
questions:
- "How to do a de novo short read paired-end genome assembly using mate-pairs?"
objectives:
- ""
keypoints:
- ""
---

# *De Novo* assembly (Paired End and Mate-pair libaries)

In most assembly projects multiple libraries with different insert sizes are used.

Here we will add an 2.5 kb Mate pair library.

Due to a different library preparation the read orientation of these libraries are different. PE: → ← MP: ← →


# SPAdes PE and MP assembly


> ## Exercise
> 
> Assemble the trimmed Paired End library and a Mate-pair library, located at: `~/asm_workshop/data/mp/MP_2.5kb_25x_?.fastq.gz` with SPAdes. Use as output dir `ecoli_pe_mp`.
>
> We have to specify in SPADES the paired-end (PE) and the mate-pair (MP) library by applying the --pex-x and --mpx-x flags. And also provide the read orientation for the Mate-pair library.
>
>
> hint: run `spades.py -h` for the command options.
>
>
>> ## Solution
>> 
>> ~~~
>> $ spades.py -h
>> ~~~
>> {: .bash}
>>
>> with --pe1-1 for read1 and --pe1-2 for read2 we specify the paired-end library.
>> with --mp1-1 for read1 and --mp1-2 for read2 we specify the mate-pair library.
>>
>> With --mp1-rf we inform SPAdes the mate-pair read orientation `<- (reverse), -> (forward)`
>>
>> The full command for the two libraries:
>>
>> ~~~
>> $ spades.py \
>>      --pe1-1 ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_1.trim.fastq.gz \
>>      --pe1-1 ~/asm_workshop/data/trimmed_fastq/PE_600bp_50x_2.trim.fastq.gz \
>>      --mp1-1 ~/asm_workshop/data/mp/MP_2.5kb_25x_1.fastq.gz \
>>      --mp1-2 ~/asm_workshop/data/mp/MP_2.5kb_25x_2.fastq.gz \
>>      --mp1-rf \
>>      -o ~/asm_workshop/results/ecoli_pe_mp
>> ~~~
>> {: .bash}
>>
> {: .solution}
{: .challenge}

# QUAST on PE and MP assemblies



