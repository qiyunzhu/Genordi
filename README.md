# Genordi

Match read alignments and annotated genes in an ordinal scale on genome.


## Mission

This program aims to combine the two fundamental analyses in metagenomics: **taxonomic profiling** (map reads to genomes) and **functional profiling** (map reads to functional genes) into one run. This saves compute, ensures consistency, and allows for **stratification** which is essential for understanding functional diversity across the microbial community.


## Installation

Genordi works out of the box. It needs Python 3 and no other dependencies. Just download and execute `genordi.py`.


## Quick Start

Construct a profile (gene-to-counts) based on a Bowtie2 alignment file and a reference gene table:

```bash
genordi.py -i sample.bowtie2.sam -g gene_table.txt -o sample.profile.txt
```


## Usage

### File operations

Genordi supports file compression and piping for efficient I/O operations. Examples:

```bash
samtools view align.bam | genordi.py -g gtab.txt > profile.txt
```

```bash
genordi.py -i <(xzcat burst.txt) -g <(zcat gtab.txt.gz) | bzip2 > profile.txt.bz2
```

### Input read alignment

Genordi supports and automatically recognizes two alignment formats: **SAM** (e.g., Bowtie2, BWA) and **BLAST tabular format** (a.k.a., "b6" or "m8") (e.g., BLAST, DIAMOND, BURST):

```bash
genordi.py -i bowtie2.sam -g gtab.txt -o profile.txt
genordi.py -i   burst.b6o -g gtab.txt -o profile.txt
```

One may use parameter `-f` to mandatorily designate a format.


### Reference gene table

A table of gene IDs and their start / end coordinates on corresponding genome sequences. Examples:

Native NCBI annotations and accessions:
```
## GCF_000005825.2
# NC_013791.2
WP_012957018.1  816 2168
WP_012957019.1  2348    3490
WP_012957020.1  3744    3959
WP_012957021.1  3971    5086
...
```

Note that one genome may contain several (e.g., chromosomes in a complete genome) to many nucleotide sequences (e.g., contigs in a draft genome). The hierarchy is indicated using genome prefix (`##` or `>>`) and nucleotide prefix (`#` or `>`).

"Web of Life"-style reannotations:
```
>>G000006745
>NC_002505.1
1       806     372
2       2177    816
3       3896    2271
4       4446    4123
5       4629    4492
...
```

Each Prodigal-inferred ORF is assigned an incremental index on the corresponding genome sequence. This reduces the size of the gene table. One may use flag `-p` to append nucleotide IDs to gene IDs in the output profile:

```bash
genordi.py -p -i input.b6o -g gtab.txt -o profile.txt
```

### Output profile

The default output profile is a gene-to-count table:

```
NC_000963.1_101 5
NC_000963.1_102 1
NC_000963.1_104 4
NC_000963.1_105 1
NC_000963.1_107 1
...
```

One may use flag `-m` to instead generate a read-to-gene(s) mapping file:

```bash
genordi.py -m -i input.b6o -g gtab.txt -o mapping.txt
```

```
S0R8/1  NC_006348.1_948
S0R8/2  NC_006348.1_948
S0R9/1  NC_006461.1_1225
S0R9/2  NC_006461.1_1225,NC_006461.1_612
S0R10/1 NC_006461.1_612
```

The latter is useful for further manipulation (e.g., taxonomy-based stratification). One can then convert it into a profile:

```bash
genordi.py -i mapping.txt -o profile.txt
```

## Match threshold

Genordi determines that a read matches a gene if the length of the alignment region (i.e., the gene/read overlapping region) is no less than a fraction of the read length. The default threshold is **80%**. One may control this value using parameter `-t`:

```bash
genordi.py -t 0.95 -i input.b6o -g gtab.txt -o profile.txt
```

## Ambiguous alignment

Alignment is not always unique. Depending on the sequence alignment algorithm and parameter settings, one read may be aligned to one or multiple regions on the same/different nucleotide/genome sequences, hence the input alignment file may contain one or multiple instances of the same read IDs.

Genordi provides parameter `-a` to control the behavior in such scenario:

- `uniq`: Drop non-unique assignments.
- `all`: Count each occurrence once.
- `norm` (default): Count each occurrence 1/_k_ times (_k_ = number of occurrences)

Note: In rare instances (e.g., the same genomic region is shared by coding sequences of two genes), one read may be assigned to more than one gene (not genome). In this scenario, all genes will be retained and counted once.