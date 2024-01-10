# AYUKA: A toolkit for fast viral genotyping using whole genome sequencing.

AYUKA is a fast viral genotyping toolkit that can analyze raw sequencing reads to determine the viral genotypes present in a sample. It was developed by Jose Afonso Guerra-Assuncao at University College London.

## Installation 

For reproducibility, controlled environment and dependency management, AYUKA is provided as Apptainer (previously known as Singularity) containers. More information and how to install the required software please see [their website.](https://apptainer.org/).

AYUKA can be installed via the Apptainer/Singularity container available on the [GitHub releases page](https://github.com/afonsoguerra/AYUKA). Alternatively, it can be installed from source by following the instructions in the Singularity recipe file.

The key dependencies are:

- Perl ([https://www.perl.org/](https://www.perl.org/))
- R ([https://www.r-project.org/](https://www.r-project.org/)) 
- LaTeX ([https://www.latex-project.org/](https://www.latex-project.org/))
- Jellyfish ([http://www.genome.umd.edu/jellyfish.html](http://www.genome.umd.edu/jellyfish.html))
- SQLite3 ([https://www.sqlite.org/index.html](https://www.sqlite.org/index.html))

## Usage

AYUKA takes as input raw sequencing reads in FASTQ or FASTA format. It can process gzipped files directly.

Please find the singularity container with the full codebase on the releases tab.

The software can be invoked by running:

```./ayuka.simg```

or 

```singularity run ayuka.simg ```
_____________________________________________________________

Method description available in our manuscript on biorXiv:

AYUKA: A toolkit for fast viral genotyping using whole genome sequencing
José Afonso Guerra-Assunção, Richard Goldstein, Judith Breuer
bioRxiv 2022.09.07.506755; doi: https://doi.org/10.1101/2022.09.07.506755

_____________________________________________________________

Associated databases available on Zenodo:

José Afonso Guerra-Assunção. (2022). AYUKA Genotyper Databases - Human Adenovirus (HAdV) (22-222) [Data set]. Zenodo. https://doi.org/10.5281/zenodo.6521576


## Usage

AYUKA takes as input raw sequencing reads in FASTQ or FASTA format. It can process gzipped files directly.

The basic usage is:

```
singularity run ayuka.simg  --seqs sample.fq.gz
```

To analyze paired-end reads:

```
singularity run ayuka.simg --seqs sample_R1.fq.gz,sample_R2.fq.gz
``` 

Or:

```
singularity run ayuka.simg --seqs sample_R1.fq.gz --seqs sample_R2.fq.gz
```

A text file listing multiple sample files can also be provided:

```
singularity run ayuka.simg --fqList list_of_samples.txt
```
In this last case, each line of the list file should contain the files for one sample.


### Filtering Options

`-min_depth`: Minimum k-mer depth to filter noise [default 10]

`-min_bin_frac`: Minimum fraction of bins to be covered [default 0.1]

### Output Options

`-outputDir`: Output directory [default current working directory] 

`-outputFile`: Output file name prefix [default random ID]

### Advanced Options

`-database`: Specify custom k-mer database

`-threads`: Number of threads for Jellyfish

`-skipReport`: Skip PDF report generation

`-pipeline`: Skip report and positional plots for pipelines

## Output

AYUKA produces two main types of output:

1. A PDF report with tables and plots summarizing the analysis

2. Tab-delimited text files with the detailed genotyping results

The PDF report contains:

- Genotype table with p-values
- Coverage and depth estimates
- Plots showing k-mer distributions across the genome

The text files contain the raw data used to generate the report.

## Applications

Some key applications of AYUKA include:

- Rapid preliminary analysis of clinical viral sequencing samples prior to mapping to the best reference
- Distinguishing mixed infections from recombinants
- Detection of virus reads in RNA-seq experiments
- Monitoring viral engineering experiments

The fast speed (under 1 minute per sample) and ability to run on a laptop makes AYUKA suitable for outbreak monitoring and routine screening. The results can guide more in-depth bioinformatics analyses when needed.

## Databases

AYUKA requires a pre-built k-mer database for the viral genotypes of interest. Databases are available for:

- Human adenoviruses
- Adeno-associated viruses (AAV) 
- Epstein-Barr virus (EBV)


## Citation

If you use AYUKA in your work, please cite:

Guerra-Assuncao et al. "AYUKA: A toolkit for fast viral genotyping using whole genome sequencing" *bioRxiv* 2022. https://doi.org/10.1101/2022.09.07.506755


# Creating an AYUKA Database

AYUKA requires a pre-built k-mer database to perform genotyping. The `ayuka.database.conf` configuration file controls database creation.

## Parameters

- **VirusType** - The type of virus to build the database for, e.g. Adenovirus. This selects the appropriate reference sequences.

- **KMERSize** - The k-mer length to use. Larger k-mers are more specific but reduce sensitivity. Typical values are 35-285. Note: Should always be less than sequencing read length being processed. Look at the manuscript for more details on the effects of k-mer size on sensitivity, specificity, memory consumption and speed.

- **SpeciesInfoFile** - A 2 column TSV file listing the genotype IDs and GenBank accessions to include. See `adenovirus.database.info` for an example.

- **BuildThreads** - Number of CPU threads to use. Parallelizes k-mer counting.

- **Number_of_bins** - Number of bins to divide each genome into for positional plots. 100 is a good default.

- **BuildFolder** - Output folder for the database files.

- **NCBI_NT_filter_string** - A string to filter the NT database with. Removes k-mers from unrelated organisms.

## Considerations

- Include representative and high quality RefSeq genomes covering all genotypes of interest

- Adjust k-mer size based on read length and specificity needed

- Use all available threads to parallelize counting and speed up database build

- Filter NT database stringently to keep only informative k-mers

- Use ~100 bins for full genome plots while limiting output file size

- Standardized build folder naming helps manage multiple databases

- Check that genotypes are divergent enough before including together

- Compare number of shared vs specific k-mers to optimize parameters


## Building the Database 

AYUKA provides a dedicated Singularity container to streamline database building.

The default app on the container is a makefile that will take care of the whole database building pipeline.

To build the database, customise the database configuration files and in the same directory run:

```
singularity run ayuka-db.simg
```

This will launch the `ayuka-db.simg` container and perform the database build process using the parameters in the `ayuka.database.conf` file.

The Makefile will:

- Retrieve reference genomes from GenBank
- Run Jellyfish to count k-mers
- Annotate and filter the k-mer set 
- Save the database files

The Singularity container bundles all the required dependencies in a portable runtime. This ensures a consistent environment for database building.

The Makefile automates the entire pipeline from config to final database. Together with the container, it provides a turn-key solution for generating custom AYUKA genotyping databases.
