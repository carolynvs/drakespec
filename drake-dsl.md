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

This section describes valid use of the top-level fields of a `Drakefile.yaml`.

Pipeline producers MUST utilize fields only as prescribed in order to define
DrakeSpec-compliant pipelines. The Drake DSL reserves all undocumented fields
for future use. As such, pipeline producers MUST NOT utilize any
as-yet-undefined fields.

DrakeSpec-compliant pipeline consumers MUST interpret fields only as prescribed
and MUST proactively reject all invalid `Drakefile.yaml` during parsing or
validation. DrakeSpec-compliant pipeline executors MUST NOT execute any actions
prescribed by jobs and/or pipelines defined within an invalid `Drakefile.yaml`.

### uri

__Field name:__ `uri`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>
__Allowed values:__ `github.com/lovethedrake/drakespec`<br/>

Because it is an open specification, the DrakeSpec's authors acknowledge that it
could, realistically, be forked in the future. In anticipation of that
possibility, pipeline producers MUST utilize the top-level `uri` field with a
value of `github.com/lovethedrake/drakespec` to explicitly denote compliance
with the DrakeSpec (as opposed to compliance with one of its forks).

If a `Drakefile.yaml` omits the `uri` field or utilizes it with a value other
than `github.com/lovethedrake/drakespec`, DrakeSpec-compliant pipeline consumers
MUST proactively reject it during parsing or validation. DrakeSpec-compliant
pipeline executors MUST NOT execute any actions prescribed by jobs and/or
pipelines defined within such a `Drakefile.yaml`.

### version

__Field name:__ `version`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>
__Allowed values:__ `vX.Y.Z`

Because the DrakeSpec authors anticipate the specification evolving over time,
with a succession of semantically versioned releases, pipeline producers MUST
utilize the top-level `version` field to explicitly denote compliance with a
given version of the DrakeSpec.

If a `Drakefile.yaml` omits the `version` field or utilizes it with a value
denoting either an older or newer version of the specification than a
DrakeSpec-compliant pipeline consumer is able to support, that consumer MUST
proactively reject it during parsing or validation. DrakeSpec-compliant pipeline
executors MUST NOT execute any actions prescribed by jobs and/or pipelines
defined within such a `Drakefile.yaml`.

Note that DrakeSpec-compliant pipeline consumers MUST accept a
`Drakefile.yaml`'s claim of being compliant with a given version of the
DrakeSpec at face value. This is to say, if a `Drakefile.yaml` indicated
compliance with a newer version, unsupported by the consumer, but incidentally
did not leverage any newer Drake DSL features unsupported by the consumer, the
consumer MUST still reject such a `Drakefile.yaml` as unsupported.
DrakeSpec-compliant pipeline executors MUST NOT make a "best effort" to execute
the pipelines defined within such a `Drakefile.yaml`.

### jobs

__Field name:__ `jobs`<br/>
__Field type:__ `map[string]`[`Job`](jobs.md)<br/>
__Required:__ N<br/>

The top-level `jobs` field MAY contain a map of `Job` objects keyed by name. By
definition, a `Drakefile.yaml` with no `jobs` is a _trivial_ `Drakefile.yaml`.

Jobs are such a critical concept within the DrakeSpec that `Job` objects are
discussed in-depth in their own [section](jobs.md).

### pipelines

__Field name:__ `pipelines`<br/>
__Field type:__ `map[string]`[`Pipeline`](pipelines.md)<br/>
__Required:__ N<br/>

The top-level `pipelines` field MAY contain a map of `Pipelines` objects keyed
by name.

Pipelines are such a critical concept within the DrakeSpec that `Pipeline`
objects are discussed in-depth in their own [section](pipelines.md).
