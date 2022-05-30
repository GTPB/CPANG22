---
layout: page
title: Day 3a - Saccharomyces cerevisiae pangenome graphs
schemadotorg:
  "@context": http://schema.org/
  "@type": CreativeWork
  "genre": TrainingMaterial
  isPartOf:
    url: "https://gtpb.github.io/CPANG22/"
    name: "CPANG22 - Computational PANGenomics"
---

### Learning objectives

In this exercise you learn how to

- estimate sequence divergence,
- identify variants in the graph.

### Getting started

Make sure you have `pggb` and its tools installed.

Now create a directory to work on for this tutorial:

    cd ~
	mkdir day3_yeast
	cd day3_yeast

Download 7 *Saccharomyces cerevisiae* assemblies ([Yue et al., 2016](https://doi.org/10.1038/ng.3847)) from the [Yeast Population Reference Panel (YPRP)](https://yjx1217.github.io/Yeast_PacBio_2016/welcome/) plus the `SGD` reference:

    mkdir assemblies
    cd assemblies

    wget -c http://hypervolu.me/~guarracino/CPANG22/yeast/DBVPG6044.fa.gz
    wget -c http://hypervolu.me/~guarracino/CPANG22/yeast/DBVPG6765.fa.gz
    wget -c http://hypervolu.me/~guarracino/CPANG22/yeast/S288C.fa.gz
    wget -c http://hypervolu.me/~guarracino/CPANG22/yeast/SGDref.fa.gz
    wget -c http://hypervolu.me/~guarracino/CPANG22/yeast/SK1.fa.gz
    wget -c http://hypervolu.me/~guarracino/CPANG22/yeast/UWOPS034614.fa.gz
    wget -c http://hypervolu.me/~guarracino/CPANG22/yeast/Y12.fa.gz
    wget -c http://hypervolu.me/~guarracino/CPANG22/yeast/YPS128.fa.gz


### Pangenome Sequence Naming
    
Rename the sequences according to the [PanSN-spec](https://github.com/pangenome/PanSN-spec) specification, using [fastix](https://github.com/ekg/fastix). The sequence names have to follow such a scheme:

    DBVPG6044#1#chrI
    DBVPG6044#1#chrII
    ...
    S288C#1#chrI
    S288C#1#chrII
    ...
    SGDref#1#chrI
    SGDref#1#chrII
    ...
    
Put all sequences in a single FASTA file called `scerevisiae8.fasta`. Compress such a file with `bgzip` and index it:

    bgzip -@ 8 scerevisiae8.fasta
    samtools faidx scerevisiae8.fasta.gz


### Sequence divergence

`pggb` exposes parameters that allow users to influence the structure of the graph that will represents the input sequences. In particular, reducing the mapping identity (`-p` parameter) increases the sensitivity of the alignment, leading to more compressed graphs. It is recommended to change this parameter depending on how divergent are the input sequences.

Assuming we will work with one chromosome at a time, we estimate the sequence divergence for each set of chromosomes. To partition the sequences by chromosome, execute:

    cut -f 1 scerevisiae8.fasta.gz.fai | cut -f 3 -d '#' | sort | uniq | while read CHROM; do
        CHR_FASTA=scerevisiae8.$CHROM.fasta.gz
        samtools faidx scerevisiae8.fasta.gz $(grep -P $"$CHROM\t" scerevisiae8.fasta.gz.fai | cut -f 1) | bgzip -@ 8 > $CHR_FASTA

        echo "Generated $CHR_FASTA"
    done
    
For each set of chromosomes, to estimate the [distance](https://mash.readthedocs.io/en/latest/distances.html#distance-estimation) of each input sequence to every other sequence in the set, we use [mash](https://doi.org/10.1186/s13059-016-0997-x). In particular, the `mash triangle` command outputs a lower-triangular distance matrix. For example, to compute the distances between all mitochondrial sequences, execute:

    mash triangle scerevisiae8.chrMT.fasta.gz -p 8 > scerevisiae8.chrMT.mash_triangle.txt
    cat scerevisiae8.chrMT.mash_triangle.txt | column -t
    
The distance is between 0 (identical sequences) and 1. Which is the maximum divergence? Find the maximum divergence for all chromosomes.

From this analysis, chrVII, chrVIII, and chrXIII sets should show the higher sequence divergence, with maximum value of ~ 0.0639874. In general, we should set a mapping identity value lower than or equal to `100 - max_divergence * 100`. That is, to analyze such a dataset, we have to specify `-p` lower than or equal to `93.60126`. However, in order to account for possible underestimates of sequence divergence, and medium/large structural variants leading locally to greater divergence, we can set an even smaller mapping identity, like `-p 90`.


### Pangenome graph building

Run `pggb` on each chromosome separately (including the mitochondrial one, chrMT), calling variants with respect to the `SGDref` assembly. For example, for chromosome 1, run:

    cd ~/day3_yeast
    mkdir -p graphs
    pggb -i assemblies/scerevisiae8.chrI.fasta.gz -s 20000 -p 90 -n 8 -t 8 -G 7919,8069 -o graphs/scerevisiae8.chrI -V SGDref:#

The `-V SGDref:#` parameter specify to call variants with respect to the path having as sample name `SGDref`; the sample name is identified by using `#` as separator (take a look at the [PanSN-spec](https://github.com/pangenome/PanSN-spec) specification).

Counts the number of variants in the VCF file for each chromosomes. Which chromosome has the least named variants? Are there chromosomes on which 0 variants are called? If so, why? Hint: take a look at the alignments in the PAF file produced by `pggb` (take a look at the [PAF format specification](https://github.com/lh3/miniasm/blob/master/PAF.md) too). Try to understand what is happing and how the problem can be solved.

Use `odgi squeeze` to put all the graphs together in the same file. Try to visualize the graph layout of the squeezed graph (the graph just generated with all chromosomes together) with `odgi layout` and `odgi draw`.

Run `pggb` on all chromosomes jointly, giving in input the `scerevisiae8.fasta.gz` file. Call variants too with respect to the `SGDref` sample. Take a look at the graph layout. Is the layout of the graph obtained by running `pggb` separately on each chromosome (and then combining its results) similar to the graph obtained by running `pggb` on all chromosomes jointly? 

In the newly obtained VCF, count how many variants are called for each chromosome. Are these counts similar to those obtained by calling variants on each chromosome separately? Are there variants esclusive (that do not appear in the chromosome-based VCF files) to this new VCF file?

### Back

Back to [main page](../index.md).
