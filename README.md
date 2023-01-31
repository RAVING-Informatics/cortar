
<!-- README.md is generated from README.Rmd. Please edit that file -->

# cortar 

<!-- badges: start -->
<!-- badges: end -->

The goal of cortar is to extract split and non-split reads from RNA-seq
and report the proportion of splicing events across each intron. By comparing
these values to controls, deviations from normal splicing can be characterised
in test samples.

# Installation

- cortar was built in R 4.1.3, compiled for Intel. I recommend installing a program which can manage different R versions.
- Note, I could not get cortar working for R 4.1.3 compiled for arm64-based Macs. I have an arm64-baseed Mac, but thankfully R binaries built for Intel run on Apple silicon.

## Multiple R versions

- Install a program which will let you switch between multiple R versions. I use RSwitch.

### Install RSwitch

- Install RSwitch according to the instructions in the [following link](https://rud.is/b/2020/11/18/apple-silicon-big-sur-rstudio-r-field-report/).
    - Download link [RSwitch-2.0.0b.app.zip](https://rud.is/rswitch/releases/RSwitch-2.0.0b.app.zip)
- Open RStudio and navigate to terminal.
- Following [these instructions](https://code2care.org/macos/macos-r-installation-steps), [and these instructions](https://cran.csiro.au/doc/manuals/r-devel/R-admin.pdf) (page 21) run the following to forget existing R package installation.
    - This is done so that new installations do not override existing installations. Note you will need to adapt these based on what system the binaries were compiled for (mine was a3m64). Use x86_64, if you use Intel.

```bash
sudo pkgutil --forget org.R-project.arm64.R.fw.pkg
sudo pkgutil --forget org.r-project.arm64.texinfo
sudo pkgutil --forget org.r-project.arm64.tcltk.x11 #tried but did not work
sudo pkgutil --forget org.R-project.R.GUI.pkg
sudo pkgutil --forget org.R-project.R.fw.pkg
```

- Find the R v4.1.3 precompiled binary
    - Available [here](https://cran.r-project.org/) for Intel
    - Install it using defaults

## Install cortar on 4.1.3-Intel

- Follow instructions on GitHub
    - If `devtools` isn't installed, then install it.

```r
install.packages("devtools")
```

- Install dependencies

```r
#installation for R v4.1
if (!require("BiocManager", quietly = TRUE))
  install.packages("BiocManager")

BiocManager::install("GenomicRanges")

BiocManager::install("GenomicFeatures")

BiocManager::install("GenomicAlignments")

BiocManager::install("BSgenome.Hsapiens.UCSC.hg38")

BiocManager::install("BSgenome.Hsapiens.UCSC.hg19")

BiocManager::install("BSgenome.Hsapiens.1000genomes.hs37d5")
```

- Install `cortar`

```r
# install.packages("devtools")
devtools::install_github("kidsneuro-lab/RNA_splice_tool")
```

# Usage
To use cortar, a samplefile needs to be created for each run. This file
contains six tab separated columns and a row for each sample (as shown below):
* sampleID: A unique identifier for the sample (alphanumeric symbols and underscores only)
* familyID: A unique identifier for related samples that should not be compared
to one another
* sampletype: Whether analysis and report is desired for this sample ("test")
* genes: Gene symbol for gene under investigation (can be blank for
panel/research mode)
* transcript: Transcript under investigation (blank transcript will default to
the canonical RefSeq transcript)
* bamfile: Absolute or relative file path to the processed bamfile of the sample 

``` r
#> sampleID    familyID   sampletype   genes      transcript       bamfile
#> proband_1   1	  test         DMD        NM_004006	   Z:/path/to/bamfile/proband_1.bam
#> proband_2   2	  test         TTN	  NM_001267550     Z:/path/to/bamfile/proband_2.bam
#> proband_3   3	  test         CFTR	  NM_000492	   Z:/path/to/bamfile/proband_3.bam
#> proband_4   4	  test         NF1	  NM_001042492     Z:/path/to/bamfile/proband_4.bam
#> mother_4    4		       NF1	  NM_001042492     Z:/path/to/bamfile/mother_4.bam
#> proband_5   5	  test         COL2A1	  NM_001844	   Z:/path/to/bamfile/proband_5.bam
```

After creating a samplefile, cortar can be run using the `cortar()` function as follows:

```r

cortar(
  file = "path/to/cortar_samplefile.tsv",
  mode = "default" or "panel" or "research",
  assembly = "hg38" or "hg19",
  annotation = "UCSC" or "1000genomes",
  paired = TRUE or FALSE,
  stranded = 0 or 1 or 2,
  output_dir = "path/to/output/directory",
  genelist = NULL or c("gene1", "gene2", etc)
)

```

Multiple cortar samplefiles can be run in sequence with the `cortar_batch()` function, substituting a path to a folder of samplefiles for the file argument.


## Subsetting
To subset bamfiles to use with cortar, copy the contents of `inst/` to the folder containing `.cram` files
to be subsetted.

#### Prepare cramfile.txt:
1. Remove the contents of `cramfiles_example.txt` and replace with the paths/names of the `.cram` files
to be subsetted.
2. Rename `cramfiles_example.txt` to `cramfiles.txt`

#### Prepare script:

1. Run `subsetBamfiles()` in R using a character vector of genes of interest and the assembly as arguments. For example:
```r
#Get gene coordinates for subsetting script
subsetBamfiles(c("DMD","TTN","COL1A1"), 38)

#> "''chr17:50183101-50202632'' ''chr2:178524989-178831802'' ''chr7:117286120-117716971''"
```
2. Copy the output from `subsetBamfiles()` to the position in subset.sh marked with `#replace this tag with gene coordinates#`.
Exclude the double quotation marks.
3. Add the path to the reference `.fasta` file used for alignment of the RNA-seq data to the position in subset.sh marked with
`#replace this tag with the reference .fasta#`. Do not include quotation marks
4. Add the directory into which the final subsetted files should be saved to the position in subset.sh marked with `#replace
this tag with the path/to/destination/directory#`. Ensure not to remove the double quotation marks.

#### Final steps:
1. Ensure samtools is installed and on the PATH.
2. Run subset.sh script.
3. Update cortar samplefile with subsetted `.bam` file paths

Greater automation of the subsetting process is planned for future versions of cortar.
