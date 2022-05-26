Day 1b - PGGB
===

### Learning objectives

In this exercise you learn how to

- build pangenome graphs using `pggb`,
- explore `pggb`'s results,
- understand how parameters affect the built pangenome graphs.

### Getting started

Make sure you have `pggb` and its tools installed. It is already available on the course workstations. If you want to build everything on your laptop, follow the instructions at the [pggb homepage](https://github.com/pangenome/pggb#installation) (`guix`, `docker`, `singularity`, and `conda` alternatives are available). So make sure you have checked out `pggb` repository:

    cd ~
	git clone https://github.com/pangenome/pggb.git

Check out also `wfmash` repository (we need one of its scrips):

    cd ~
    git clone https://github.com/waveygang/wfmash.git

Now create a directory to work on for this tutorial:

	mkdir day1_pggb
	cd day1_pggb
	ln -s ~/pggb/data

### Build HLA pangenome graphs

The [human leukocyte antigen (HLA)](https://en.wikipedia.org/wiki/Human_leukocyte_antigen) system is a complex of genes on chromosome 6 in humans which encode cell-surface proteins responsible for the regulation of the immune system.

Let's build a pangenome graph from a collection of sequences of the DRB1-3123 gene:

    pggb -i data/HLA/DRB1-3123.fa.gz -n 12 -t 8 -o out_DRB1_3123

Run `pggb` without parameters to get information on the meaning of each parameter:

    pggb

Take a look at the files in the `out_DRB1_3123` folder. Visualize the graph with `Bandage`.

Why did we specify `-n 12`?

How many alignments were executed during the pairwise alignment (take a look at the `PAF` output)? Visualize the alignments:

    cd out_DRB1_3123
    ~/wfmash/scripts/paf2dotplot png large *paf
    cd ..
    
Use `odgi stats` to obtain the graph length, and the number of nodes, edges, and paths. Do you think the resulting pangenome graph represents the input sequences well? Check the length and the number of the input sequences to answer this question.

How many blocks were selected and 'smoothed' during the two rounds of graph normalization (take a look at the `*.log` file to answer this question)?

Try building the same pangenome graph by specifying a lower percent identity (`-p 95` by default):

    pggb -i data/HLA/DRB1-3123.fa.gz -p 90 -n 12 -t 8 -o out2_DRB1_3123
    
Check graph statistics. Does this pangenome graph represent better or worse the input sequences than the previously produced graph?


Try to decrease the number of mappings to reteain for each segment:

    pggb -i data/HLA/DRB1-3123.fa.gz -p 90 -n 6 -t 8 -o out3_DRB1_3123

How does it affect the graph?

Try to increase the target sequence length for the partial order alignment (POA) problem (`-G 4001,4507` by default):

    pggb -i data/HLA/DRB1-3123.fa.gz -p 90 -n 12 -t 8 -G 12000,13000 -o out4_DRB1_3123

How is this changing the runtime and the memory usage? How is this affecting graph statistics? How many blocks were selected and 'smoothed' during the two rounds of graph normalization?

Try 1, 3 or 4 rounds of normalization (for example,by specifying `-G 4001`, `-G 4001,4507,4547`, or `-G 4001,4507,4547, 4999`). How does this affect graph statistics?

Take the second `pggb` run and try to increase the segment length (`-s 10000` by default):

    pggb -i data/HLA/DRB1-3123.fa.gz -s 20000 -p 90 -n 12 -t 8 -o out5_DRB1_3123

How is this affecting graph statistics? Why?

`pggb` produces intermediate graphs during the process. Let's keep all of them:

    pggb -i data/HLA/DRB1-3123.fa.gz -p 90 -n 12 -t 8 --keep-temp-files -o out2_DRB1_3123_keep_intermediate_graphs

What does the file with name ending with `.seqwish.gfa` contain? and what about the file with name ending with `.smooth.1.gfa`?
Take a look at the graph statistics of all the GFA files in the `out2_DRB1_3123_keep_intermediate_graphs` folder.

Choose another HLA gene from the `data` folder and explore how the statistics of the resulting graph change as` s`, `p`,` n` change. Produce scatter plots where on the x-axis there are the tested values of one of the `pggb` parameters (`s`, `p`, or `n`) and on the y-axis one of the graph statistics (length, number of nodes, or number of edges). You can do that using the final graph and/or the intermediate ones.

### Build LPA pangenome graphs

[Lipoprotein(a) (LPA)](https://en.wikipedia.org/wiki/Lipoprotein(a)) is a low-density lipoprotein variant containing a protein called apolipoprotein(a). Genetic and epidemiological studies have identified lipoprotein(a) as a risk factor for atherosclerosis and related diseases, such as coronary heart disease and stroke.

Try to make LPA pangenome graphs. The input sequences are in `data/LPA/LPA.fa.gz`. Sequences in this locus have a peculiarity: which one? Hint: visualize the alignments and take a look at the graph layout (with `Bandage` and/or in the `.draw_multiqc.png` files).
