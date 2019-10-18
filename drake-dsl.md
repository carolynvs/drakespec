# The Drake DSL

The Drake DSL is a YAML-based domain-specific language for the declarative
definition of DrakeSpec-compliant pipelines.

## Drakefile.yaml

By convention, the Drake DSL is used to describe pipelines, the jobs from which
they are comprised, and other supporting metadata in a file named
`Drakefile.yaml`. By convention, `Drakefile.yaml` is stored at the root of a
software project's source code repository. The DrakeSpec does NOT require
DrakeSpec-compliant platforms or utilities to observe or enforce these
conventions.

By establishing the above as conventions, the DrakeSpec authors wish to
acknowledge what they imagine to be the most ubiquitous use of the DSL--
defining CI/CD pipelines "as code" alongside the code those pipelines operate
upon. By stopping short of requiring DrakeSpec-compliant platforms or utilities
to observe or enforce these conventions, providers of such software retain
significant latitude to support a broader and more imaginative range of use
cases as well as to innovate alternative "developer experiences." To illustrate,
it is easy to imagine uses for pipelines that are defined somewhere other than a
source code repository-- or pipelines that don't operate upon source code at
all.

In deference to conventions-- and for convenience-- instances of the Drake DSL
are referenced as `Drakefile.yaml` throughout the remainder of this
specification. Readers should understand this as shorthand for any valid use of
the Drake DSL.

## Top-Level Fields

This section describes the top-level fields of a `Drakefile.yaml`, including
valid use of each field by producers and correct interpretation and use of each
field by DrakeSpec-compliant executors and other consumers.

Producers MUST utilize fields only as prescribed in order to define
DrakeSpec-compliant jobs and/or pipelines. The Drake DSL reserves all
undocumented fields for future use. As such, producers MUST NOT utilize any
as-yet-undefined fields.

DrakeSpec-compliant consumers MUST interpret fields only as prescribed and MUST
reject all invalid and/or unsupported `Drakefile.yaml` during parsing or
validation and MUST report that rejection through any implementation-defined
means, which could include, for example, output or system logs.

DrakeSpec-compliant executors MUST NOT execute any jobs or pipelines defined
within an invalid or unsupported `Drakefile.yaml`.

### `specUri`

__Field name:__ `specUri`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>
__Allowed values:__ `github.com/lovethedrake/drakespec`<br/>

Producers MUST populate the top-level `specUri` field with the value
`github.com/lovethedrake/drakespec` to explicitly signal compliance with the
DrakeSpec (as opposed to a future, hypothetical fork of the specification).

If the `specUri` field is left undefined or contains a value other than
`github.com/lovethedrake/drakespec`, DrakeSpec-compliant consumers MUST reject
it during parsing or validation and MUST report that rejection through any
implementation-defined means, which could include, for example, output or system
logs.

### `specVersion`

__Field name:__ `specVersion`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>
__Allowed values:__ `vX.Y.Z`

Producers MUST populate the top-level `specVersion` field with a value that
claims compliance with a specific version of the DrakeSpec. Because the
DrakeSpec authors anticipate the specification evolving over time, with a
succession of semantically versioned releases, this requirement helps mitigate
the possibility of any DrakeSpec compliant consumer attempting to consume a
`Drakefile.yaml` that it cannot support.

If the `specVersion` field is left undefined or contains a value claiming
compliance with an older or newer version of the specification than a
DrakeSpec-compliant consumer is able to support, that consumer MUST reject it
during parsing or validation and MUST report that rejection through any
implementation-defined means, which could include, for example, output or system
logs.

### jobs

__Field name:__ `jobs`<br/>
__Field type:__ `map[string]`[`Job`](jobs.md)<br/>
__Required:__ N<br/>

Job producers MAY populate the top-level `jobs` field with a map of `Job`
objects keyed by name. A `Drakefile.yaml` object with no values in the `jobs`
field is currently considered valid by this specification, but is, by
definition, a trivial `Drakefile.yaml` that could serve no practical purpose.

Jobs are such a critical concept within the DrakeSpec that `Job` objects are
discussed in-depth in their own [section](jobs.md).

### pipelines

__Field name:__ `pipelines`<br/>
__Field type:__ `map[string]`[`Pipeline`](pipelines.md)<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the top-level `pipelines` field with a map of
`Pipelines` objects keyed by name. A `Drakefile.yaml` object with no values in
the `pipelines` field is considered valid by this specification since some some
DrakeSpec-compliant executors or consumers may be capable of executing or
otherwise consuming jobs independently of any pipeline.

Pipelines are such a critical concept within the DrakeSpec that `Pipeline`
objects are discussed in-depth in their own [section](pipelines.md).
