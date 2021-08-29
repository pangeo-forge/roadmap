# Use a REST API to provide a central source of truth for Pangeo Forge Federation

## Status

Draft

## Context

Pangeo Forge contains many moving parts. There are the recipe feedstocks and associated github actions, the bakeries, the target buckets, etc.
Eventually there will be a data catalog.
And then there is the website.
Ideally the website would organize all of this information and present it in a coherent, unified way.
Formalizing the relationships between the different components of Pangeo Forge through some sort of model would also help us developers build better.

Some, but not all of the "data" about Pangeo Forge lives in GitHub. For example:
- [ADR 2](https://github.com/pangeo-forge/roadmap/blob/master/doc/adr/0002-use-meta-yaml-to-track-feedstock-metadata.md) describes
  the format and contents of recipe feedstock "meta.yaml", which gives metadata about the recipe feedstock and the recipes it contains.
  [Here is an example](https://github.com/pangeo-forge/noaa-oisst-avhrr-only-feedstock/blob/main/feedstock/meta.yaml) of a live meta.yaml.
- [ARD 4](https://github.com/pangeo-forge/roadmap/blob/master/doc/adr/0004-use-yaml-file-for-bakery-database.md) describes
  the bakery database, which is just a [single yaml file](https://github.com/pangeo-forge/bakery-database/blob/main/bakeries.yaml) in GitHub.

In theory, web apps could read and parse these files directly.
However, there is information missing from those files that we would like to track.
Which recipes were run when?
Which recipes produced which datasets?
etc.

## Decision

I propose we deploy a standalone REST api at <https://api.pangeo-forge.org>.
This API will have public endpoints, for getting information, and authorized endpoints,
for various actors (e.g. GitHub workflows, bakeries, etc.) to post updates.
We will start as simple as possible and expand the API as needed.
All endpoints will return JSON.

### Examples

#### Bakery API

- `GET /bakeries` returns bakeries and their details
- `GET /bakeries/{bakery_id}` returns the details about a specific bakery
- `GET /bakeries/{bakery_id}/recipes_runs` returns all recipes run by that bakery

There are no POST functions because the bakery information is set in the GitHub repo.
The REST API would sync (and possibly cache) this information from GitHub

#### Feedstock API

- `GET /feedstocks` returns a (potentially paginated) list of feedstocks
- `GET /feedstocks/{feedstock_id}` returns the details about a specific feedstock
- `GET /feedstocks/{feedstock_id}/recipe_runs` returns all recipe runs associated with that feedstock

There are no POST functions because the bakery information is set in the GitHub repo.
The REST API would sync (and possibly cache) this information from GitHub

#### ~Recipes~ "Recipe Runs" API

Just writing this down has forced me to sharpen my mental model for what are the durable objects in Pangeo Forge.
Is a "Recipe" such an object?
At first this seemed like an obvious "yes".
But I'm no longer sure.
For example, `meta.yaml` does not necessarily explicitly enumerate that feedstock's recipes since the recipes might be
encoded in a [python dict](https://github.com/pangeo-forge/roadmap/blob/master/doc/adr/0002-use-meta-yaml-to-track-feedstock-metadata.md#recipes-section).
Furthermore, the feedstock could change its recipes, or their names, or both.
There can be multiple recipes per feedstock.

Instead, the thing worth keeping track of to me seems like a "recipe run". A "recipe run" schema in a table might look like this:

| name | description |
| -- | -- |
| `id` | Primary key for the recipe run itself |
| `recipe_id` | What was the ID of this recipe, as specified in [ADR 2](https://github.com/pangeo-forge/roadmap/blob/master/doc/adr/0002-use-meta-yaml-to-track-feedstock-metadata.md#recipes-section) |
| `run_date` | When was the recipe run |
| `bakery_id` | Which bakery ran the run |
| `feedstock_id` | Where did the recipe come from |
| `commit` | Which commit of the feedstock generated the recipe |
| `status` | Success / Failure / etc. |
| `message` | Anything else the bakery wants to add about the run |

The information about recipe runs is NOT in GitHub.
It could be populated by the bakeries as they process recipes. For example

- `POST /recipe_runs` log a new recipe run

Then everyone else could know what the Bakeries are up to via

- `GET /recipe_runs`

And similar methods. We could also implement queries, etc.

#### Datasets API

We could also expose the datasets via the API.
This of course overlaps with STAC stuff / catalogging stuff.
It would be really cool to be able to track `dataset` ➡️ `recipe_run` ➡️ `feedstock`

More thought is required here. We wouldn't have to implement this right away.

## Consequences

The various parts of Pangeo Forge could use this API to communicate their status with each other.

The Vue website could could query this API to populate itself.

We could even develop a CLI that would talk to the API. This could simplify some of our existing CI and automation.

The downside of course is that it is another thing to build and maintain.

