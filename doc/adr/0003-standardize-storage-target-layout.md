# 2. Standardize storage target layout

Date: 2021-03-28

## Status

Draft

## Context

The main point of Pangeo Forge is to populate object-store-based data libraries.

Object store paths are valid S3 object key names ([S3 docs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html)):
> The name for a key is a sequence of Unicode characters whose UTF-8 encoding is at most 1,024 bytes long.
> You can use any UTF-8 character in an object key name. However, using certain characters in key names can cause problems with some applications and protocols.

Each Bakery will address one or more buckets in object storage.
Our solution should work with many different object storage services:
- AWS S3
- S3-compatible API
  - Wasabi
  - Some OSN Nodes
- Google Cloud Storage
- Azure Blob Storage
- Ceph

Catalog services will have to expose the contents of these buckets via APIs, websites, etc.

It would be great if different parts of Pangeo Forge could use the object store itself
as the master database of what data exist in the system.
In order to enable this, we need to define conventions about the meaning of paths / keys.

## Decision

A **Path** in Pangeo Forge points to a specific **Dataset**.
Every Path should be openable as a high-level dataset in Xarray, Pandas, etc.
The Path is the string used to point to a dataset.

A Path string has the following structure

```
{prefix}/pangeo-forge/{subcatalog_name}/{feedstock_name}/{recipe_name}-{release_tag}.{dataset_type}
```

This template contains four keys:
- `prefix`: the root location for the target. This could just be a bucket name,
  or it could be a sub-path within a bucket.
  **Question: should prefix include the protocol, e.g. s3:// ?**
  For now I am assuming no.
- `subcatalog_name`: [**optional**] the name of a subcatalog to which the dataset belongs. If `subcatalog_name` is not provided, the feedstock will be included in the top-level Pangeo Forge STAC Catalog.
- `feedstock_name`: the name of the feedstock subdirectory within `pangeo-forge/staged-recipes/recipes` (which contains, minimally, a `recipe.py` Python module and associated `meta.yaml`).
- `recipe_name`: the name of the recipe. This maps directly to a value defined in
  the `meta.yaml` file (see ADR 1).
- `release_tag`: A valid multi-digit release tag of the `pangeo-forge-recipes` Python package, with the dots removed; e.g., release tag `0.4.0` becomes `040`.
- `dataset_type`: a suffix like `zarr`, `parquet`, etc.


Example *without* optional `subcatalog_name`:
```
/pangeo-us-west-2/pangeo-forge/noaa_oisst/avhrr_only_040.zarr
```
> This build path will be cataloged as follows:
> - `noaa_oisst`: STAC Collection in the top-level Pangeo Forge STAC Catalog
> - `avhrr_only`: STAC Item in the `noaa_oisst` Collection
> - `avhrr_only.zarr`: STAC Asset of the `avhrr_only` Item

Example *including* optional `subcatalog_name`:
```
/pangeo-us-west-2/pangeo-forge/swot-adac/gigatl/region01-surface-winter.zarr
```
> This build path will be cataloged as follows:
> - `swot-adac`: STAC Catalog within the top-level Pangeo Forge STAC Catalog
> - `gigatl`: STAC Collection in the `swot-adac` Catalog
> - `region01-surface-winter`: STAC Item in the `gigatl` Collection
> - `region01-surface-winter.zarr`: STAC Asset of the `region01-surface-winter` Item

`prefix` can be any valid characters for the object store. We assume that
the bakery manager may not have control over this, e.g. if one organization is
sharing a single bucket among many teams.

For the other three keys, we will use a restricted set of characters.
- the `/` character is interpreted as a path separator and can't be used
- non `/` characters must be valid python identifiers:
  - digits `0-9`
  - lowercase letters `a-z` (lowercase keeps path names clean and pythonic)
  - underscores `_`

> **Note** on `recipe_name`: If your recipe builds a subset of a larger dataset or model, 
the `recipe_name` should identify that subset according to a consistent dash-delimited syntax, e.g.:
```
{spatial_subset}-{variable_subset}-{temporal_subset}
```
Different datasets and domains may adopt their own syntaxes, but efforts should be made to
uphold internal consistency within a given feedstock.

So far the only valid dataset type supported by the recipes is `zarr`.
This is not strictly required by Zarr, but it helps to identify a path that
contains a dataset.

Below the Path, the Dataset itself may have lots of structure and use key names
that don't conform to this spec.

## Consequences

Bakeries can use this specification to determine where to store data.
Catalogs can crawl the bucket and index the datasets.
APIs and CLIs can use these naming conventions to identify and load datasets.
