# famosab/wrrocmetatest

<!--
[![GitHub Actions CI Status](https://github.com/famosab/wrroc-meta-test/actions/workflows/ci.yml/badge.svg)](https://github.com/famosab/wrroc-meta-test/actions/workflows/ci.yml)
[![GitHub Actions Linting Status](https://github.com/famosab/wrroc-meta-test/actions/workflows/linting.yml/badge.svg)](https://github.com/famosab/wrroc-meta-test/actions/workflows/linting.yml)[![Cite with Zenodo](http://img.shields.io/badge/DOI-10.5281/zenodo.XXXXXXX-1073c8?labelColor=000000)](https://doi.org/10.5281/zenodo.XXXXXXX)
-->
[![nf-test](https://img.shields.io/badge/unit_tests-nf--test-337ab7.svg)](https://www.nf-test.com)
[![Nextflow](https://img.shields.io/badge/nextflow%20DSL2-%E2%89%A524.04.2-23aa62.svg)](https://www.nextflow.io/)
[![run with conda](http://img.shields.io/badge/run%20with-conda-3EB049?labelColor=000000&logo=anaconda)](https://docs.conda.io/en/latest/)
[![run with docker](https://img.shields.io/badge/run%20with-docker-0db7ed?labelColor=000000&logo=docker)](https://www.docker.com/)
[![run with singularity](https://img.shields.io/badge/run%20with-singularity-1d355c.svg?labelColor=000000)](https://sylabs.io/docs/)
<!--
[![Launch on Seqera Platform](https://img.shields.io/badge/Launch%20%F0%9F%9A%80-Seqera%20Platform-%234256e7)](https://cloud.seqera.io/launch?pipeline=https://github.com/famosab/wrroc-meta-test)
-->

## Introduction

**famosab/wrrocmetatest** is a bioinformatics pipeline that runs a minimal working example (fastp + megahit) on a small metagenome data set containing simulated Illumina reads from 15 microbial genomes to test the nf-prov plugin for metagenomics.

![](./wrrocmetatest.excalidraw.png)

The application of this pipeline was to be used as exemplary Nextflow pipeline for the development and testing of the nf-prov Nextflow plugin. More information can be found in the [BioHackathon Europe report](https://github.com/SandyRogers/biohack24-metagenomic-crates-report).

## Using the pipeline to test the nf-prov Nextflow plugin
A small metagenome data set containing simulated Illumina reads from 15 microbial genomes

- https://openstack.cebitec.uni-bielefeld.de:8080/swift/v1/denbi-mg-course/read1.fq.gz
- https://openstack.cebitec.uni-bielefeld.de:8080/swift/v1/denbi-mg-course/read2.fq.gz

To run the pipeline locally on the testdata you need to download the fq files and create the following samplesheet

`testsheet.csv`:
```csv
sample,fastq_1,fastq_2
test,<path/to/>read1.fq.gz,<path/to/>read2.fq.gz
```

And add the following config file

`testdata.config`:
```groovy
process{
    withName: FASTP {
        cpus = 8
        memory = 16.GB
        }
    withName: MEGAHIT {
        cpus = 8
        memory = 16.GB
        ext.args = { "--k-min 51 --k-max 71 --k-step 20" }
        }
}

// add the following lines to run it with nf-prov
plugins {
	id 'nf-prov@1.4.0'
}

prov {
	enabled = true
	formats {
    	wrroc {
        	file = "${params.outdir}/ro-crate-metadata.json"
        	overwrite = true
        	agent {
            	name = "John Doe"
            	orcid = "https://orcid.org/0000-0000-0000-0000"
        	}
          license = "https://spdx.org/licenses/MIT"
          profile = "provenance_run_crate"
    	}
	}
}
```
> [!NOTE]
> Generating workflow run provenance RO-crates is supported by the [nf-prov plugin](https://github.com/nextflow-io/nf-prov/tree/1.4.0) starting from v.1.4.0.

Then you can run the workflow with the following command

```bash
nextflow run <path/to/>wrrocmetatest/main.nf -profile docker --input <path/to/>testsheet.csv --outdir results -c <path/to/>testdata.config
```

Depending on your resources this test run takes around 5 minutes.

### Installation of the nf-prov plugin
Clone the repository with the current working version to your local machine. In our case this is [famosab/nf-prov](https://github.com/famosab/nf-prov). Checkout the relevant branch, here `workflow-run-crate`. Then run `make install`. This will add `nf-prov` to `.nextflow/plugins/` from which it can be used with any pipeline.

## Usage

> [!NOTE]
> If you are new to Nextflow and nf-core, please refer to [this page](https://nf-co.re/docs/usage/installation) on how to set-up Nextflow. Make sure to [test your setup](https://nf-co.re/docs/usage/introduction#how-to-run-a-pipeline) with `-profile test` before running the workflow on actual data.

First, prepare a samplesheet with your input data that looks as follows:

`samplesheet.csv`:

```csv
sample,fastq_1,fastq_2
CONTROL_REP1,AEG588A1_S1_L002_R1_001.fastq.gz,AEG588A1_S1_L002_R2_001.fastq.gz
```

Each row represents a pair of fastq files (paired end).

Now, you can run the pipeline using:

```bash
nextflow run famosab/wrroc-meta-test \
   -profile <docker/singularity/.../institute> \
   --input samplesheet.csv \
   --outdir <OUTDIR>
```

> [!WARNING]
> Please provide pipeline parameters via the CLI or Nextflow `-params-file` option. Custom config files including those provided by the `-c` Nextflow option can be used to provide any configuration _**except for parameters**_; see [docs](https://nf-co.re/docs/usage/getting_started/configuration#custom-configuration-files).

## Credits

famosab/wrroc-meta-test was originally written by Famke Bäuerle, Tom Tubbesing, Keiler Collier, Matt Burridge, Benedikt Osterholz, Alex Sczyrba, Neil Wipat and Sandy Rogers during the [BioHackathon Europe 2024](https://biohackathon-europe.org/).

## Contributions and Support

If you would like to contribute to this pipeline, please see the [contributing guidelines](.github/CONTRIBUTING.md).

## Citations

<!-- If you use famosab/wrroc-meta-test for your analysis, please cite it using the following doi: [10.5281/zenodo.XXXXXX](https://doi.org/10.5281/zenodo.XXXXXX) -->

This pipeline uses code and infrastructure developed and maintained by the [nf-core](https://nf-co.re) community, reused here under the [MIT license](https://github.com/nf-core/tools/blob/main/LICENSE).

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).
