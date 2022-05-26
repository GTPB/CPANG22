Day 1a - Introduction
===

### Learning objectives

In this exercise you learn how to

- find toy examples to work with,
- construct graphs using `vg construct`,
- visualize graphs using `vg view` and `Bandage`,
- convert graphs using `vg view`.


### Getting started

Make sure you have `vg` installed. It is already available on the course workstations. If you want to build it on your laptop, follow the instructions at the [vg homepage](https://github.com/vgteam/vg) (note that building `vg` and all submodules from source can take ~1h). In this exercise, you will use small toy examples from the `test` directory. So make sure you have checked out `vg` repository:

    cd ~
	git clone https://github.com/vgteam/vg.git

Now create a directory to work on for this tutorial:

	mkdir day1_intro
	cd day1_intro
	ln -s ~/vg/test/tiny


### Constructing and viewing your first graphs

Like many other toolkits, `vg` is comes with many different subcommands. First we will use `vg construct` to build our first graph. Run it without parameters to get information on its usage:

	vg construct

Let's construct a graph from just one sequence in file `tiny/tiny.fa`, which looks like this:

	>x
	CAAATAAGGCTTGGAAATTTTCTGGAGTTCTATTATATTCCAACTCTCTG

To construct a graph, run

	vg construct -r tiny/tiny.fa -m 32 > tiny.ref.vg

This will create a graph that just consists of a linear chain of nodes, each with 32 characters.

The switch `-m` tells `vg` to put at most 32 characters into each graph node (you might want to run it with different values and observe the different results.) To visualize a graph, you can use `vg view`. Per default, `vg view` will output a graph in [GFA](https://github.com/GFA-spec/GFA-spec/blob/master/GFA1.md) format. By adding `-j` or `-d`, you can generate [JSON](https://www.json.org/) or [DOT](https://www.graphviz.org/doc/info/lang.html) output.

	vg view tiny.ref.vg
	vg view -j tiny.ref.vg
	vg view -d tiny.ref.vg

To work with the JSON output the tool [jq](https://stedolan.github.io/jq/) comes in handy. To get all sequences in the graph, for instance, try

    vg view -j tiny.ref.vg | jq '.node[].sequence'

Next, we use [graphviz](https://graphviz.org/) to layout the graph representation in DOT format

	vg view -d tiny.ref.vg | dot -Tpdf -o tiny.ref.pdf

View the PDF and compare it to the input sequence. Now vary the parameter passed to `-m` of `vg construct` and visualize the result.

Now let's build a new graph that has some variants built into it. First, take a look at at `tiny/tiny.vcf.gz`, which contains variants in (gzipped) [VCF](https://samtools.github.io/hts-specs/VCFv4.2.pdf) format.

	vg construct -r tiny/tiny.fa -v tiny/tiny.vcf.gz -m 32 > tiny.vg

Are there any warnings in the output? If yes, what do they indicate? Visualize the resulting graph.

Ok, that's nice, but you might wonder which sequence of nodes actually corresponds to the sequence (`tiny.fa`) you started from? To keep track of that, `vg` adds a **path** to the graph. Let's add this path to the visualization.

	vg view -dp tiny.vg | dot -Tpdf -o tiny.pdf

You find the output too crowded? Option `-S` removes the sequence labels and only plots node IDs.

	vg view -dpS tiny.vg | dot -Tpdf -o tiny.pdf

Another tool that comes with the `graphviz` package is `Neato`. It creates force-directed layouts of a graph.

	vg view -dpS tiny.vg | neato -Tpdf -o tiny.pdf

For these small graphs, the difference it not that big, but for more involved cases, these layouts can be much easier to read.

It's also possible to read the graph in different formats.
For instance, we can write the graph in GFA, modify it using text processing tools, and read it back in:

    vg view tiny.vg > tiny.gfa

The GFA file has three elements. `S` records represent nodes, `L` records represent edges, and `P` records represent paths.

We can use grep to remove the path lines:

    cat tiny.gfa | grep -v ^P | vg view -dp - | dot -Tpdf -o tiny.no_path.pdf
    
Try to remove the nodes and/or the edges from the GFA file and visualize them. What happens?

Another tool for visualizing (not too big) graphs is [Bandage](https://github.com/rrwick/Bandage). It supports graphs in GFA format. Try to visualize the graph

    Bandage load tiny.gfa 

Let's step up to a slightly bigger example.

	ln -s ~/vg/test/1mb1kgp

This directory contains 1Mbp of 1000 Genomes data for `chr20:1000000-2000000`. As for the tiny example, let's' build one linear graph that only contains the reference sequence and one graph that additionally encodes the known sequence variation. The reference sequence is contained in `1mb1kgp/z.fa`, and the variation is contained in `1mb1kgp/z.vcf.gz`. Make a reference-only graph named `ref.vg`, and a graph with variation named `z.vg`. Look at the previous examples to figure out the command.

Take a look at the length of the graph, and the number of nodes/edges using `vg stats`.

You might be tempted to visualize these graphs (and of course you are welcome to try), but they are sufficiently big already that `neato` can run out of memory and crash. Try to load the graph in `Bandage` too (it will be slow). Try to generate a PNG image with `Bandage` (take a look at the `Bandage -h` output).



### Spoiler: build graphs from sequence alignments

A set of pairwise alignments implies a variation graph, so pangenome graphs can be obtained from alignments too. Use `minimap2` and `seqwish` to build graphs from the HLA gene haplotypes

    ln -s ~/vg/test/GRCh38_alts/FASTA/HLA/
    minimap2 -c -x asm20 -X -t 8 HLA/DRB1-3123.fa HLA/DRB1-3123.fa > DRB1-3123.paf
    seqwish -s HLA/DRB1-3123.fa -p DRB1-3123.paf -g DRB1-3123.gfa
    odgi build -g DRB1-3123.gfa -o - | odgi sort -i - -o DRB1-3123.og
    odgi viz -i DRB1-3123.og -o DRB1-3123.png -x 2000

Use `Bandage` to visualize graphs.
