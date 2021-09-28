# 1. Use meta.yaml to track feedstock metadata

Date: 2021-03-27

## Status

Proposed

## Context

We need a way to keep track of metadata associated with each feedstock repo.

### Goals

- The metadata should describe everything we need to know about the recipes that is not contained in the recipe python code itself.
- It should facilitate generation of a STAC collection for the data that comes out of the recipe.
- It should tell the bakery everything it needs to know to run the recipe.

### Inspiration

- Conda forge [meta.yaml](https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html).
- [STAC collection example](https://github.com/radiantearth/stac-spec/blob/master/examples/collection.json)


## Decision

We will track metadata in a `meta.yaml` which lives at the top level of the the `feedstock/` directory within the generated feedstock repo.

The format and contents of this file are specified as follows.

### Top Level Data

```yaml
id: noaa-oisst
version: "1.0" 
title: "NOAA Optimum Interpolated SST"
description: "Analysis-ready Zarr datasets derived from NOAA OISST NetCDF"
pangeo_forge_recipes_version: "0.1"
pangeo_forge_metadata_spec_version: "2"
```

| name | description | valid characters |
| -- | -- | -- |
| id | top-level ID for the feedstock. must be unique and must match feedstock repo name | lower case ascii alpha-numeric, no spaces, only dashes allowed |
| version | feedstock version | `{MAJOR}.{̨MINOR}` |
| description | - | Unicode |

#### Version

We use a two-element version of semantic versioning: `MAJOR.MINOR`.
The recipe version starts at 1.0.
Updates to the minor version number are considered minor changes;
when a new minor version is released, bakeries will _overwrite the previous minor version_ of the feedstock's datasets.
Instead, the bakery should _always retain the latest major version_.
Updates that involve concatenation of new elements (e.g. extend the temporal range) are considered minor version updates.
Changes are _minor_ if they will be backwards-compatible with user code which relies on the previous version of the dataset. Changes are _major_ if they have the potential to break existing user code. The following table provides examples for each category:

| Change | Minor | Major |
| -------- | ----- | ------ |
| Updating metadata (e.g. to correct units, descriptions, etc.) | :ballot_box_with_check:  |  |
| Adding new variables or timesteps | :ballot_box_with_check: |  |
| Changing existing variable names | | :ballot_box_with_check: |
| Changing preprocessing functions (that alter existing data) | | :ballot_box_with_check: |

### `recipes` section

The `recipes` section explains what recipes are contained in the repo.
The recipes are defined in python code in a `.py` file (python module).
We need a way to point to python objects in from meta.yaml.
For the simple case of one recipe, it looks like this:

```yaml
recipes:
  - id: avhrr-only
    object: "recipe:recipe"
```

To find the code for this recipe, the parsing library would have to do something link

```python
from recipe import recipe
```
If the repo provides multiple recipes in the same module, it might look like this

```yaml
recipes:
  - id: avhrr-only
    object: "recipe:avhrr_only"
  - id: amsr-avhrr
    object: "recipe:amsr_avhrr"
```

(We could also have multiple distinct modules.)

Some feedstocks may want to provide many recipes in one `.py` file.
In this case, it is not feasible to enumerate every one explicitly in meta.yaml.
Instead, we can point at a python dict directly using `dict_object`.
The keys of the dict will become the recipe ids, and the values the recipe objects.
For example:

```yaml
recipes:
  - dict_object: "recipe:all_recipes
```

Rules for this section:
1. `id` must be unique
2. `id` must use restricted character set compatible with S3
3. `object` uses entrypoints syntax to specify a module and object name to import
   The object must be an instance of a [Recipe object](https://pangeo-forge.readthedocs.io/en/latest/recipes.html#the-recipe-object).
4. Each entry in the `recipes` section must have *either* `id` and `object` *or*  `dict_object`.

### `provenance` section

The provenance section describes _where the data came from_.

#### `providers` section

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

The bakeries will add additional provenance (e.g. "processor") when they generate data.

#### `license` section

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

### `maintainers` section

Who created the recipe. A list with a least one entry.

```yaml
maintainers:
  - name: "Ryan Abernathey"
    orcid: "0000-0001-5999-4917"
    github: rabernat
```

Only `github` is required.

### `bakeries` section

Tells how to bake the recipe.

Resource specifications should conform to [dask worker spec](https://yarn.dask.org/en/latest/configuration.html#example-deploy-mode-local-with-node-label-restrictions).

```yaml
bakeries:
  - id: "pangeo-aws-west-1"  # must come from a valid list of bakeries
    resources:  # mapped to worker settings somehow
      memory: "10 GB"  # this is optional, should have a default of 4GB
```

### Recipe related files

The files used by the Recipes outlined in the `meta.yaml` file must be adjacent or in subdirectories adjacent to the `meta.yaml` file when the PR to `staged-recipes` is raised.

For example, both of the examples below are **valid** structures that we expect:

**Directly Adjacent**
```
a
└── pr
    └── contents
        └── pr-feedstock
            ├── meta.yaml # metadata file is directly adjacent to the recipe related files
            ├── recipe.py
            ├── recipe_2.py
            └── sidecar_file.txt
```

**Adjacent Subdirectories**
```
a
└── pr
    └── contents
        ├── meta.yaml # metadata file is adjacent to subdirectories container recipe related files
        ├── recipe_1
        │   ├── recipe.py
        │   └── utils
        │       └── utils.py
        └── recipe_2.py
```

The `feedstock-creation-action` will use the parent directory of `meta.yaml` to copy all Recipe related files into the generated Feedstock repository.

## Consequences

This format specification will be used by several different parts of the system.

- Users writing new recipes will need to write their `meta.yaml`, ideally starting
  from a nice template or example. More documentation is needed.
- The feedstock github actions will need to parse `meta.yaml` in order to dispatch
  recipes to bakeries. Some of the metadata may need to be injected directly into
  the dataset `attrs`.
- The catalog database will need to parse `meta.yaml` in order to fill in catalog
  metadata for the recipes added by the feedstock.
