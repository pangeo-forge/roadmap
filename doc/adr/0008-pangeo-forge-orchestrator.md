# Pangeo Forge Orchestrator

## Status

Proposed

## Context

For some time it has been clear that the many disparate components of Pangeo Forge are hard to keep track of and there is currently no single entry point from which all components of the system are orchestrated. This challenge is felt on an organization level:

- By users looking to experiment with the platform, but unsure how to invoke the various modular components;
- By developers, who experience confusion regarding where to locate existing features (or add new ones).

In addition to organizational aspect, the implementation of the platform is currently bifurcated between execution steps within the Python recipes and integration steps within GitHub Actions workflows and/or standalone repositories.

## Decision

We should provide a single entry point from which all aspects of Pangeo Forge can be introspected and run. This entry point should be a `pip`- and `conda`-installable Python package.

### `pangeo-forge-orchestrator` repository

We will create and maintain a new repository within the Pangeo Forge organization, called `pangeo-forge-orchestrator`, which:

1. Contains integration and orchestration scripts for creating, validating, running, and cataloging recipes, feedstocks, and catalog objects
2. Exposes a command line interface for introspecting and executing these scripts

Some of these scripts will be new and others will be adapted from existing GitHub Actions, which themselves can be refactored to call Python scripts from `pangeo-forge-orchestrator`.

### Package and entrypoint name

For installation and command line invocation purposes, the user-facing name of `pangeo-forge-orchestrator` will be, simply, `pangeo-forge`.

The aim of differentiating the repository name from the package name is to make the repository's functionality self-evident to developers, while exposing that functionality as a more generic name to reduce friction in user land.

## Commands

The CLI (and functionality it wraps) will include both top-level functions as well as sub-modules.

Examples of top-level functions include:

- `create-feedstock`

Examples of submodules and associated functions include:

- `recipe`
    - `lint`
    - `test`
    - `catalog`

- `bakeries`
    - `list`

- `catalog`
    - `create`
    - `browse`


## Consequences

Following approximately the contribution steps outlined in [flow-charts/ci-flow-with-callstack.png](https://raw.githubusercontent.com/pangeo-forge/flow-charts/main/renders/ci-flow-with-callstack.png), the following code would be added to, refactored as a result of, and/or subsumed by `pangeo-forge-orchestrator`:

| Step | Implementation |
| ---- | -------------- |
| Local recipe execution (from CLI, w/out Jupyter) | _New_ |
| Lint recipe PRs | _New_ |
| Interactive feedback based on lint | _New_ |
| Dispatch recipe test | _Refactor_: https://github.com/pangeo-forge/staged-recipes/blob/master/.github/workflows/run-recipe-test.yaml |
| Setup recipe test | _Subsumes_: https://github.com/pangeo-forge/recipe-prefect-action |
| Run Prefect Flow (test or full) | _Subsumes_: https://github.com/pangeo-forge/pangeo-forge-prefect |
| Interactive feedback on recipe test | _New_ |
| Dispatch feedstock creation | _Refactor_: https://github.com/pangeo-forge/staged-recipes/blob/master/.github/workflows/create-feedstock.yaml |
| Create feedstock repo | _Subsumes_: https://github.com/pangeo-forge/feedstock-creation-action |
| Run full recipe | _Refactor_: https://github.com/pangeo-forge/feedstock-template/blob/main/.github/workflows/run-recipe.yaml |
| Generate STAC Item | _New_ |
