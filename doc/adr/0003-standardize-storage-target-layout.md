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
{prefix}/pangeo-forge/{feedstock_name}/v{major_version}/{recipe_id}.{dataset_type}
```

This template contains four keys:
- `prefix`: the root location for the target. This could just be a bucket name,
  or it could be a sub-path within a bucket.
  **Question: should prefix include the protocol, e.g. s3:// ?**
  For now I am assuming no.
- `feedstock_name`: the name of the repository that holds the feedstock with
  the recipe that generated the dataset.
- `major_version`: the major version of the feedstock
- `recipe_id`: the ID of the recipe. This maps directly to a value defined in
  the `meta.yaml` file (see ADR 1).
- `dataset_type`: a suffix like `zarr`, `parquet`, etc.


Example:
```
/pangeo-us-west-2/pangeo-forge/noaa_oisst/v1/avhrr_only.zarr
```

`prefix` can be any valid characters for the object store. We assume that
the bakery manager may not have control over this, e.g. if one organization is
sharing a single bucket among many teams.

For the other three keys, we will use a restricted set of characters.
- the `/` character is interpreted as a path separator and can't be used
  **Question: do we want to allow `recipe_name` to include "sub-directories"**?
- non `/` characters must be valid python identifiers:
  - digits `0-9`
  - lowercase letters `a-z` (lowercase keeps path names clean and pythonic)
  - underscores `_`

So far the only valid dataset type supported by the recipes is `zarr`.
This is not strictly required by Zarr, but it helps to identify a path that
contains a dataset.

Below the Path, the Dataset itself may have lots of structure and use key names
that don't conform to this spec.

## Consequences

Bakeries can use this specification to determine where to store data.
Catalogs can crawl the bucket and index the datasets.
APIs and CLIs can use these naming conventions to identify and load datasets.
