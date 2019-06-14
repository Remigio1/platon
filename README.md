# Platon: Plasmid contig classification and characterization.
Author: Oliver Schwengers (oliver.schwengers@computational.bio.uni-giessen.de)


## Contents
- [Description](#description)
- [Input/Output](#inputoutput)
- [Installation](#installation)
- [Usage](#usage)
- [Examples](#examples)
- [Database](#database)
- [Dependencies](#dependencies)
- [Citation](#citation)



## Description
Platon classifies contigs from bacterial WGS short read assemblies as plasmid or
chromosome contigs, i.e. they either originate from a plasmid or a chromosome, respectively.
Therefore, Platon takes advantage of pre-computed protein distribution statistics
and computes mean protein scores (**MPS**) for each contig and finally tests
them against certain thresholds. Contigs below a sensitivity threshold get
classified as chromosome, contigs above a specifivity threshold get classified
as plasmid. Contigs which protein score lies in between these thresholds get
comprehensively characterized and finally classified following an heuristic approach.

In detail Platon conducts three analysis steps. First, it predicts open reading
frames and searches the coding sequences against a database of marker genes.
These are based on the NCBI RefSeq PCLA clusters to which we automatically
pre-computed individual protein scores capturing the probability on which kind of
replicon a certain protein is rather to be found on, i.e. on a plasmid or a
chromosome. Platon then calculates the **MPS** for each contig and either
classifies them as chromosome if the **MPS** is below a sensitivity cutoff
(counting for 95 % sensitivity) or as plasmid if the **MPS** is above a
specificity cutoff, counting for 99.99 % specificity.
These threshold have been calculated by Monte Carlo simulations of artifical
contigs created from closed RefSeq chromosome and plasmid sequences. In a second
step contigs passing the sensitivity filter get comprehensivley characterized.
Hereby, Platon tries to circularize the contig sequences, searches for rRNA,
replication, mobilization and conjugation genes as well as incompatibility group
DNA probes and finally performs a BLAST search against a plasmid database.
In a third step, Platon finally classifies all remaining contigs based on an heuristic
approach, i.e. a decision tree of simple rules exploiting all information at hand.


## Input/Output

### Input
Platon accepts draft genomes in fasta format. If contigs have been assembled with
SPAdes, Platon is able to extract the coverage information stored in contigs names.

### Output
Contigs classified as plasmid sequences are printed as tab separated values to
`STDOUT` comprising the following columns:
- Contig ID
- Length
- Coverage
- \# ORFs
- Protein Score
- Circularity
- Incompatibility Type(s)
- \# Replication Genes
- \# Mobilization Genes
- \# Conjugation Genes
- \# rRNA Genes
- \# Plasmid Database Hits

Additionally, Platon writes the following files into the output directory:
- `<prefix>`.plasmid.fasta: contigs classified as plasmids or plasmodal origin
- `<prefix>`.chromosome.fasta: contigs classified as chromosomal origin
- `<prefix>`.tsv: dense information as printed to STDOUT (see above)
- `<prefix>`.json: comprehensive results and information on each single plasmid contig.
All files are prefixed (`<prefix>`) as the input genome fasta file.


## Installation
Platon can be installed/used in 3 different ways.

Additionally, a separate database must be downloaded which we provide for download
as a zipped tarball:
https://s3.computational.bio.uni-giessen.de/swift/v1/platon/db.tar.gz

### GitHub
1. clone the latest version of the repository
2. download and extract the database

Example:
```
git clone git@github.com:oschwengers/platon.git
wget db `path`
tar -xzf db.tar.gz
rm db.tar.gz
platon/bin/platon --db ./db ...
```

If you move the extracted database directory into the platon directory, PLATON will
automatically recognise it. In this case, the database path doesn't need to be specified:
```
git clone git@github.com:oschwengers/platon.git
wget https://s3.computational.bio.uni-giessen.de/swift/v1/platon/db.tar.gz
tar -xzf db.tar.gz
rm db.tar.gz
mv db $PLATON_HOME
platon/bin/platon ...
```

### Pip
1. install PLATON per pip
2. download and extract the database

Example:
```
pip3 install cb-platon
wget https://s3.computational.bio.uni-giessen.de/swift/v1/platon/db.tar.gz
tar -xzf db.tar.gz
rm db.tar.gz
platon --db ./db ...
```


### Docker
1. download our Docker shell wrapper script
2. download and extract the database
```
wget https://raw.githubusercontent.com/oschwengers/platon/master/platon-docker.sh
wget https://s3.computational.bio.uni-giessen.de/swift/v1/platon/db.tar.gz
tar -xzf db.tar.gz
rm db.tar.gz
platon-docker.sh ./db <genome>
```



Alternatively, just use the Docker image (oschwengers/platon) in order to ease
the setup process.


## Usage
Usage:
```
usage: platon [-h] [--threads THREADS] [--verbose] [--output OUTPUT]
              [--version]
              <genome>

Plasmid contig classification and characterization

positional arguments:
  <genome>              draft genome in fasta format

optional arguments:
  -h, --help            show this help message and exit
  --threads THREADS, -t THREADS
                        number of threads to use (default = number of
                        available CPUs)
  --verbose, -v         print verbose information
  --output OUTPUT, -o OUTPUT
                        output directory (default = current working directory)
  --version             show program's version number and exit
```

## Examples
Simple:
```
platon ecoli.fasta
```

Expert: writing results to `results` directory with verbose output using 8 threads:
```
platon --output ./results --verbose --threads 8 ecoli.fasta
```

With Docker shell script:
```
platon-docker.sh <PLATON_DB> <genome>
```

## Database
Platon depends on a custom database based on NCBI RefSeq nonredundant proteins
(NRP), PCLA clusters, RefSeq Plasmid database, PlasmidFinder db as well as custom
HMM models. These databases (RefSeq release 90) can be downloaded here:
(zipped 1.8, unzipped 2.5 Gb)
`www.lorem.ipsum`

## Dependencies
Platon was developed and tested on Python 3.5.
It depends on BioPython (1.71).

Additionally, it depends on the following 3rd party executables:
- Prodigal (2.6.3) <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2848648> <https://github.com/hyattpd/Prodigal>
- Ghostz (1.0.1) <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4393512> <http://www.bi.cs.titech.ac.jp/ghostz>
- Blast+ (2.7.1) <https://www.ncbi.nlm.nih.gov/pubmed/2231712> <https://blast.ncbi.nlm.nih.gov>
- MUMmer (4.0.0-beta2) <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC395750/> <https://github.com/gmarcais/mummer>
- INFERNAL (1.1.2) <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3810854> <http://eddylab.org/infernal>

Platon has been tested against aforementioned software versions.


## Citation
A manuscript is in preparation... stay tuned!
