---
layout: page
title: Day 4b - Read mapping
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

- align reads against the graph,
- call variants.


### Getting started

Go to the `day1_intro` folder:

    cd ~/day1_intro


### Read mapping

Let's step up to a slightly bigger example.

	ls ~/vg/test/1mb1kgp

This directory contains 1Mbp of 1000 Genomes data for `chr20:1000000-2000000`. As for the tiny example, let's' build a graph that encodes the known sequence variation.

    vg construct -r 1mb1kgp/z.fa -v 1mb1kgp/z.vcf.gz -m 1024 > z.vg

Let's build the indexes needed for the mapping:

    vg index -L -x z.xg z.vg
    vg index -g z.gcsa -k 16 z.vg

Get the reads to map:

    wget -c http://hypervolu.me/~guarracino/CPANG22/NA12878.20_1M-2M.30x.bam

    samtools fastq -1 NA24385-20_1M-2M-30x_1.fq.gz -2 NA24385-20_1M-2M-30x_2.fq.gz NA12878.20_1M-2M.30x.bam 

The sample was used in the preparation of the 1000 Genomes results.

Map reads against the built graph:

    vg map -d z -f NA24385-20_1M-2M-30x_1.fq.gz -f NA24385-20_1M-2M-30x_2.fq.gz > NA24385-20_1M-2M-30x.gam


### Variant calling

Compute the read support:

    vg pack -x z.xg -g NA24385-20_1M-2M-30x.gam -o NA24385-20_1M-2M-30x.pack -Q 5

Compute the snarls (using fewer threads with `-t` can reduce memory at the cost of increased runtime):
    
    vg snarls z.xg > z.snarls


Call variants:

    vg call z.xg -r z.snarls -k NA24385-20_1M-2M-30x.pack > NA24385-20_1M-2M-30x.vcf 

### Back

Back to [main page](../index.md).
