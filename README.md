# TIGER v2.0: Tree-Independent Generation of Evolutionary Rates

This page describes the software implementation of the TIGER method. For more information on the algorithm itself,
please see our publication: http://sysbio.oxfordjournals.org/content/60/6/833.short

Unlike the initial version, TIGER v2.0 is split into 3 modes, allowing for comparisons to be run concurrently. This can
prove very useful when running analyses on compute clusters and this software has been designed with job arrays in mind.

First, the input file needs to be broken down and indexed (index mode). It can be broken into as many pieces as you
wish, all of which can be compared on separate processors. Each site needs to be compared to every other site in the
alignment for the rate calculation, so an index file for the full dataset will be generated too. Each piece must be
compared to it (reference index). This will be named like `*.ref.ti`

Next, the rate calculation must be performed (rate mode). Each index file should be compared to the reference index and
can be run independently of the other index files.

Finally, the data can be combined and output in a number of formats with options to mask or remove offending sites.

### Modes:

* _index_:    prepare the data for rate calculation, generate 'tiger index' (`.ti`) file(s).
* _rate_:     preform calculation of tiger rate for each site, create 'generated rates' (`.gr`) file(s).
* _output_:   write sequence files based on `.gr` files, integrate rates from a split analysis into a single file, mask
  and edit alignment based on tiger rates. Please note that this function will not really do anything unless an --excl
  or --incl option is specified.

## Installation

Tiger is very much a standalone package and can be used out-of-the-box by simply running the tiger executable

## Using tiger

### tiger *index* options:

`-i|input`

Specify input file. File must be in FastA format and must be aligned prior. Datasets with uneven sequence lengths will
return an error.

`-s|split`

Split dataset across multiple files to run simultaneously. Takes int argument.

`-o|output`

Specify the prefix name of output files.

`-u|unknowns`

Specify unknown characters in the alignment. Unknown characters are omitted from site patterns and so are not considered
in the analysis. `-u ?,-,*` : defines `?`, `-` and `*` as unknown characters. (Be sure to put only a comma between
characters, **NO SPACE!!**) Default is `?` only

**Examples:**

1. Generate a `.ti` file for complete sequence named `full_seq.ti` & set unknowns to `?` and `-` :

```bash
    tiger index -i sample_data/test_alignment.aln -o test_alignment -u ?,-
```

2. Generate 10 subsets of the data with an output prefix of `tiger_split` and a reference:

```bash
    tiger index -i sample_data/test_alignment.aln -o test_alignment_split -s 10
```

Results in files named `tiger_split.0.ti`, `tiger_split.1.ti`, and so on, along with `tiger_split.ref.ti`

### tiger *rate* options:

`-i|input`

Specify input file. File should be in `.ti` format.

`-r|reference`

Specify reference sequence (`.ti`). `--input` file is used as default if none is provided.

`-o|output`

Specify prefix name for output files.

`-rl|rate_list`

A list of the rate at each site may be optionally written to a specified file. `-rl <file.txt>` : writes list of the
rates at each site to file.txt. Default is `<input_file_prefix>.rates` if no filename is specified

**Examples:**

1. Calculate rates for file test.ref.ti against itself and create a file containing a list of rates:

```bash
    tiger rate -i test_alignment.ref.ti -o test_alignment
```

2. Calculate rates for file `test.0.ti` against the reference index (`test.ref.ti`)

```bash
    tiger rate -i test_alignment.0.ti -r test_alignment.ref.ti -o test_alignment
```

### tiger *output* options:

`-i|input`

Specify input file. Must be in `.gr` format.

`-c|combine`

Specify input file. This file should contain a list of `.gr` files to be combined.

`-fa|fasta`

Provide original `.fa` file for sequence data.

`-o|output`

Specify prefix name for output files.

`-f|format`

Changes formatting options.

NEXUS, with comments:

* `-f 0` : output bin numbers, sites unsorted
* `-f 1` : output bin number, sites sorted on rank
* `-f 2` : displays rank values rather than bin numbers
* `-f 3` : displays rank values and sites sorted on rank
* `-f 5` : \*NEW\* basic nexus output

FastA (default):

* `-f 4`

`-inc|include_only`

Give list of bins to include. `-inc 3,4,5,6` (Note: No spaces, just commas)

`-exc|exclude_only`

Give list of charsets to exclude. `-exc 1,2,9,10`

`-m|mask`

Mask `--include_only`/`--exclude_only` sites rather than removing them (default).

`-b|bins`
Set the number of bins to be used. `-b <int>` : Sites will be placed into `<int>` number of bins. `<int>` is a whole
number. Default is 10. Note: Bin 1 always contains constant sites only.

**Examples:**

1. Write a FastA file, masking site that fall into bin 1, bin 2, bin 9 and bin 10 of 10 bins:

```bash
    tiger output -i test_alignment.gr -fa sample_data/test_alignment.aln -exc 1,2,9,10 -b 10 --mask
```

2. Write a NEXUS file combining test.0.gr, test.1.gr, test.2.gr with sites sorted on rank

```bash
    tiger output -c list_of_gr_files.txt -fa sample_data/test_alignment.aln -f 3
```

## Tiger and Python

Tiger has been written in python. The structure of the software is, essentially, a script calling modules from its
sister package - biotiger. This, then, means that users of the software may avail of all the methods used to perform the
tiger analysis.

It may be noted that both `.ti` and `.gr` files are python cPickle dumps and can be loaded and browsed using iPython (or
similar)

* cPickle: https://docs.python.org/2/library/pickle.html#module-cPickle
* iPython: https://ipython.org/

### Examples

