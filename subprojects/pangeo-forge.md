# Pangeo Forge Roadmap

[Pangeo-forge](https://github.com/pangeo-forge/pangeo-forge) provides a central workflow manager and API for the productions of cloud-optimized datasets. It is being desinged to include a high-level Pipeline API (built on top of [Prefect](https://www.prefect.io/)) that will be useful inside and outside of pangeo-forge infrastructure.

Pangeo-forge is still a work-in-progress, but when complete, it will:

- Include a low-level API for building complexe data transformation workflows
- Include a high-level `Pipeline` API for executing data transformation workflows

## API design

The pangeo-forge API should be simple, extensible, and highly scalable.
To this end, we define five main concepts that make up the pangeo-forge API: the Sources, Targets, Tasks, Flows, and Pipelines.

```python
import prefect
import pangeo_forge


class Pipeline(pangeo_forge.AbstractPipeline):

    def __init__(self, ...) -> None:
        # ...
    
    @property
    def sources(self) -> List[str]:
        return [...]
    
    @property
    def targets(self) -> List[str]:
        return [...]
    
    @property
    def flow(self) -> prefect.Flow:
        with prefect.Flow as flow:
            # ... apply one or more prefect.Tasks
        return flow
```

### Sources

Sources is a list of URLs that define the source location of the dataset. 
These URLs may point to any publically web-accessable location and use a variety of access protocols.

### Targets

Targets is a list of URLs that define the target location of the dataset(s).
These URLs will most commonly point to a cloud bucket.

### Tasks

A `Task` represents a disrete action in a Prefect workflow.
Pangeo-forge will include a collection of ready-to-use Prefect Tasks for common opperations.
Custom tasks can be defined using Prefect's task api.

See [`pangeo_forge.tasks`](https://github.com/pangeo-forge/pangeo-forge/tree/master/pangeo_forge/tasks) for initial examples of pangeo-forge Prefect Tasks.

### Flow

A Prefect Flow is a container for Prefect Tasks.
Pangeo-forge will include a collection of ready-to-use Prefect Flows for common pipeline patterns.
We currently expect to expose these Flow's as mixin classes that set the `flow` property.

### Pipeline

The Pipeline provides the high-level interface to the pangeo-forge API.