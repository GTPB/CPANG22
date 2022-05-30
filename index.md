---
layout: page
schemadotorg:
  "@context": http://schema.org/
  "@type": CreativeWork
  "genre": TrainingMaterial

  # Course details
       # "name" -> The acronym and extended name of the course, separated by " - "
       # "description" -> Short description of the course
  name: "CPANG22 - Computational PANGenomics"
  description: "Training Material aimed in the exploration of modern bioinformatic tools that allow researchers to use pangenomes as their reference system when engaging in studies of organisms of all types."

  # Keywords -> Consult EDAM:Topic
keywords:  "http://edamontology.org/topic_0622"

  # Audience -> Following Elixir-Tess input
  audience: ["Academia/ Research Institution", "Industry", "Non-Profit Organisation", "Healthcare"]

  # Author info
  author:
    - "@type": Organization
      name: "The Gulbenkian Training Programme in Bioinformatics"
      alternateName: "GTPB"
      sameAs: "gtpb.igc.gulbenkian.pt/bicourses/index.html"

  # predominant type of learning resources
  "learningResourceType": ["presentation", "exercise", "scripts", "handout"]

  # Contributor info
  contributor:
    - "@type": Person
      name: "Erik Garrison"
    - "@type": Person
      name: "Andrea Guarracino"

  # License & Language & url
  license: https://creativecommons.org/licenses/by/4.0/
  inLanguage: "en-us"
  url: "https://gtpb.github.io/Web_course_template/"
---


## Course Description

Reference genomes are central to most bioinformatics approaches. However, the use of a single reference genome to guide an analysis can result in reference bias: other genomes appear more similar to the reference than they actually are. We might miss, or misunderstand information about genome variability and relationship that cannot be expressed relative to a chosen reference genome. Moreover, new assembly methods are making it easier than ever to generate high-quality complete genome assemblies. To obtain a total understanding of variation between multiple whole genomes, we need to use a model that expresses many genomes and their mutual alignment. A general solution to these problems is to use a pangenome graph wherein genomes are described as labeled walks through an underlying s equence graph. In this course, we will work with methods to build such graphs from whole genome assemblies, and to use the built graphs in an array of downstream applications in comparative genomics, evolution, variation analysis, sequence alignment, and phenotype association.

## Target Audience

This course is oriented towards biologists and bioinformaticians. The course will be of particular interest to researchers investigating organisms without a reference genome or populations featuring high levels of genetic diversity, to researchers in comparative genomics, and to researchers who are assembling pangenomes of any species.

## Detailed Program

### Introduction
- [Slides](assets/CPANG22%20-%20Day%201a%20-%20Introduction.pdf)
- [Activities](pages/Day1a_Introduction.md)

### PGGB
- [Slides](assets/CPANG22%20-%20Day%201b%20-%20PGGB.pdf)
- [Activities - Build your first pangenomes](pages/Day1b_PGGB.md)

### Pangenome visualization
- [Slides](assets/CPANG22%20-%20Day%202a%20-%20Pangenome%20visualization.pdf)

### Homo sapiens pangenomes
- [Activities - Pangenome building](pages/Day2a_Homo_sapiens_pangenome_graphs.md)
- [Activities - Deep exploration](pages/Day3a_Pangenome_subgraphs.md)

### ODGI
- [Slides](assets/CPANG22%20-%20Day%203a%20-%20ODGI.pdf)

### Saccharomyces cerevisiae pangenomes
- [Activities - Divergence estimation and pangenome building](pages/Day3b_Saccharomyces_cerevisiae_pangenome_graphs.md)
- [Activities - Community detection](pages/Day4a_Saccharomyces_cerevisiae_pangenome_graphs.md)

### Read mapping and variant calling
- [Activities](pages/Day4b_Read_mapping.md)

### Other notes
- [Notes](https://hackmd.io/@i-Mv45MZTMa2ZQH2rMAArQ/cpang22notes)


---

### Learning objectives
Participants will develop an understanding of pangenome concepts, and refine this through practical experience with methods to build and interrogate pangenome graphs.
We will apply these methods to difficult study questions wherein we need to understand the relationship between many genomes, or account for variability when we analyze new genomes.
Participants will leave with a deep understanding of pangenome methods based on whole genome assemblies.

### [Instructors](pages/instructors.md)

---

The source for this course webpage is [in github](https://github.com/GTPB/CPANG22).

<br/>

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">Web_course_template</span> by <span xmlns:cc="http://creativecommons.org/ns#" property="cc:attributionName">GTPB</span> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
