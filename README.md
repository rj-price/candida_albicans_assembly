# *C. albicans* Assembly
Documentation of work performed on Candida albicans project in collaboration with the Buscaino laboratory at the University of Kent.

## Sequencing
Long read sequencing libraries were prepared using the SQK-LSK109 Ligation Sequencing Kit with the EXP-NBD104 Native Barcoding Kit (Oxford Nanopore Technologies) from approximately 1μg of high molecular weight genomic DNA, following the manufacturer’s protocol.

Long read libraries were sequenced on R9.4.1 Spot-On Flow cells (FLO-MIN106) using the GridION X5 platform set to super accurate base calling.

Sequencing data have been deposited in the Sequence Read Archive under BioProject PRJNA879282.

## Assembly

### QC, Trim and Filter Reads

Trim adapters from super accurate reads using Porechop
```
porechop-runner.py -i input_dir -b out_dir
```

QC trimmed reads using NanoPlot
```
NanoPlot -t 2 --fastq trimmed.fastq.gz -o out_dir
```

Remove reads below 1kb and keep 90% of the best remaining reads
```
filtlong --min_length 1000 --min_mean_q 87.4 trimmed.fastq.gz | gzip > filtered.fastq.gz
```

QC filtered reads using NanoPlot
```
NanoPlot -t 2 --fastq filtered.fastq.gz -o out_dir
```


### Assembly using NECAT

Generate read list and config files
```
realpath filtered.fastq.gz > read_list.txt
necat.pl config config.txt
```

Edit config files as below
```
PROJECT=SC5314
ONT_READ_LIST=read_list.txt
GENOME_SIZE=16000000
THREADS=16
```

Run assembly
```
necat.pl correct config.txt && \
    necat.pl assemble config.txt && \
    necat.pl bridge config.txt
```

### Assembly polishing and QC

One round of Racon using Raconnn wrapper
```
raconnn 1 filtered.fastq.gz SC5314.fasta > SC5314_racon.fasta
```

One round of Medaka
```
medaka_consensus -i filtered.fastq.gz -d SC5314_racon.fasta -o out_dir -t 16 -m r941_min_high_g360
```

QC final assembly
```
busco -m genome -c 8 -i SC5314_final.fasta -o BUSCO -l saccharomycetes_odb10

assembly_stats.py SC5314_final.fasta
```


## Citation
Rizzo M, Soisangwan N, Vega-Estevez S, <b>Price RJ</b>, Uyl C, et al. (2022) [Stress combined with loss of the Candida albicans SUMO protease Ulp2 triggers selection of aneuploidy via a two-step process.](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1010576) PLOS Genetics 18(12): e1010576. https://doi.org/10.1371/journal.pgen.1010576
