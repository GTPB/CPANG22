---
layout: page
title: Day 4a - Saccharomyces cerevisiae pangenome graphs
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

- partition sequences.


### Getting started

Make sure you have `pggb` and its tools installed. In particular, check out `pggb` repository:

    cd ~
    git clone https://github.com/pangenome/pggb.git


Next, go in the directory created in the previous activity on *Saccharomyces cerevisiae*:

    cd ~/day3_yeast


### Sequence partitioning

We can't really expect to pairwise map all sequences together and obtain well separated connected components. It is likely to get a giant connected component, and probably a few smaller ones, due to incorrect mappings or false homologies. This might unnecessarily increase the computational burden, as well as complicate the downstream analyzes. Therefore, it is recommended to split up the input sequences into communities in order to find the latent structure of their mutual relationship.

We need to obtain the mutual relationship between the input assemblies in order to detect the underlying communities. To compute the pairwise mappings with `wfmash`, execute:

    cd assemblies
    wfmash scerevisiae8.fasta.gz -p 90 -n 7 -t 8 -m > scerevisiae8.mapping.paf

Why did we specify `-n 7`?

To project the PAF mappings into a network format (an edge list), execute:

    python3 ~/pggb/scripts/paf2net.py -p scerevisiae8.mapping.paf
    
The `paf2net.py` script creates 3 files:

- `scerevisiae8.mapping.paf.edges.list.txt` is the edge list representing the pairs of sequences mapped in the PAF;
- `scerevisiae8.mapping.paf.edges.weights.txt` is a list of edge weights (long and high estimated identity mappings have greater weight);
- `scerevisiae8.mapping.paf.vertices.id2name.txt` is the 'id to sequence name' map.

To identity the communities, execute:

    python3 ~/git/pggb/scripts/net2communities.py \
        -e scerevisiae8.mapping.paf.edges.list.txt \
        -w scerevisiae8.mapping.paf.edges.weights.txt \
        -n scerevisiae8.mapping.paf.vertices.id2name.txt

How many communities were detected?

The `paf2net.py` script creates a set of `*.community.*.txt` files one for each the communities detected. Each `txt` file lists the sequences that belong to the same community.

Are there communities that contain multiple chromosomes? Which ones?

Identity the communities again, but this time add the `--plot` option to visualize them too:

    python3 ~/git/pggb/scripts/net2communities.py \
        -e scerevisiae8.mapping.paf.edges.list.txt \
        -w scerevisiae8.mapping.paf.edges.weights.txt \
        -n scerevisiae8.mapping.paf.vertices.id2name.txt \
        --plot

Take a look at the `scerevisiae8.mapping.paf.edges.list.txt.communities.pdf` file.

Write a little script that take the `*.community.*.txt` files in input and create the corresponding FASTA files, ready to be input to `pggb`. Run `pggb` on the communities with multiple chromosomes and compare the results (layout and variants) from the previous activities.

### Back

Back to [main page](../index.md).
