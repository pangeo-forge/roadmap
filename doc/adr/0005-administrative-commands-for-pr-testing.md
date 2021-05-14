# Administrative commands for managing PR integration testing.

## Status
Proposed

## Context

Pangeo Forge is inspired by [Conda Forge](https://conda-forge.org/).  Conda Forge uses continuous integration to react to Github events in order to build and publish new packages.  The initial vision for PR flow within the `staged-recipes` repo was that submission of a PR would run normal linting and code verification procedures but would also register a reduced dimension version of the recipe as a flow with Prefect cloud and the appropriate labeling for execution by the development bakery.  This is problematic as Github workflows initiated by an event from a fork cannot access the repository or organization level secrets necessary to correctly register the flow with Prefect Cloud. See [here](https://docs.github.com/en/actions/reference/encrypted-secrets#using-encrypted-secrets-in-a-workflow) and [here](https://github.com/pangeo-forge/staged-recipes/pull/28#issuecomment-831469876) for further details.

## Decision

To alleviate this limitation we will include some administrator intervention in the PR review process by using [slash command dispatch](https://github.com/peter-evans/slash-command-dispatch) to allow administrators to comment on a PR in order to register and run a reduced dimension version of the recipe with the development bakery.  So the current workflow will be 

1. Recipe author forks `staged-recipes` and develops a new recipe with a `meta.yaml` entrypoint.
2. Recipe author submits a PR from their fork to `staged-recipes`.
3. This PR event triggers a workflow with initial linting and code validation that does not require secrets.
4. A `pangeo-forge` admin adds a comment command to register and run the reduced dimension version of the recipe on the development bakery.
5. If successful the bakery operator whose bakery id is referenced in `meta.yaml` should review the PR prior to merge into `master`.
6. The PR is merged into `master` which triggers a workflow to create a new `feedstock` repository for the recipe.

There are a few outstanding points here. There has been some [discussion](https://github.com/pangeo-forge/pangeo-forge-recipes/issues/97) about how to expose reduced recipe dimensions but we'll need to finalize an approach for this for repeatable recipe integration testing.  This discussion only outlines the workflows within the `staged-recipes` repo.  There will be an additonal ADR describing the detailed workflows in the `feedstock` repos to follow.

## Consequences

This will require adminstrator intervention in the PR review process will also add complexity for recipe authors who will need to incorporate reduced testing dimensions into their recipes.
