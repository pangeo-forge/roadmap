# 4. Use yaml file for Bakery database

Date: 2021-03-29

## Status

Draft

## Context

We need to keep track of what bakeries exist as part of the system

## Decision

We will create a repository called `pangeo-forge/bakery-database`.
Inside it, we will have a file called `bakeries.yaml`.

### Example `bakeries.yaml`

That file will look like this

```yaml
---
aws_us_west2:
  description: >
    Main bakery used for development and testing of Pangeo Forge
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
          key: {AWS_KEY_ENV_VAR}
          secret: {AWS_SECRET_ENV_VAR}
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


### Storage Options

The `storage_options` should look exactly like an [intake catalog](https://intake.readthedocs.io/en/latest/catalog.html#remote-access).
That's how we encode how to open up the store.


### Open Questions
- How do we deal with the secrets? I am assuming they will be populated from an environment variable, and here we are giving the name of the environment variable.
- I am using an ad-hoc definition of `region`. Is there a way to standardize this?
- What is required / optional, etc.

## Consequences

This information will be used by Bakeries to configure their storage targets when executing recipes.
If a Bakery has multiple storage targets (as in this example), the recipe `meta.yaml` will have to pick a specific one.
Catalog services can use this information to discover and crawl buckets.
