# Handing of Recipe Secrets

## Status
Proposed

## Context

Recipes often need to pull data from download locations (e.g. FTP servers) that require authentication with secres (e.g. username / password).
We need a robust and secure way to pass these secrets from the recipe maintainer to the bakery.

## Decision

### Storing Secrets in Feedstocks

We will add an additional optional configuration file to the recipe feedstock called `secrets.yaml.gpg`.
The unencrypted version of the file might look something like this
```yaml
username: Foo
password: Bar
```

Pangeo Forge will create a GPG key pair and publish the public key under the idenity `secrets@pangeo-forge.org`.
Users will [encrypt](https://www.gnupg.org/gph/en/manual/x110.html) the secrets file using gpg on the command line as follows
```bash
gpg --output secrets.yaml.gpg --encrypt --recipient secrets@pangeo-forge.org secrets.yaml
```
and add this file to their feedstock

The CI environment will have access to the private key and will be able to decrypt the file and read the secrets.
`pangeo_forge_prefect` will inject any secrets found in the file into the flow runtime using the [Prefect Client Secrets API](https://docs.prefect.io/orchestration/concepts/secrets.html#prefect-client).

When defining the recipe, users will no longer specify plain-text passwords, instead they will use a new `pangeo_forge_recipes.Secret` type.

```python
pattern = FilePattern(
    ...,
    fsspec_open_kwargs={"auth": aiohttp.BasicAuth(Secret['username'], Secret['password'])},
)
```

The `Secret` type will subclass `str` but will load its values from either environment variables or Prefect Secrets.

If the user is not willing to trust the Pangeo Forge github org with their encrypted secrets, they can run their own bakery and set their secrets locally.

## Consequences

- Recipes will no longer store plain-text passwords.
- We will have to create and publish these key pairs.
- We will have to test whether we can correctly pass these secrets through to the bakeries.
