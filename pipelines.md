# Pipelines

Pipelines are the heart and soul of the DrakeSpec and the Drake DSL. A
`Pipeline` object is a directed graph of inter-dependent [`Job`](jobs.md)
objects (referenced by name) which are executed as a means to some end. For
instance, a `Pipeline` might be used to execute a suite of quality checks (e.g.
unit tests, integration tests, lint checks) on a software project or to effect
release, distribution, or deployment of such a project. These are common uses,
but `Pipeline`s are open-ended in terms of what they might accomplish.

## `Pipeline` Fields

This section describes the fields of a `Pipeline` object, including valid use of
each field by pipeline producers and correct interpretation and use of each
field by DrakeSpec-compliant pipeline executors and other consumers.

### Implied name field

A `Pipeline` object has no intrinsic `name` field nor any other intrinsic
identifier, but does have an extrinsic name that is implied by its key in the
enclosing `Drakefile.yaml`'s `pipelines` map.

### `triggers`

__Field name:__ `triggers`<br/>
__Field type:__ `[]`[`PipelineTrigger`](pipeline-triggers.md)<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the `triggers` field of a `Pipeline` object with
an ordered list of `PipelineTrigger` objects to enumerate the conditions under
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
but _does_ execute any pipeline when explicitly requested by name by a user.

Triggers are such a critical concept within the DrakeSpec that `PipelineTrigger`
objects are discussed in-depth in their own [section](pipeline-triggers.md).

### `jobs`

__Field name:__ `jobs`<br/>
__Field type:__ `[]`[`PipelineJob`](#pipelinejob-fields.md)<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the `jobs` field of a `Pipeline` object with an
ordered list of `PipelineJob` objects. `PipelineJob` objects reference a
[`Job`](jobs.md) object from the enclosing `Drakefile.yaml`'s `jobs` map by name
and may optionally enumerate dependencies on other `Jobs` (also referenced by
name). A `Pipeline` object with no values in the `jobs` field is considered
valid by this specification, but is, by definition, a trivial `Pipeline` that
could serve no purpose.

Note that `Pipeline`s are, conceptually, _directed graphs_ of inter-dependent
`Job`s, but because directed graphs are difficult to represent textually, the
DrakeSpec authors have opted to use an ordered list for the `jobs` field since
order implies a directional "flow" through a `Pipeline`. This notion is further
aided by a requirement that all `PipelineJob`s with dependencies appear
"downstream" from the `PipelineJob`s upon which they depend. (You can read more
about this in the documentation for the [`dependencies`](#dependencies) field of
the `PipelineJob` type.)

DrakeSpec-compliant pipeline executors MUST attempt to execute all `Jobs`
referenced by the `jobs` field of a `PipelineJob` object until all `Job`s have
completed successfully or until any `Job` fails or times out.
DrakeSpec-compliant pipeline executors MAY execute the `Job`s referenced by the
`jobs` field of a `PipelineJob` object in any order and MAY execute multiple
`Job`s concurrently, but MUST NOT execute any `Job` before all `Job`s upon which
it depends (if any) have completed successfully. If any `Job` fails or times out
DrakeSpec-compliant pipeline executors MUST cancel all pending
(not-yet-executing) `Job`s, but MUST permit in-progress `Job`s to complete.

## `PipelineJob` Fields

This section describes the fields of a `PipelineJob` object, including valid use
of each field by pipeline producers and correct interpretation and use of each
field by DrakeSpec-compliant executors and other consumers.

### `name`

__Field name:__ `name`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>

Pipeline producers MUST populate the `name` field of a `PipelineJob` object to
reference a [`Job`](jobs.md) object from the enclosing `Drakefile.yaml`'s `jobs`
map by name. Any value for the `name` field is invalid if it does not reference
a defined `Job` object.

### `dependencies`

__Field name:__ `dependencies`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the `dependencies` field of a `PipelineJob`
object with an (implicitly unordered) list of references to [`Job`](jobs.md)
objects from the enclosing `Drakefile.yaml`'s `jobs` map in order to denote a
dependency on successful completion of those `Job`s. Any value in the
`dependencies` field is invalid if it does not reference a defined `Job` object.
Any value in the `dependencies` field is also invalid if it does not reference a
`Job` that is mutually referenced by a `PipelineJob` object that _precedes_ the
enclosing `PipelineJob` object in the enclosing `Pipeline`'s `jobs` collection.
This requirement enforces a sensible ordering of `PipelineJob`s such that none
ever precedes its own dependencies. This requirement also, implicitly, prevents
the possibility of defining a dependency cycle.

DrakeSpec-compliant pipeline executors MUST execute all `Job`s referenced by the
`dependencies` field of a `PipelineJob` both successfully and to completion
prior to execution of the `Job` referenced by the `name` field.

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

For each pipeline identified for execution, if any container of any job therein
indicates a dependency on shared storage (by providing a non-empty value for the
`sharedStorageMountPath` field), Drakespec-compliant pipeline executors MUST
provision a shared volume that MUST be mounted to the file system of all such
containers at the indicated path.

__Note: The DrakeSpec authors anticipate significant changes to the spec as it
relates to shared storage-- namely, they anticipate requiring "layered" storage
that enables re-execution of a failed job by returning to the state it was in
prior to initial execution of that job. Due to this, the authors urge caution as
this section of the specification is very likely to change.__

### Order of Job Execution

The `jobs` field of a pipeline describes a _directed graph_ of inter-dependent
jobs. (See the [jobs](#jobs) section for further details.)

DrakeSpec-compliant pipeline executors MUST never execute a job prior to
successful completion of all jobs on which it depends. As long as this
requirements is satisfied, they MAY execute jobs in any order and MAY execute
any number of jobs concurrently.

### Handling Job Failure

If execution of a job within a pipeline fails or times out, DrakeSpec-compliant
pipeline executors MUST permit execution of any concurrently executing jobs to
continue until completion or timeout. All pending jobs MUST immediately be
canceled, even if the failed job(s) were not among their dependencies.

### Re-Executing Jobs / Resuming Failed Pipelines

DrakeSpec-compliant pipeline executors MUST NOT permit re-execution of a failed
job and MUST NOT offer functionality for "resuming" execution of a pipeline that
has failed due to job failure or timeout. Pipelines MUST only be executed as
atomic units.

__This requirement is a direct consequence of the [shared storage
requirement](#shared-storage) not (yet) sufficiently addressing how the state of
the shared storage volume can be restored to the correct state prior to
re-execution of such a job. The DrakeSpec authors wish to urge caution as this
section of the spec is likely to evolve signigicantly in subsequent releases.__
