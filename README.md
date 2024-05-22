<p align="center"><img src="pangeo-forge-logo-blue.png" /></p>

# Pangeo Forge Public Roadmap

🆕 Updated May 2024 🆕 

In this repository, you can find a high level overview of the Pangeo Forge project, its subprojects, how they fit together, and the road ahead.
Pangeo Forge is a community driven project so please open [issues](https://github.com/pangeo-forge/roadmap/issues) to ask questions or to propose changes and/or additions to the roadmap itself.
Pangeo Forge has grown out of the [Pangeo Project](http://pangeo.io/), an open-source community promoting open, reproducible, and scalable science. 

## Inspiration

Pangeo Forge is inspired by the very successful pattern of [Conda Forge](https://conda-forge.org/).
Conda Forge makes it easy for anyone to create a [conda package](https://docs.conda.io/projects/conda/en/latest/user-guide/concepts/packages.html), a binary software package that can be installed with the conda package manager.
In Conda Forge, a maintainer contributes [a recipe](https://conda-forge.org/#add_recipe) which is used to generate a conda package from a source code tarball. Behind the scenes, CI downloads the source code, builds the package, and uploads it to a repository.
By automating the difficult parts of package creation, Conda Forge has enabled the open-source community to collaboratively maintain a huge and dynamic library of software packages.

## Vision

Pangeo Forge aspires to be like Conda Forge, but for data — specifically, for Analysis Ready, Cloud Optimized (ARCO) data. For a detailed working definiton of ARCO data, see our paper [Cloud Native Repositories for Big Scientific Data](https://ieeexplore.ieee.org/abstract/document/9354557).

We envision a vibrant, dynamic library of open-access ARCO data stored in public clouds, shared among thousands of scientists and directly accessible to data-proximate computing.

However, manually populating such a library would be prohibitively difficult and tedious. Instead, we are building Pangeo Forge to automate the production of ARCO data and enable the crowdsourcing of such a data library.

Execution environments, managed by different institutions, are used to automate and scale the pipeline of downloading original files from their source (e.g. FTP, HTTP, or OpenDAP), combining them into one coherent dataset (e.g. using xarray), and writes the data in a cloud optimized format (e.g. Zarr) to cloud storage in a streaming fashion. Object storage may also be considered a data store but the download step may be skipped in cases where the execution environment is deployed in the same region as the same cloud environment executing the recipe.

## Technical Concepts and Architecture

Pangeo Forge’s high-level infrastructure includes:

1. Recipes
2. Execution environments

### Recipes

In Pangeo Forge, recipes generate analysis-ready cloud-based copies of datasets in a cloud-optimized format like Zarr. A recipe defines how to transform data in one format + location into another format + location. Recipes reduce the technical burden on scientists, enabling them to contribute without needing in-depth knowledge of cloud infrastructure. 

The standalone python package [`pangeo-forge-recipes`](https://github.com/pangeo-forge/pangeo-forge-recipes) consists of configurable and chainable algorithms for ARCO data production. These recipes can be used in a standalone fashion, without integration with an execution environment or they can be deployed into an execution environment for offline and scaling features.

### Execution Environments (aka Bakeries)

Execution environments (aka "bakeries") are automated systems for executing these recipes on a distributed system for scalability. Execution environments could be constructed and configured for running recipes using many frameworks, such as flink, spark, dask and possibly many others.

We hope that eventually there will be one or more Pangeo Forge bakeries for all major cloud providers.

There are a number of execution environments in development at time of writing. Reach out to the current Pangeo Forge development team (via community meetings, see the Governance section) for the latest details.

- A pyspark runner: https://github.com/moradology/beam-pyspark-runner) with AWS EMR deployment in-progress.
- A flink runner: https://github.com/NASA-IMPACT/veda-pforge-job-runner
- It is also possible to use a DirectRunner with options for multi_processing set to scale things without a managed distributed infrastructure.

### Pangeo Forge Website

<https://github.com/pangeo-forge/pangeo-forge-vue-website>

## Governance

Pangeo Forge is a community-driven project with lots of work to do and lots of room for contributors to engage. Please join one of our community meetings which are included on the calendar of the [Pangeo Meeting Schedule and Notes page](https://pangeo.io/meeting-notes.html)

* Open Pangeo Forge coordination meetings will be held every 2 weeks. The agenda will be flexible to the needs of current Pangeo Forge developers to address open and context-heavy questions as a group. This is a meeting for Pangeo Forge developers to focus on getting pangeo-forge-recipes and the runner to a new stable release.
* Open Pangeo Forge jam sessions will be held once a week. This agenda will focus on troubleshooting or knowledge sharing of Pangeo Forgee developers on active development tasks.
* Community recipe development: Previously, all recipes were submitted to staged-recipes. Soon we will have a new method for developing and contributing recipes:
    * There will be a template (Github repository template or cookiecutter) for users to get started with a skeleton recipe. Users are encouraged to create a recipe in their own organization. Once the recipe has been developed and tested, users can optionally request to transfer the recipe repo to the official pangeo-forge organization.
    * Questions about recipes and datasets could be asked on those individual repositories.
* Github discussions in pangeo-forge should be used for higher level issues.

## Roadmap May 2024 - October 2024

### Recipes

* We expect to test and document the following functionality:
    * parquet kerchunk reference generation and append (MUR SST)
    * OpenWithXarray supports Zarr
 * Methods for validating zarr stores and chunk manifests will be developed or recommended.

### Execution environments

* Pressure test and release the pyspark beam runner through testing existing and new recipes, such as MUR SST kerchunk, CMIP6 and GPM IMERG. Documentation will provide instructions on how to deploy and use the pyspark beam runner.
* A pyspark beam runner will be cloud agnostic but we anticipate maintaining a stable deployment on one cloud provider in the short term.

### Documention + Governance

* [Documentation](https://pangeo-forge.readthedocs.io) will be updated to reflect changes, such as the migration from per-recipe feedstocks for deployment on a single centralized runner (formerly known as a bakery) to decentralized recipe actions and execution environments.
* Move recipes out of staged-recipes or forked repos into their own repos
* Archive feedstocks in the pangeo-forge organization
* Create a template for recipe development
* Migrate pangeo-forge jam sessions to a shared calendar

------

<p xmlns:dct="http://purl.org/dc/terms/" xmlns:cc="http://creativecommons.org/ns#" class="license-text">This work is licensed under <a rel="license" href="https://creativecommons.org/licenses/by/4.0">CC BY 4.0<img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/cc.svg?ref=chooser-v1" /><img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/by.svg?ref=chooser-v1" /></a>.</p>
