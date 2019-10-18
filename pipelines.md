# Pipelines

Pipelines are the heart and soul of the DrakeSpec and the Drake DSL. A
`Pipeline` object is a directed graph of inter-dependent [`Job`](jobs.md)
objects (referenced by name) which are executed as a means to some end. For
instance, a `Pipeline` might be used to execute a suite of quality checks (e.g.
unit tests, integration tests, lint checks) on a software project or to effect
release, distribution, or deployment of such a project. These are common uses,
but `Pipeline`s are open-ended in terms of what they might accomplish.

## `Pipeline` Fields

This section describes the fields of a `Pipeline` object.

### Implied name field

A `Pipeline` object has no intrinsic `name` field nor any other intrinsic
identifier, but does have an extrinsic name that is implied by its key in the
enclosing `Drakefile.yaml`'s `pipelines` map.

### `triggers`

__Field name:__ `triggers`<br/>
__Field type:__ `[]`[`PipelineTrigger`](pipeline-triggers.md)<br/>
__Required:__ N<br/>

The `triggers` field of a `Pipeline` object MAY contain an ordered list of
`PipelineTrigger` objects. Such a list enumerates the conditions (if any) under
which DrakeSpec-compliant pipeline executors MAY consider the enclosing
`Pipeline` to be eligible for execution.

The DrakeSpec does not define any specific triggers that compliant executors are
obligated to support; opting instead to treat triggers as extension points in
the spec to grant providers of DrakeSpec-compliant executors the flexibility to
pick and choose the triggers they will support. `PipelineTrigger` objects are
therefore _references_ to third-party trigger specifications along with
corresponding configuration.

Note that pipeline executors MUST document supported triggers to claim Drakespec
compliance.

When evaluating pipeline triggers to determine a pipeline's eligibility for
execution (for instance, in response to a given event), DrakeSpec-compliant
pipeline executors, MUST evaluate supported triggers in the order they are
defined in the `triggers` field. The _first_ trigger that is both supported and
whose conditions are met, if such a trigger is defined, MUST short-circuit
further trigger evaluation.

It is valid for pipeline producers to omit the `triggers` field from a
`Pipeline` object. In such cases, the conditions for executing such a pipeline
are undefined. This does not inherently render such a pipeline entirely
inoperative. DrakeSpec-compliant pipeline executors MAY trigger any pipeline in
response to arbitrary implementation-defined events. For instance,
[__DevDrake__](https://github.com/lovethedrake/devdrake) supports no triggers,
but _does_ execute any pipeline upon explicit request by a user.

Triggers are such a critical concept within the DrakeSpec that `PipelineTrigger`
objects are discussed in-depth in their own [section](pipeline-triggers.md).

### `jobs`

__Field name:__ `jobs`<br/>
__Field type:__ `[]`[`PipelineJob`](#pipelinejob-fields.md)<br/>
__Required:__ N<br/>

The `jobs` field of a `Pipeline` object MAY contain an ordered list of
`PipelineJob` objects. `PipelineJob` objects reference a [`Job`](jobs.md) object
by name and may optionally enumerate the job's dependencies on other jobs.

Note that `Pipeline`s are, conceptually, _directed graphs_ of inter-dependent
`Job`s, but because directed graphs are difficult to represent textually, the
DrakeSpec authors have opted to use an ordered list for the `jobs` field since
order implies a notion of "flow" through a pipeline. This notion is further
aided by a requirement that all jobs with dependencies are preceded by (are
"downstream from") the jobs upon which they depend. (You can read more about
this in the documentation for the [`dependencies`](#dependencies) field of the
`PipelineJob` type.) _However_, it should be noted that DrakeSpec-compliant
pipeline executors MAY actually execute jobs in any order and MAY execute
multiple jobs concurrently so long as no job is ever executed prior to its
dependencies.

## `PipelineJob` Fields

This section describes the fields of a `PipelineJob` object.

### `name`

__Field name:__ `name`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>

The value of the `name` field of a `PipelineJob` object MUST reference a defined
[`Job`](jobs.md) object by name (i.e. by a key from the top-level `jobs` map).
Any value for the `name` field is invalid if it does not reference a defined
`Job` object.

### `dependencies`

__Field name:__ `dependencies`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>

The `dependencies` field of a `PipelineJob` object is an unordered list of jobs
that a DrakeSpec-compliant job executor MUST execute both successfully and to
completion as prerequisites for execution of the job referenced by the `name`
field.

Note that all values in the `dependencies` field MUST reference a defined
[`Job`](jobs.md) object by name (i.e. by a key from the top-level `jobs` map)
and MUST only reference jobs that precede them in the pipeline (i.e. jobs
preceding them or "upstream" in the enclosing `Pipeline`'s `jobs` list). This
requirement enforces a sensible ordering of jobs such that no job ever precedes
its own dependencies. This requirement also, inherently, prevents the
possibility of defining a dependency cycle.

## General Runtime Requirements for Pipeline Execution

This section captures the general requirements DrakeSpec-compliant pipeline
executors must meet, with special emphasis on requirements not covered well by
the preceding discussion of pipeline-related types.

This section is a logical starting point for any provider wishing to implement
a Drakespec-compliant pipeline executor.

### Pipeline Triggers

DrakeSpec-compliant pipeline executors MAY support any set of pipeline triggers
they choose. Such triggers SHOULD be based on open specifications.

DrakeSpec-compliant pipeline executors MUST be event-driven and MUST utilize the
details of any given event in compliance with applicable third-party trigger
specifications to identify a set of pipelines that may be eligible for
execution. For example, a DrakeSpec-compliant pipeline executor that receives
"check suite requested" webhook from GitHub may utilize details from such an
event payload in compliance with an applicable third-party trigger specification
to identify a particular source code repository. If that source code repository,
for instance, contains a `Drakefile.yaml`, the `pipelines` field in that
document identifies a set of pipelines that may be eligible for execution.

In further assessing the eligibility of each pipeline for execution,
DrakeSpec-compliant pipeline executors MUST evaluate the triggers for each
pipeline in the order they are defined by the `triggers` field. In doing do,
DrakeSpec-compliant pipeline executors MUST immediately bypass any unsupported
trigger and any trigger not applicable for the event type in question. (For
instance, receipt of a "check suite requested" webhook from GitHub cannot
reasonably be expected to _ever_ meet the criteria set forth by a trigger that
responds to events from BitBucket. Consideration of such a trigger should
therefore be bypassed.) Upon identification of any trigger that may be
applicable, DrakeSpec-compliant pipeline executors MUST evaluate the details of
the inciting event against the configuration of the trigger in compliance with
the applicable third-party trigger specification. The first trigger whose
criteria are met MUST identify the enclosing pipeline as eligible for execution,
at which point further trigger evaluation for the enclosing pipeline MUST be
short-circuited.

It is a valid scenario for an event to ultimately result in the identification
of no pipelines for execution. It is an equally valid scenario for a single
event to ultimately result in _multiple_ pipelines being identified for
execution.

When multiple pipelines are identified for execution, DrakeSpec-compliant
pipeline executors MUST execute those pipelines identified in full isolation
from one another. i.e. Execution of one MUST NOT affect the execution of any
other in any way.

### Shared Storage

For each pipeline identified for execution, Drakespec-compliant pipeline
executors MUST provide shared storage wherein artifacts produced by any given
job within the pipeline may be relayed to downstream jobs within the same
pipeline.

If applicable per the applicable third-party trigger specification, project
source code MAY be written to shared storage. It is useful to examine two very
different implementations of this requirement for the sake of example.

[__DevDrake__](https://github.com/lovethedrake/devdrake) satifies the shared
storage requirement by mounting the local working directory into every job's
container. While this particular implementation has the potential to interfere
with the requirement that multiple pipelines executing concurrently must do so
in isolation, DevDrake avoids this particular difficulty by never executing
multiple pipelines concurrently.

[__BrigDrake__](https://github.com/lovethedrake/devdrake) satisfies the shared
storage requirement differently-- by provisioning network attached storage at
the onset of every pipeline's execution. If applicable, source code is cloned
to the shared volume and the shared volume is mounted into every job's
container.

__The DrakeSpec authors readily acknowledge some difficulties with this section
of the specification.__

__For one, the example implementations cited above have significant
shortcomings. Network attached storage, for instance, is notoriously slow and
likely to be a limiting factor for many implementations of DrakeSpec-compliant
pipeline executors.  Mounting a local directory into containers "contaminates"
the local working directory, potentially tainting the results of _future_
pipeline executions, which _should_ be completely isolated from prior pipeline
executions.__

__Shortcomings of the example implementations aside, the authors further
acknowledge the shared storage requirement, as currently written, to be
inadequate to support many potential improvements to the specification that are
already under consideration. For instance, the shared storage requirement, as
currently written, would not facilitate reliable re-execution of a failed job or
resumption of a failed pipeline because failed jobs would be re-executed with
state contaminated by its own prior execution and (potentially) prior execution
of other jobs in the pipeline. Reliable re-execution of a failed job
realistically requires something like a layered file system to permit shared
storage to be returned to the state it was in prior to the first (failed)
execution of the job.__

__In acknowledging the shortcomings of the referenced implementations of the
shared storage requirement, and more specifically, the limitations of the
requirement itself, as currently written, the DrakeSpec authors wish to urge
caution as this section of the spec is likely to evolve signigicantly in
subsequent releases.__

### Order of Job Execution

The `jobs` field of a pipeline describes a _directed graph_ of inter-dependent
jobs. (See the [jobs](#jobs) section for further details.)

DrakeSpec-compliant pipeline executors MUST never execute a job prior to
successful completion of all jobs on which it depends. As long as this
requirements is satisfied, they MAY execute jobs in any order and MAY execute
any number of jobs concurrently.

### Handling Job Failure

If execution of a job within a pipeline fails or times out, DrakeSpec-compliant pipeline executors MUST permit execution of any concurrently executing jobs to
continue until completion or timeout. All pending jobs MUST immediately be
canceled, even if the failed job(s) were not among their dependencies.

### Re-Executing Jobs / Resuming Failed Pipelines

This current draft of the specification requires that DrakeSpec-compliant
pipeline executors MUST NOT permit re-execution of a failed job and MUST NOT
offer functionality for "resuming" execution of a pipeline that has failed due
to job failure or timeout. Pipelines MUST only be executed as atomic units.

__This requirement is a direct consequence of current limitations in the [shared
storage requirement](#shared-storage). The DrakeSpec authors wish to urge
caution as this section of the spec is likely to evolve signigicantly in
subsequent releases.__
