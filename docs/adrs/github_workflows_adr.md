# Using Github Workflows/Actions for Recipe orchestration

## Status

Accepted

## Context

Pangeo Forge is inspired by [Conda Forge](https://conda-forge.org/).  Conda Forge uses continuous integration to react to Github events in order to build and publish new packages.  Building on this approach, Pangeo Forge will use Github Workflows to communicate with [Prefect Server/Cloud](https://docs.prefect.io/orchestration/) to register and run [Prefect Flows](https://docs.prefect.io/orchestration/concepts/flows.html) on cloud based infrastructure separately managed by partner organizations. 

![diagram](./pangeo_forge_arch.png) 

## Decision

Pangeo Forge's goal of managing scientific data creation and transformation in an open collaborative environment requires orchestration tooling which is simple and accessible to everyone.  In previous architecture discussions it was decided that almost all of the steps of reviewing and publishing Recipes would be managed with Github Workflows.  There was some remaining uncertainty about if actual Recipe execution in a partner controlled bakery (which could result in large cloud resource expenditures) should be managed directly with Github Workflows. The current consensus decision is now to use Github release and PR merge events to trigger Recipe execution in registered bakeries.

## Consequences

This drastically simplifies Prefect Cloud account management since we will only need to manage a single Prefect Tenant [token](https://docs.prefect.io/orchestration/concepts/tokens.html#token-types) which can be shared by Feedstock repos.  

The biggest concern is how to appropriately manage security for [Prefect Flow Labels](https://docs.prefect.io/orchestration/agents/overview.html#labels).  Since bakeries will be running [Prefect Agents](https://docs.prefect.io/orchestration/agents/overview.html#overview) and will only run flows with the relevant Labels.  Most likely, there will need to be some communication with bakery infrastructure managers and feedstock repository administrators to merge pull requests with the appropriate Flow Labels when bakeries want to execute recipes releases.
