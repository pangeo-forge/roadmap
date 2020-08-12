# Pangeo Forge public roadmap

In this repository, you can find the the [Pangeo Forge project roadmap](https://github.com/pangeo-forge/roadmap/projects/2).
The roadmap is where you can learn about Pangeo Forge project, its subprojects (i.e. _pangeo-smithy_) and how they fit together, and the road ahead.
Pangeo Forge is just getting started so please open [issues](https://github.com/pangeo-forge/roadmap/issues) to ask questions or to propose changes and/or additions to the roadmap itself.

## Background

The idea of Pangeo Forge is to copy the very successful pattern of [Conda Forge](https://conda-forge.org/) for crowdsourcing the curation of an analysis-ready data library.
In Conda Forge, a maintainer contributes [a recipe](https://conda-forge.org/#add_recipe) which is used to generate a conda package from a source code tarball. Behind the scenes, CI downloads the source code, builds the package, and uploads it to a repository.
In Pangeo Forge, a maintainer contributes a recipe which is used to generate an analysis-ready cloud-based copy of a dataset in a cloud-optimized format like Zarr. Behind the scenes, CI downloads the original files from their source (e.g. FTP, HTTP, or OpenDAP), combines them using xarray, writes out the Zarr file, and uploads to cloud storage.
Pangeo Forge has grown out of the [Pangeo Project](http://pangeo.io/), an open source community promoting open, reproducible, and scalable science. 

## Subprojects

Pangeo Forge brings together a number of smaller subprojects to enable automatic the automatic production and publication of cloud-optimized datasets. Those subprojects are described briefly below:

### pangeo-forge

[Pangeo-forge](https://github.com/pangeo-forge/pangeo-forge) provides a central workflow manager and API for the productions of cloud-optimized datasets.
It is being desinged to include a high-level Pipeline API (built on top of [Prefect](https://www.prefect.io/)) that will be useful inside and outside of pangeo-forge infrastructure.
Read about the pangeo-forge roadmap [here](./subprojects/pangeo-forge.md).

### pangeo-smithy

[Pangeo-smithy](https://github.com/pangeo-forge/pangeo-smithy) is a tool for managing pangeo-forge feedstocks.
It combines a pangeo-forge recipie with the Continuous Integration and Continuous Deployment (CI/CD) servcices.
Read about the pangeo-smithy roadmap [here](./subprojects/pangeo-smithy.md).

### staged-recipies

[Staged-recipies](https://github.com/pangeo-forge/staged-recipes) is a GitHub repository that manages the submission of new pangeo-forge recipies. You can think of this as a holding area for new feedstocks.
Read about the staged-recipies roadmap [here](./subprojects/staged-recipies.md).

## Contributing

Pangeo-forge is just getting started. There's lots of work to do and lots of room for contributors to engage.
Here are a few ways you may consider getting involved:

1. [Document an example recipie](https://github.com/pangeo-forge/staged-recipes/issues/new?assignees=&labels=example&template=example-pipeline.md&title=Example+pipeline+for+%5BDataset+Name%5D)
2. Contribute to any of the subprojects above. At the time of writing (8/11/2020), the pangeo-forge API is the most active area of development.
3. Comment on the project road map in this repository.

## Definitions

- **Pipeline**: A Python object that defines the steps to aquire, convert, and publish a dataset.
- **Feedstock**: A GitHub repository in the pangeo-forge GitHub organization that is managed by pangeo-smithy.

------

<p xmlns:dct="http://purl.org/dc/terms/" xmlns:cc="http://creativecommons.org/ns#" class="license-text">This work is licensed under <a rel="license" href="https://creativecommons.org/licenses/by/4.0">CC BY 4.0<img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/cc.svg?ref=chooser-v1" /><img style="height:22px!important;margin-left:3px;vertical-align:text-bottom;" src="https://mirrors.creativecommons.org/presskit/icons/by.svg?ref=chooser-v1" /></a>.</p>
