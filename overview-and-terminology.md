# Overview and Terminology

This section presents a high-level overview of the DrakeSpec. It is a reasonable
starting point for developers wishing to implement DrakeSpec-compliant platforms
or utilities as well as developers wishing to declaratively define
DrakeSpec-compliant pipelines.

## Glossary of Terms

This section defines key terms that are used throughout the remainder of the
specification.

Note that terms in this section are not organized alphabetically. Rather, they
are introduced in an order the authors consider sensible for readers previously
unfamiliar with this specification.

### Drake

The unqualified term __Drake__ is too easily mistaken as a reference to a
particular [__DrakeSpec-compliant__](#drakespec-compliant)
[__utility__](#utilities) properly known as
[__DevDrake__](https://github.com/lovethedrake/devdrake). The confusion arises
because the tool's executable command is tersely abbreviated as `drake` to
appease lazy typers (including the DrakeSpec's authors).

DevDrake is not discussed at any length within this specification. It is, at
times, metioned only when convenient for the sake of example.

This clarification is offered here only to _discourage_ readers, users,
contributors, etc. from incorrectly using the unqualified term "Drake" in
reference to the specification-- which is properly referred to as [__the
DrakeSpec__](#drakespec).

### DrakeSpec

The __DrakeSpec__ is the implementation-neutral, open specification you are
reading right now.

### DrakeSpec-Compliant

The term __DrakeSpec-compliant__, when applied to a [__CI/CD
platform__](#ci/cd-platforms), [__utility__](#utilities), or any other software
denotes faithful support for [__the DrakeSpec__](#drakespec).

When applied to a [__job__](#jobs) or [__pipeline__](#pipelines), the term
DrakeSpec-compliant denotes that the job or pipeline's definition syntactically
and semantically adheres to the DrakeSpec.

### The Drake DSL

__The Drake DSL__ is a YAML-based domain-specific language for the declarative
definition of [__DrakeSpec-compliant__](#drakespec-compliant)
[__jobs__](#jobs) and [__pipelines__](#pipelines).

### Pipelines

__Pipelines__ are the heart and soul of [__the DrakeSpec__](#drakespec). A
pipeline is a directed graph of inter-dependent [__jobs__](#jobs) which are
executed as a means to some end. For instance, a pipeline might be used to
execute a suite of quality checks (e.g. unit tests, integration tests, lint
checks) on a software project or to effect release, distribution, or deployment
of such a project. These are common examples, but pipelines are open-ended in
terms of what they might accomplish.

Pipelines are such a critical concept within the DrakeSpec that they are
discussed in more detail in their own [section](pipelines.md).

### Jobs

Along with [__pipelines__](#pipelines), __jobs__ are the other most fundamental
concept in [__the DrakeSpec__](#drakespec). Jobs are _containerized_ tasks. In
other words, they are tasks that execute within an OCI container. (Or in some
cases, execute in an OCI container with one or more additional OCI containers
acting in a supporting role.) A job might be used to execute a single, discrete
bit of work such as unit testing, compiling source to a binary, or uploading a
build artifact to object storage. These are common examples, but jobs are
open-ended in terms of what they might accomplish. Pipelines are _compositions_
of jobs.

Jobs are such a critical concept within the DrakeSpec that they are discussed in
more detail in their own [section](jobs.md).

### Pipeline Triggers

__Pipeline triggers__ (alternatively "triggers", for brevity), reference
discrete events that prompt execution of a [__pipeline__](#pipelines). [__The
DrakeSpec__](#drakespec) defines triggers, generically, as a concept, but
deliberately avoids enumerating any specific triggers that
[__DrakeSpec-compliant__](#drakespec-compliant) [__pipeline
executors__](#pipeline-executors) are obligated to support. In this manner,
triggers are, conceptually, an extension point within the DrakeSpec, allowing
for the emergence of many triggers based on their own (ideally open)
specifications. [__Providers__](#providers) of DrakeSpec-compliant pipeline
executors may pick and choose which triggers they will support.

Pipeline triggers are such a critical concept within the __DrakeSpec__ that they
are discussed in more detail in their own [section](pipeline-triggers.md).

### CI/CD Platforms

__CI/CD platforms__ references a class of software intended to effect continuous
integration and/or continuous delivery through the execution of
[__pipelines__](#pipelines).

In the interest of brevity, the DrakeSpec often shortens "CI/CD platform(s)" to
"platform(s)".

### Utilities

__Utilities__ references software products that cannot reasonably be classified
as [__CI/CD platforms__](#ci/cd-platforms), but may still offer some utility
with respect to [__DrakeSpec-compliant__](#drakespec-compliant)
[__jobs__](#jobs) or [__pipelines__](#pipelines). For example, a hypothetical
tool for visually modeling pipelines could not rightfully be considered a CI/CD
platform, but is still useful.

### Job Executors

__Job executors__ references any software that executes [__jobs__](#jobs.md).

### Pipeline Executors

__Pipeline executors__ references any software that executes
[__pipelines__](#pipelines). By definition, this encompasses _all_ [__CI/CD
platforms__](#ci/cd-platforms). It may _also_ encompass some
[__utilities__](#utilities)-- for example, a command line tool such as
[__DevDrake__](https://github.com/lovethedrake/devdrake). By definition, this
also encompasses _all_ __job executors__ since pipelines are compositions of
jobs.

### Executors

__Executors__ generically references all [__job executors__](#job-executors) and
[__pipeline executors__](#pipeline-executors) alike.

### Job Producers

__Pipeline producers__ references any party, whether human or software, that
authors [__job__](#jobs) definitions.

### Pipeline Producers

__Pipeline producers__ references any party, whether human or software, that
authors [__pipeline__](#pipelines) definitions.

### Producers

__Producers__ generically references all [__job producers__](#job-producers) and
[__pipeline producers__](#pipeline-producers) alike.

### Job Consumers

__Job consumers__ references any software that parses [__job__](#jobs)
definitions to derive semantic meaning. By definition, this encompasses _all_
[__job executors__](#job-executors) and _all_ [__pipeline
executors__](#pipeline-executors), including all [__CI/CD
platforms__](#ci/cd-platforms). It may _also_ encompass some
[__utilities__](#utilities)-- for example, a hypothetical tool for performing
lint checks on job definitions.

### Pipeline Consumers

__Pipeline consumers__ references any software that parses [__pipeline__](#pipelines) definitions
to derive semantic meaning. By definition, this encompasses _all_ [__pipeline
executors__](#pipeline-executors), including all [__CI/CD
platforms__](#ci/cd-platforms). It may _also_ encompass some
[__utilities__](#utilities)-- for example, a hypothetical tool for performing
lint checks on pipeline definitions.

### Consumers

__Consumers__ generically references all [__job consumers__](#job-consumers) and
[__pipeline consumers__](#pipeline-consumers) alike.

### Providers

__Providers__ references the vendors or other parties (for instance, an open
source community) responsible for the development, delivery, operation,
maintenance, etc. of [__CI/CD platforms__](#ci/cd-platforms) and/or
[__utilities__](#utilities).
