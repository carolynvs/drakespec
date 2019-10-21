# Pipeline Triggers

Pipeline triggers (alternatively "triggers", for brevity), reference discrete
events that prompt execution of a [`Pipeline`](pipelines.md). The DrakeSpec
defines triggers, generically, as a concept, but deliberately avoids enumerating
any specific triggers that compliant pipeline executors are obligated to
support. In this manner, triggers are, conceptually, an extension point within
the specification, allowing for the emergence of third-party triggers based on
their own (ideally open) specifications. Providers of DrakeSpec-compliant
pipeline executors are free to pick and choose which triggers they will support.

A `PipelineTrigger` object represents a reference to a third-party trigger
specification combined with any applicable configuration.

## `PipelineTrigger` Fields

This section describes the fields of a `PipelineTrigger` object, including valid
use of each field by pipeline producers and correct interpretation and use of
each field by DrakeSpec-compliant pipeline executors and other consumers.

### `specUri`

__Field name:__ `specUri`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>

Pipeline producers MUST populate the `specUri` field of a `PipelineTrigger`
object with a value that references a third-party specification, with the value
used having been prescribed by that same specification. In general, this will be
a globally unique value and the authors of the specification in question will
ideally have assured that by prescribing a `specUri` value that is a URL that is
solely under the authors' control.

If the `specUri` field of a `PipelineTrigger` object is left undefined,
DrakeSpec-compliant pipeline consumers MUST reject it during parsing or
validation and MUST report that rejection through any implementation-defined
means, which could include, for example, output or system logs.

If the `specUri` field of a `PipelineTrigger` references an _unsupported_
third-party specification, DrakeSpec-compliant pipeline executors MUST ignore
the `PipelineTrigger`, but SHOULD issue a warning to that effect through any
implementation-defined means, which could include, for example, output or system
logs.

If the `specUri` field of a `PipelineTrigger` references a _supported_
third-party specification, DrakeSpec-compliant pipeline executors MUST utilize
the `specVersion` field to further determine whether the specific _version_
of the third-party specification is supported.

### `specVersion`

__Field name:__ `specVersion`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>
__Allowed values:__ `vX.Y.Z`

Pipeline producers MUST populate the `specVersion` field of a `PipelineTrigger`
object with a value that claims compliance with a specific version of the
third-party specification referenced by the `specUri` field.

If the `specVersion` field of a `PipelineTrigger` object is left undefined,
DrakeSpec-compliant pipeline consumers MUST reject it during parsing or
validation and MUST report that rejection through any implementation-defined
means, which could include, for example, output or system logs.

If the `specUri` field of a `PipelineTrigger` references an _unsupported_
third-party specification, the value of the `specVersion` field is effectively
irrelevant and DrakeSpec-compliant pipeline executors MUST ignore the
`PipelineTrigger`.

If the `specUri` field of a `PipelineTrigger` references a _supported_
third-party specification and the `specVersion` field contains a value claiming
compliance with an older or newer version of that third-party specification than
a DrakeSpec-compliant pipeline executor is able to support, that executor MUST
ignore the `PipelineTrigger`, but SHOULD issue a warning to that effect through
any implementation-defined means, which could include, for example, output or
system logs.

If the `specUri` field of a `PipelineTrigger` references a _supported_
third-party specification and the `specVersion` field contains a value claiming
compliance with a supported version of that third-party specification, a
DrakeSpec compliant pipeline executor MAY utilize the contents of the
`PipelineTrigger` object's `config` field in compliance with the referenced
third-party specification to make a runtime determination of whether the
enclosing `Pipeline` is eligible for execution in response to a given event.

### `config`

__Field name:__ `config`<br/>
__Field type:__ Flexible<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the `config` field of a `PipelineTrigger` object
with any valid JSON that complies with the thrid-party specification, and
specific revision thereof, referenced by the `specUri` and `specVersion` fields
of the same `PipelineTrigger` object.

If the `specUri` and `specVersion` fields of a `PipelineTrigger`, combined,
reference an _unsupported_ third-party specification (or unsupported revision
thereof), the value of the `config` field is effectively irrelevant and
DrakeSpec-compliant pipeline executors MUST ignore the `PipelineTrigger`.

If the `specUri` and `specVersion` fields of a `PipelineTrigger`, combined,
reference a _supported_ third-party specification, a DrakeSpec compliant
pipeline executor MAY utilize the contents of the `PipelineTrigger` object's
`config` field in compliance with the referenced third-party specification to
make a runtime determination of whether the enclosing `Pipeline` is eligible for
execution in response to a given event.

__Note the contents of this field are _entirely opaque_ from the perspective of
the DrakeSpec. Third-party trigger specificatons prescribe how
DrakeSpec-compliant pipeline executors that support them may derive semantically
meaningful configuration from the contents of this field.__

## General Requirements for Third-Party Trigger Specifications

This section documents the general requirements that a third-party trigger
specification MUST adhere to in order to, itself, be compatible with the
DrakeSpec and with DrakeSpec-compliant pipeline executors that may additionally
opt to support such a third-party trigger specification.

### Specification URI

Third-party trigger specifications MUST clearly document a globally unique URI
that:

1. Permits pipeline producers to unambiguously reference said specification.
   (See documentation for the [`specUri`](#specuri) field of the
   `PipelineTrigger` object.)

1. Permits DrakeSpec-compliant pipeline executors to dynamically load and
   execute logic that implements support for said specification.

To ensure global uniquenes, the URI MUST be a URL that is solely under the
control of the third-party trigger specification's authors. The URL MUST NOT be
prefixed with any protocol.

### Semantic Versioning

Third-party trigger specifications MUST follow semantic versioning practices, as
DrakeSpec-compliant pipeline executors MUST assume such specifications do so
when determining whether a specific revision of such a specification is
supported.

### Openness

Third-party trigger specifications SHOULD be _open_ specifications.

### Schema

Third-party trigger specifications MUST clearly document their configuration
schema.

### Events

Third-party trigger specifications MUST clearly document the events that
supporting DrakeSpec-compliant pipeline executors MUST support and how, if
applicable, DrakeSpec-compliant pipeline executors MUST utilize trigger-specific
configuration in assessing a pipeline's eligibility for execution in response to
a given event.

To illustrate the above requirement, consider a hypothetical third-party trigger
specification that descibes support for GitHub. Such a specification might
outline which types of webhooks supporting pipeline executor must listen for,
and for each, how details from that webbook's payload, such as branch or tag
names, may be compared to trigger-specific configuration to further determine
whether a pipeline should be executed in response to that webhook.

### Job & Pipeline Execution Hooks

Third-party trigger specifications MAY impose requirements on
DrakeSpec-compliant pipeline executors to implement additional, trigger-specific
behaviors before, during, or after execution of jobs or entire pipelines.
