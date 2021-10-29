---
title: "Genome Annotation"
teaching: 10
exercises: 0
questions:
- "How to find the genes present in a genome assembly"
objectives:
- ""
keypoints:
- ""
---

# Genome annotation

~~~
$ prokka --kingdom Bacteria --outdir ~/asm_workshop/results/prokka \
	      --genus Escherichia --prefix Ecoli_K12 \
	      --addgenes --species coli --strain K12 --usegenus \
	      ~/asm_workshop/results/ecoli_ont/ont_pilon_polished.fasta
~~~
{: .bash}
