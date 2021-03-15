# Pangeo Forge `meta.yaml` Draft Specification

The `meta.yaml` file lives in a pangeo forge recipe feedstock repo.
Here we document the format and contents of this file.

## Goals

- The `meta.yaml` file should describe everything we need to know about the recipes that is not contained in the recipe python code itself.
- It should facilitate generation of a STAC collection for the data that comes out of the recipe.
- It should tell the bakery everything it needs to know to run the recipe.


## Inspiration

- Conda forge [meta.yaml](https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html). 
- [STAC collection example](https://github.com/radiantearth/stac-spec/blob/master/examples/collection.json)

## Top Level Data

```yaml
title: "NOAA Optimum Interpolated SST"
description: "Analysis-ready Zarr datasets derived from NOAA OISST NetCDF"
pangeo_forge_version: "0.0.1"  # do we need a separate spec version
```

### Questions
- Is this enough? What else might we want to include?
- I'm thinking we don't need a top-level `ID` attribute. That will be determined automatically based on the repo name.
- Spec version as well?

## `recipes` section


The `recipes` section explains what recipes are contained in the repo. For the simple case of one recipe, it looks like this

```yaml
recipes:
  - id: noaa-oisst-avhrr-only
    module: recipe.py  # the module where to find the recipe
    name: recipe  # the name of the object to import
```

To find the code for this recipe, the parsing library would have to do something link

```python
from recipe import recipe
```
If the repo provides multiple recipes in the same module, it might look like this

```yaml
recipes:
  - id: noaa-oisst-avhrr-only
    module: recipe.py
    name: recipe_avhrr_only
  - id: noaa-oisst-amsr-avhrr
    module: recipe.py  
    name: recipe_amsr_avhrr
```

(We could also have multiple distinct modules.)

Rules for this section:
1. `id` must be unique
2. `id` must use restricted character set compatible with S3
3. `name` must be a valid python identifier

### Questions

- Is `id` redundant? Could it be generated based on the recipe module name?

## `provenance` section

The provenance section describes _where the data came from_.

### `providers` section

This section mirrors the [STAC providers object](https://github.com/radiantearth/stac-spec/blob/master/collection-spec/collection-spec.md#provider-object).

```yaml=
provenance:
  providers:
    - name: "NOAA NCEI"
      description: "National Oceanographic & Atmospheric Administration National Centers for Environmental Information"
      roles:
        - producer
        - licensor
      url: https://www.ncdc.noaa.gov/oisst
      
```

### `license` section

This section mirrors the [STAC license section](https://github.com/radiantearth/stac-spec/blob/master/collection-spec/collection-spec.md#license):

> license(s) as a SPDX [License identifier](https://spdx.org/licenses/). Alternatively, use `proprietary` (see below) if the license is not on the SPDX license list or `various` if multiple licenses apply. In all cases links to the license texts SHOULD be added, see the `license` link relation type. If no link to a license is included and the `license` field is set to `proprietary`, the Collection is private, and consumers have not been granted any explicit right to use the data.

However, we don't want to use STAC "links", so we need to specialize a bit. Instead, we will add an optional `url` key for `proprietary` license.

Examples

```yaml
provenance:
  ...
  license: "CC-BY-NC-4.0"
```

```yaml
provenance:
  ...
  license: "proprietary"
  url: https://www.ncdc.noaa.gov/oisst/license.txt  # not a real link
```

Presumably other entries will be added to this before it becomes a STAC catalog (`processor`, `host`, etc.).

## `maintainers` section

Who created the recipe. A list with a least one entry.

```yaml
maintainers:
  - name: "Ryan Abernathey"
    orcid: "0000-0001-5999-4917"
    github: rabernat
```

### Questions
- What is required? Only github I think.

## `bakeries` section

Tells how to bake the recipe.

Resource specifications should conform to [dask worker spec](https://yarn.dask.org/en/latest/configuration.html#example-deploy-mode-local-with-node-label-restrictions).

```yaml
bakeries:
  - id: "pangeo-aws-west-1"  # must come from a valid list of bakeries
    resources:  # mapped to worker settings somehow
      memory: "10 GB"  # this is optional, should have a default of 4GB
```

### Questions
- If a bakery has multiple possible targets, how do we specify this?


## Suggestions

These came up at today's meeting

- extras section?
- json schema
- or [yamale](https://github.com/23andMe/Yamale)? https://github.com/nrkno/yaml-schema-validator-github-action
- Dependencies?
