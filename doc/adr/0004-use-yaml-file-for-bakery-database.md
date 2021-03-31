# 4. Use yaml file for Bakery database

Date: 2021-03-29

## Status

Accepted

## Context

We need to keep track of what bakeries exist as part of the system

## Decision

We will create a repository called `pangeo-forge/bakery-database`.
Inside it, we will have a file called `bakeries.yaml`.

### Example `bakeries.yaml`

That file will look like this

```yaml
---
ldeo_aws_us_west2:
  description: >
    Main bakery used for development and testing of Pangeo Forge.
    This bakery is operated by Lamont Doherty Earth Observatory of Columbia University.
  region: aws-us-west2
  admins:
    - name: "Ryan Abernathey"
      orcid: "0000-0001-5999-4917"
      github: rabernat
  targets:
    pangeo_forge_aws_us_west2:
      region: aws-us-west2
      description: "S3 Bucket For General Use"
      private:
        protocol: s3
        prefix: /pangeo_forge_aws_us_west2/
        storage_options:
          key: {AWS_ACCESS_KEY_ID}
          secret: {AWS_SECRET_ACCESS_KEY}
      public:
        protocol: s3
        prefix: /pangeo_forge_aws_us_west2/
        storage_options:
          requester_pays: True
    osn_ncsa:
      region: ncsa
      description: >
        Pangeo Open Storage Network Bucket at National Center for
        Supercomputing Applications
      private:
        protocol: s3
        prefix: /Pangeo/
        storage_options:
          key: {OSN_KEY}
          secret: {OSN_SECRET}
          client_kwargs:
            endpoint_url: 'https://ncsa.osn.xsede.org'
      public:
        protocol: https
        prefix: "ncsa.osn.xsede.org/Pangeo/"
```

Each target should have both a **private** and **public** entry.
The private one is used by the Bakery to store the data.
The public one is used to generate catalog entries for directly opening the data.

### Bakery Name

The bakery name should uniquely identify the bakery by its operator and region.
Use only lowercase letters, numbers, and underscores.

### Admins

This section should use the same synatx to describe people as the recipe `meta.yaml`.

### Storage Options

The `storage_options` should look exactly like an [intake catalog](https://intake.readthedocs.io/en/latest/catalog.html#remote-access).
That's how we encode how to open up the store.

### Secrets

Secrets will be populated from environment variables. The names of the environment variables should follow the standards for each cloud provider.

### Region Definition

We are using an ad-hoc definition of `region` with the syntax `{cloud_provider}-{region_name}`. 

## Consequences

This information will be used by Bakeries to configure their storage targets when executing recipes.
If a Bakery has multiple storage targets (as in this example), the recipe `meta.yaml` will have to pick a specific one.
Catalog services can use this information to discover and crawl buckets.
