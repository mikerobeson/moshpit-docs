---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---
# Host read removal
There are a few different options to perform host read removal in QIIME 2: a more generic one using the `filter-reads` action
and a more specific one using the `filter-reads-pangenome` action. Below you can learn how to use both of them. In this tutorial we will 
use the `filter-reads-pangenome` action to remove human reads from the dataset.

## Removal of contaminating reads
Removal of contaminating reads can generally be done by mapping the reads to a reference database and filtering out the reads
that map to it. In QIIME 2 this can be done by using the `filter-reads` action from the `quality-control` plugin. Before filtering
we need to construct the index of the reference database that will be used by Bowtie 2:
- start with the FASTA files containing the reference sequences - we will import them into a QIIME 2 artifact:
```{code-cell}
mosh tools cache-import \
    --cache ./cache \
    --key reference_seqs \
    --type "FeatureData[Sequence]" \
    --input-path ./reference_seeqs.fasta
```
- build the Bowtie 2 index:
```{code-cell}
mosh quality-control bowtie2-build \
    --i-sequences ./cache:reference_seqs \
    --o-database ./cache:reference_index
```
- filter out the reads that map to the reference database:
```{code-cell}
mosh quality-control filter-reads \
    --i-demultiplexed-sequences ./cache:reads_trimmed \
    --i-database ./cache:reference_index \
    --o-filtered-sequences ./cache:reads_filtered
```

## Human host reads
Contaminating human reads can also be filtered out using the approach shown above by providing a human reference genome.
Since a single human reference genome is not enough to cover all the human genetic diversity, it is recommended to use a
collection of genomes represented by the human pangenome (__CIT). We have built a new QIIME 2 action `filter-reads-pangenome`
which allows to first fetch the human pangenome sequence, combine it with the GRCh38 reference genome, build a combined 
Bowtie 2 index and, finally, filter the reads against it. Next to the filtered reads, the action will also return the generated 
index so that it can be used in any other experiments.
```{code-cell}
mosh annotate filter-reads-pangenome \
    --i-reads ./cache:reads_trimmed \
    --o-filtered-reads ./cache:reads_filtered \
    --o-reference-index ./cache:human_reference_index
```
