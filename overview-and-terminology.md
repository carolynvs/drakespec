# Overview and Terminology

This section presents a high-level overview of the drakespec. It is a reasonable
and common starting point for developers wishing to implement
drakespec-compliant platforms or utilities as well as developers wishing to
author drakespec-compliant pipelines.

## Glossary of Terms

This section defines and/or disambiguates key terms that are used throughout
the remainder of the specification.

Note that terms in this section are not organized alphabetically. Rather, they
are introduced in an order the authors consider sensible for readers previously
unfamiliar with this specification.

### drake

The unqualified term __drake__ references a particular drakespec-compliant
utility which is alternatively known by the less ambiguous name
[__devdrake__](https://github.com/lovethedrake/devdrake).

drake / devdrake is not discussed at any length within this specification. The
term is defined here only to _discourage_ readers, users, contributors, etc.
from incorrectly using the unqualified term "drake" in reference to the
specification-- which is properly refered to as [__the drakespec__](#drakespec).

### drakespec

The __drakespec__ is the implementation-neutral, open specification you are
reading right now.

As a matter of style, "drakespec" is never capitalized when written and is
typically prefixed with the definite article "the," especially wherever required
to comply with typical English capitalization rules (e.g. at the beginning of a
sentence; e.g. "The drakespec is ...").

### drakespec-compliant

The term __drakespec-compliant__, when applied to a [__CI/CD
platform__](#ci/cd-platforms) or [__utility__](#utilities) denotes that the
software product in question faithfully supports the
[__drakespec__](#drakespec).

When applied to a [__pipeline__](#pipelines), the term __drakespec-compliant__
denotes that the pipeline's definition syntactically and semantically adheres to
the drakespec.

As a matter of style, "drakespec-compliant," like the word "drakespec" is never
capitalized when written. To avoid conflict with typical English capitalization
rules (e.g. at the beginning of a sentence), additional verbiage may be used to
introduce "drakespec-compliant" at the beginning of a sentence. For instance,
instead of beginning a sentence, "drakespec-compliant platforms..." consider
beginning the sentence, "All drakespec-compliant platforms..." or,
situationally, "Most drakespec-compliant platforms..."

### Pipelines

__Pipelines__ are the heart and soul of the drakespec. A pipeline is a directed
graph of related [__jobs__](#jobs) which are executed as a means to some end.
For instance, a pipeline might be used to execute a suite of quality checks
(e.g. unit tests, integration tests, lint checks) on a software project or to
effect release, distribution, or deployment of such a project. These are common
examples, but pipelines are open-ended in terms of what they might accomplish.

Pipelines are such a critical concept within the drakespec that they are
discussed in more detail in their own [section](pipelines.md).

### Jobs

Along with [__pipelines__](#pipelines), __jobs__ are the other most fundamental
concept in the drakespec. Jobs are _containerized_ tasks. In other words, they
are tasks that execute within OCI containers. (Or in some cases, execute in an
OCI container with one or more additional containers acting in a supporting
role.) A job might be used to execute a single, discrete bit of work such as
unit testing, compiling source to a binary, or uploading a build artifact to
object storage. These are common examples, but jobs are open-ended in terms of
what they might accomplish. Pipelines are _compositions_ of jobs.

Jobs are such a critical concept within the drakespec that they are discussed in
more detail in their own [section](jobs.md).

### Pipeline Triggers

__Pipeline triggers__ (alternatively "triggers", for brevity), reference
discrete events that prompt execution of a [__pipeline__](#pipelines). The
drakespec defines triggers, generically, as a concept, but deliberately avoids
enumerating any specific triggers that
[__drakespec-compliant__](#drakespec-compliant) platforms are obligated to
implement. In this manner, triggers are, conceptually, an extension point within
the drakespec, allowing for the emergence of many triggers-- themselves based on
(ideally) open specifications-- which drakespec-compliant
[__platforms__](#ci/cd-platforms) and [__utilities__](#utilities) may elect to
or decline to support.

Pipeline triggers are such a critical concept within the __drakespec__ that they
are discussed in more detail in their own [section](pipeline-triggers.md).

### CI/CD Platforms

__CI/CD platforms__ references a class of software intended to effect continuous
integration and/or continuous delivery through the execution of
[__pipelines__](#pipelines).

In the interest of brevity, the drakespec often shortens "CI/CD platform(s)" to
"platform(s)".

### Utilities

__Utilities__ references software products that cannot reasonably be classified
as [__CI/CD platforms__](#ci/cd-platforms), but may still offer some utility
with respect to drakespec-compliant pipelines. For example, a hypothetical tool
for visually modeling pipelines could not rightfully be considered a CI/CD
platform, but is still useful.

### Pipeline Executors

__Pipeline executors__ references any software that executes
[__pipelines__](#pipelines). By definition, this encompasses _all_ [__CI/CD
platforms__](#ci/cd-platforms). It may _also_ encompass some
[__utilities__](#utilities)-- for example, a command line tool such as
[__devdrake__](https://github.com/lovethedrake/devdrake) for executing pipelines
in the local development environment.

### Pipeline Producers

__Pipeline producers__ references any [__utility__](#utilities) that authors
or aids in the authorship of pipeline definitions.

### Pipeline Consumers

__Pipeline consumers__ references any software that parses pipeline definitions
to derive semantic meaning. By definition, this encompasses _all_ [__pipeline
executors__](#pipeline-executors), including all [__CI/CD
platforms__](#ci/cd-platforms). It may _also_ encompass some
[__utilities__](#utilities)-- for example, a hypothetical tool for performing
lint checks on pipeline definitions.

### Providers

__Providers__ references the vendors or other parties (for instance, an open
source community) responsible for the development, delivery, operation,
maintenance, etc. of [__CI/CD platforms__](#ci/cd-platforms) and/or
[__utilities__](#utilities).
