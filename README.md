# A plan for benchmarking decentTree
Compare the running time, memory usage and accuracy of different neighbor-joining software with different input alignments and threads.

## Requirments
  **Script required**
  - timeout (https://github.com/pshved/timeout, for the CPU time and memory usage limitation)
  - pigz
  - Snakemake v5.13.0
  - IQ-Tree (for Multiple sequence alignment conversion, from `.fasta` to `.dist`)
  
  **Software to compare** (can add or remove any software)
  - decenttree
  - quicktree
  - RapidNJ
  - FastME
  - Fasttree
  - BioNJ 

## Usage
### 1. Download the pipeline
```
https://github.com/asdcid/Snakemake-of-neighbor-joining-software.git
```

### 2. Input format
```
1. subread < total reads, if subread > total reads, output all reads

4. converted file format (.sth, dist)
```

***NOTE***
>The name of input alignment file should not contain ':'.

### 3. Modify the configure file `config.yaml`

```
# The path of directory including the input alignments (support MSA in .fasta, .sth and distance matrix format)
INPUTDIR : '/path/of/input/directory'


# The path of output directory
OUTPUTDIR : '/path/of/output/directory'


# Memory limit, kb, 1,000,000,000 = 1 Tb
MEMLIMIT : 1000000000

# CPU limit, seconds, 43200s = 12 hours
TIMELIMIT : 43200


# How many threads you want to use to compare, can be only one value. For example, only use 5 threads: [5]
THREADS : [64, 32, 16, 8, 4, 2, 1]


# The software you want to use to compare, but can only use single thread. Can be empty []
SOFTWARE_SINGLE_THREAD : ['bionj', 'quicktree']

# The software you want to use to compare, but can use multiple threads. Can be empty []
SOFTWARE_MULTIPLE_THREADS : ['decenttree', 'rapidnj', 'fastme', 'fasttree']

```

### 4. Run
```
# $NUM is the threads you want to use, should not be less than the maximum THREADS in `config.yaml`.
snakemake --cores $NUM
```

Each software in `SOFTWARE_SINGLE_THREAD` will run each alignment in the input directory with different threads in `THREADS` (`config.yaml`), and each softare in `SOFTWARE_MULTIPLE_THREADS` will run each alignment in the input directory with single thread.

### 5. Output files

This pipeline outputs three files of each {software}:{threads}:{alignment} combination

```
# The output of timeout and error message during the running. 
# Exceed the CPU time limitation : TIMEOUT 
# Exceed the memory usage        : MEM 
# Not exceeding time and memory  : FINISH
{software}:{threads}:{alignment}.errorLog

# The output of /usr/bin/time, including the CPU times, memory usage
{software}:{threads}:{alignment}.timeLog

# The output tree file
{software}:{threads}:{alignment}.newick
```

For example, if run `decenttree` with `8` threads, and the input alignments is `1000_SSU_NR99.fasta.gz`, 

the output files are:

`decenttree:8:1000_SSU_NR99.fasta.gz.errorLog`

`decenttree:8:1000_SSU_NR99.fasta.gz.timeLog`

`decenttree:8:1000_SSU_NR99.fasta.gz.newick`



### 6. Add new software or remove the default software

***Add new software***

If you want to add new software to this snakemake pipeline, you need to change `config.yaml` and `snakefile`.

**`snakefile`**
- New software with *multiple* threads: add it into `SOFTWARE_MULTIPLE_THREADS` in `config.yaml`. 
 
Such as 
 ```
 SOFTWARE_MULTIPLE_THREADS : ['decenttree', 'new_multiple_software']
 ```

- New software with *single* threads  : add it into `SOFTWARE_SINGLE_THREADS` in `config.yaml`. 

If it is added into the `SOFTWARE_MULTIPLE_THREADS`, it will be repeatly run multiple times (depending on how many threads combination you set in `THREADS`).



The name should be the same in rule (`snakefile`) and in SOFTWARE_SINGLE/MULTIPLE_THREAD (`config.yaml`)

***Remove existing software***

If you don't want to compare some software, just remove them from `SOFTWARE_SINGLE_THREAD` or `SOFTWARE_MULTIPLE_THREADS` in `config.yaml`. 

Don't need to change the `snakefile`.

For example, if you only want to run `decenttree`, just set the `SOFTWARE` in `config.yaml`:
```
SOFTWARE_SINGLE_THREAD : []
SOFTWARE_MULTIPLE_THREADS : ['decenttree']
```



**NOTE**
>1. All software should be in the environment variable, if not, do
```
export PATH="/place/with/the/file":$PATH
```

 


## 1. Original datasets
1.1. Public SARS-CoV-2 alignment: 364,834 sequences (https://github.com/bpt26/parsimony/blob/main/1_sample_selection/28000_samples_less_than_2_ambiguities.fa.xz, 01/04/2021)

1.2. Sliva rRNAs (138.1). https://www.arb-silva.de/download/arb-files/

 - SSU NR99 (small ribosomal subunit): 510,508 sequences, `SILVA_138.1_LSURef_NR99_tax_silva_full_align_trunc.fasta.gz`
 ```
  It is based on the Ref dataset with a 99% criterion applied to remove redundant sequences using the Opens external link in new windowvsearch tool. 
  Sequences from cultivated species have been preserved independent from prior filtering.
  ```

  - SSU: 2,224,740 sequences,  `SILVA_138.1_SSURef_tax_silva_full_align_trunc.fasta.gz`
```
  aligned 16S/18S ribosomal RNA sequences with a minimum length of 1200 bases for Bacteria and Eukarya and 900 bases for Archaea.
```



  - LSU NR99 (large ribosomal subunit): 95,286 sequences,  `SILVA_138.1_LSURef_NR99_tax_silva_full_align_trunc.fasta.gz`
 ```
  It is based on the Ref dataset with a 99% criterion applied to remove redundant sequences using the Opens external link in new windowvsearch tool. 
  Sequences from cultivated species have been preserved independent from prior filtering.
  ```
  - LSU: 227,331 sequences,  `SILVA_138.1_LSURef_tax_silva_full_align_trunc.fasta.gz`
```
 Aligned 23S/28S ribosomal RNA sequences with a minimum length of 1900 bases.
```

## 2. Parameters


 2.2 Subsample datasets (fasta alignments and IQ-TREE distance matrix)
 
  - decenttree (>5000 reads, run out the memory (1TB))
  - quicktree (1 threads, Stockholm format)
  - RapidNJ
  - FastME (although the manual said it allow , but it is not)
  - Fasttree ()
  - BioNJ (cannot install)
 
