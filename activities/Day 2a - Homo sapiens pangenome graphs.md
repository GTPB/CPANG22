Day 2a - Homo sapiens pangenome graphs
===

### Learning objectives

In this exercise you learn how to

- collect and preprocess *de novo* human assemblies,
- partition the assembly contigs by chromosome,
- built chromosome-specific pangenome graphs.


### Getting started

Make sure you have `pggb`, its tools, and `gfaestus` installed.

Now create a directory to work on for this tutorial:

    cd ~
	mkdir day2_hsapiens
	cd day2_hsapiens

Get the URLs of the assemblies:

    mkdir -p ~/day2_hsapiens/assemblies
    cd ~/day2_hsapiens/assemblies

    wget https://raw.githubusercontent.com/human-pangenomics/HPP_Year1_Assemblies/main/assembly_index/Year1_assemblies_v2_genbank.index
    grep 'chm13\|h38' Year1_assemblies_v2_genbank.index | awk '{ print $2 }' | sed 's%s3://human-pangenomics/working/%https://s3-us-west-2.amazonaws.com/human-pangenomics/working/%g' > refs.urls
    grep 'chm13\|h38' -v Year1_assemblies_v2_genbank.index | awk '{ print $2; print $3 }' | sed 's%s3://human-pangenomics/working/%https://s3-us-west-2.amazonaws.com/human-pangenomics/working/%g' > samples.urls
    
Download the 2 haplotypes of the `HG01978` diploid sample and the references:

    cat refs.urls <(grep 'HG01978' samples.urls | head -n 6) | parallel -j 4 'wget -q {} && echo got {}'

Unpack the assemblies:

    ls *.f1_assembly_v2_genbank.fa.gz | while read f; do echo $f; gunzip $f && samtools faidx $(basename $f .gz); done
    

### Pangenome Sequence Naming

We follow the [PanSN-spec](https://github.com/pangenome/PanSN-spec) naming to simplify the identification of samples and haplotypes in pangenomes. We do that by adding a prefix to the reference sequences:

    fastix -p 'grch38#1#' <(zcat GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.gz) | bgzip -c -@ 8 > grch38_full.fa.gz && samtools faidx grch38_full.fa.gz
    fastix -p 'chm13#1#' <(zcat chm13.draft_v1.1.fasta.gz) | bgzip -c -@ 8 > chm13.fa.gz && samtools faidx chm13.fa.gz

    # Remove unplaced contigs from grch38 that are (hopefully) represented in chm13
    samtools faidx grch38_full.fa.gz $(cat grch38_full.fa.gz.fai | cut -f 1 | grep -v _ ) | bgzip -@ 8 -c > grch38.fa.gz

    # Cleaning
    rm chm13.draft_v1.1.fasta.gz GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.gz grch38_full.fa.gz.*

Take a look at how sequence names are changed in the FASTA files.


### Sequence partitioning

Put the two references together:

    zcat chm13.fa.gz grch38.fa.gz | bgzip -c -@ 8 > chm13+grch38.fa.gz && samtools faidx chm13+grch38.fa.gz

Why are we using two reference genomes? Does the old `grch38` reference have chromosomes that the `chm13` reference does not have? If so, which ones?

We partition contigs by chromosome by mapping each assembly against the scaffolded references:

    PATH_REFERENCE_FA_GZ=~/day2_hsapiens/assemblies/chm13+grch38.fa.gz

    mkdir -p ~/day2_hsapiens/assemblies/partitioning

    ls *.f1_assembly_v2_genbank.fa | while read FASTA; do
      NAME=$(basename $FASTA .fa);
      echo $NAME

      PATH_PAF=~/day2_hsapiens/assemblies/partitioning/$NAME.vs.ref.paf
      wfmash $PATH_REFERENCE_FA_GZ ~/day2_hsapiens/assemblies/$FASTA -s 50k -p 90 -N -m -t 8 > $PATH_PAF
    done
    
Run `wfmash` without parameters to get information on the meaning of each parameter. What does `-N` mean?

For each haplotype (for each FASTA file), count how many contigs were partitioned in each reference chromosome. Which of the two references (`grch38` and` chm13`) do more contigs map to? If there is a clear winner, why?

What is the chromosome against which more contigs map? Make a plot displaying the number of contigs (on the y-axis) with respect to the chromosome (on the x-axis) to which they map. Consider `grch38` and `chm13` in the same way: for example, merge the number of contigs mapping against `grch38#1#chr1` and `chm13#1#chr1` in a single count for `chr1`.

Collect unmapped contigs and remap them in split mode:

    ls *.f1_assembly_v2_genbank.fa | while read FASTA; do
      NAME=$(basename $FASTA .fa);
      echo $NAME

      PATH_PAF=~/day2_hsapiens/assemblies/partitioning/$NAME.vs.ref.paf
      comm -23 <(cut -f 1 $FASTA.fai | sort) <(cut -f 1 $PATH_PAF | sort) > $NAME.unaligned.txt

      wc -l $NAME.unaligned.txt
      if [[ $(wc -l $NAME.unaligned.txt | cut -f 1 -d\ ) != 0 ]];
      then 
        samtools faidx $FASTA $(tr '\n' ' ' < $NAME.unaligned.txt) > $NAME.unaligned.fa
        samtools faidx $NAME.unaligned.fa

        PATH_NO_SPLIT_PAF=~/day2_hsapiens/assemblies/partitioning/$NAME.vs.ref.no_split.paf
        wfmash $PATH_REFERENCE_FA_GZ ~/day2_hsapiens/assemblies/$NAME.unaligned.fa -s 50k -p 90 -m -t 8 > $PATH_NO_SPLIT_PAF
      fi
    done

Collect our best mapping for each of our attempted split rescues:

    cd ~/day2_hsapiens/assemblies/partitioning

    ls *.vs.ref.no_split.paf | while read PAF; do
      cat $PAF | awk -v OFS='\t' '{ print $1,$11,$0 }' | sort -n -r -k 1,2 | \
        awk -v OFS='\t' '$1 != last { print($3,$4,$5,$6,$7,$8,$9,$10,$11,$12,$13,$14,$15); last = $1; }'
    done > rescues.paf
    
Subset by chromosome, including the references:

    cd ~/day2_hsapiens/assemblies

    DIR_PARTITIONING=~/day2_hsapiens/assemblies/partitioning

    mkdir -p parts

    ( seq 22; echo X; echo Y; echo M ) | while read i; do
      awk '$6 ~ "chr'$i'$"' $(ls $DIR_PARTITIONING/*.vs.ref.paf | \
        grep -v unaligned | sort; echo $DIR_PARTITIONING/rescues.paf) | cut -f 1 | sort | uniq \
        > parts/chr$i.contigs;
    done

    ( seq 22; echo X; echo Y; echo M ) | while read i; do
      echo chr$i
      samtools faidx chm13+grch38.fa.gz chm13#1#chr$i grch38#1#chr$i > parts/chr$i.pan.fa

      ls *.f1_assembly_v2_genbank.fa | while read FASTA; do
        NAME=$(basename $FASTA .fa);
        echo chr$i $NAME

        samtools faidx $FASTA $( comm -12 <(cut -f 1 $FASTA.fai | sort) <(sort parts/chr$i.contigs) ) >> parts/chr$i.pan.fa
      done

      bgzip -@ 8 parts/chr$i.pan.fa && samtools faidx parts/chr$i.pan.fa.gz
    done


### Pangenome graph building

Build the pangenome graph for chromosome 20. Since human presents a low sequence divergence, we will set the mapping identity (`-p` parameter) in `pggb` to `98`.

    mkdir -p ~/day2_hsapiens/graphs

    pggb -i ~/day2_hsapiens/assemblies/parts/chr20.pan.fa.gz -o ~/day2_hsapiens/graphs/chr20.pan -t 8 -p 98 -s 5000 -n 4 -k 311 -G 4000 -O 0.03 -T 8


Run `pggb` without parameters to get information on the meaning of each parameter:

    pggb

Why did we specify `-n 4`?

The `-k` parameter is used to filter exact matches shorter than 311 bps. Graph induction with `seqwish` often works better when we filter very short matches out of the input alignments. In practice, these often occur in regions of low alignment quality, which are typical of areas with large indels and structural variations in the `wfmash` alignments. This underalignment is then resolved in the final `smoothxg` step. Removing short matches can simplify the graph and remove spurious relationships caused by short repeated homologies.

The `-O` parameter specifies to expand a little bit the partial order alignment (POA) blocks at both sides. This help in better resolving POA blocks boundaries and avoiding wrong alignments due to not optimal block selection.

**Try this at home**: build the pangenome graph for all chromosomes.

Use `odgi stats` to obtain the graph length, and the number of nodes, edges, and paths. Do you think the resulting pangenome graph represents the input sequences well? Check the length and the number of the input sequences to answer this question.

Take a look at the PNG files in the `chr20.pan` folder. Is the layout of the graph roughly linear? Are there unaligned regions?

Generate another `odgi viz` visualization with

    cd ~/day2_hsapiens/graphs/chr20.pan
    odgi paths -i chr20.pan.fa.gz.d8a8cb6.04f1c29.5486fb6.smooth.final.og -L | cut -f 1,2 -d '#' | uniq > prefixes.txt
    odgi viz -i chr20.pan.fa.gz.d8a8cb6.04f1c29.5486fb6.smooth.final.og -o chr20.pan.fa.gz.d8a8cb6.04f1c29.5486fb6.smooth.final.og.viz_multiqc.2.png -x 1500 -y 500 -a 10 -I Consensus_ -M prefixes.txt

What do you think is different between the `chr20.pan.fa.gz.d8a8cb6.04f1c29.5486fb6.smooth.final.og.viz_multiqc.png` image and the newly generated image (`chr20.pan.fa.gz.d8a8cb6.04f1c29.5486fb6.smooth.final.og.viz_multiqc.2.png`)?


Go to the folder where the `gfaestus` repository is checked out and open the chart layout with it:

    cd ~/gfaestus
    ./target/release/gfaestus ~/day2_hsapiens/graphs/chr20.pan/chr20.pan.fa.gz.*.smooth.final.gfa ~/day2_hsapiens/graphs/chr20.pan.fa.gz.*.smooth.final.og.lay.tsv
    
Download the coordinates of the p and q amrs on `chm13` in BED format:

    wget https://raw.githubusercontent.com/pangenome/pggb-paper/main/data/chm13.pq_arms.bed

and load the file in `gfaestus`: `Tools` -> `BED Label Wizard` -> Go to the folder where the BED file is -> `Double click on the file` (do not click `Accept`, it is buggy).

Produce full gene annotations and visualize them:

    wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_40/gencode.v40.annotation.gff3.gz
    zcat gencode.v40.annotation.gff3.gz | grep ^chr20 | awk '$3 == "gene"' | cut -f 1,4,5,9 | tr ';' '\t' | grep gene_name= | cut -f 1-3,7 | sed s/gene_name=// | sed s/^chr20/grch38#1#chr20/ > grch38#1#chr20.gene.bed
