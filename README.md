# Genordi

Match read alignments and annotated genes in an ordinal scale on genome.


## Mission

This program aims to combine the two fundamental analyses in metagenomics: **taxonomic profiling** (map reads to genomes) and **functional profiling** (map reads to functional genes) into one run. This saves compute, ensures consistency, and allows for **stratification** which is essential for understanding functional diversity across the microbial community.


## Installation

This program works out of the box. It needs Python 3 and no other dependencies. Just download and execute `genordi.py`.


## Examples

```bash
genordi.py -i S01.bt2.sam -g coords.txt -o test2.txt
xzcat S01.burst.b6.xz | genordi.py -g coords.txt -p > S01.txt
samtools view S01.bt2.bam | genordi.py -g coords.txt -p > S01.txt
```
