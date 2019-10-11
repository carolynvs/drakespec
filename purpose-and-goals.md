# Purpose and Goals

The __drakespec__ is an _open specification_ that describes:

1. A domain-specific language for declaratively defining CI/CD pipelines

1. Runtime requirements for CI/CD platforms that execute those pipelines

Open source and commercial CI/CD platforms (and related tools) are both
permitted and encouraged to adopt support for this specification. By doing so,
they will advance the spec authors' three primary (and inter-related)
objectives:

1. Enabling pipeline portability

1. Achieving parity between the local development environment and CI

1. Commoditizing the CI/CD marketplace

## Pipeline Portability

In the current state of the industry, developers typically define CI/CD
pipelines using syntax specific to a given CI/CD platform. By way of examples:

* CircleCI configuration does not work with Travis CI.

* Travis CI configuration does not work with Jenkins.

* A `Jenkinsfile` won't work with CircleCI _or_ Travis CI.

* A `Jenkinsfile` may not work on _all_ Jenkins instances because of wide
  variability in how Jenkins itself can be configured.

Syntax differences aside, different CI/CD platforms are often not _semantically_
equivalent to one another either. Again, by way of example, Circle CI models
each job in a "workflow" (their name for a pipeline) as one or more co-operating
containers whose lifespan is scoped to the _job_. Jenkins, on the other hand,
models entire _pipelines_ as one or more co-operating containers whose lifespan
is scoped to the _pipeline_. Each job simply executes commands in containers
that will outlive it and persist until the _entire pipeline_ completes. Neither
approach is inherently superior to the other, but the contrast between the two
is noteworthy.

These inherent syntactic and semantic differences between existing (and popular)
CI/CD platforms seem to be taken for granted. Developers (begrudgingly) _accept_
that if they need to move their pipelines from one platform to another (for
whatever reason-- cost, etc.), they will likely incur significant effort not
only to adapt their pipelines to the new platform's syntax, but will also,
likely, need to _re-model_ their pipelines based on the new platform's unique
semantics and idiosyncrasies.

Worse still, for developers porting pipelines from one CI/CD platform to
another, there is frequently no convenient way to execute pipelines locally.
Nearly every developer who has ever modeled a CI or CD pipeline will report
experiences of crafting test PRs in order to troubleshoot their workflows.

The authors of the drakespec, having extensive experience with CI/CD, have noted
that most pipelines we have encountered _could_ be modeled and described using a
common, ubiquitous domain-specific language-- if only such a thing existed. The
drakespec seeks to bring just such a DSL into existence, then, by encouraging
new and existing CI/CD providers to support the specification, usher in an era
of "write once, run anywhere" pipelines.

## Development Environment / CI Parity

Having contributed to innumerable software projects-- both open source and
commercial-- the drakespec authors have observed how difficult it is to ensure
all of a project's contributors (many of whom may contribute on a casual or
part-time basis) work with a _uniform_ or "certified" development environment.
Too often, this difficulty leads to cases of "it worked on my machine," and
precious contributor and maintainer hours wasted troubleshooting the system
disparities that caused a test to pass locally but fail in CI. Even when this
problem doesn't arise, careful curation of the languages, libraries, tools, etc.
present on a contributor's laptop may have been required to head it off. Worse,
the required dependencies for building or testing one project may conflict with
the dependencies for building or testing another project. _No one has time for
this._

This problem and its less than ideal solution have been written about
extensively
[here](https://medium.com/swlh/containerized-development-101-9e9711397f12) by
one of the drakespec's authors.

Through containerization of a pipeline's constituent jobs, the drakespec's
authors wish to minimize the degree of setup required to execute tests, builds,
or other development-related tasks for any drakespec-compliant project. More
importantly, by ensuring pipeline portability, the authors wish to unlock the
ability for contributors to such projects to execute individual jobs or entire
pipelines _locally_ before ever opening a PR.

By ensuring _parity_ between the local development environent and CI,
contributors can be confident that jobs and pipelines that execute correctly
locally will do so as well in CI.

## Commoditizing the CI/CD Marketplace

Though it is the primary benefit, the authors of the drakespec do not wish for
pipeline portability _only_ for the sake of developer convenience and
productivity. Pipeline portability is additionally a means to an end in terms of
commoditizing the CI/CD marketplace.

Once developers can pick up their pipelines and move them around more easily,
_they will_. While this will almost certainly, over time, work to the advantage
of some providers and to the disadvantage of others, the authors anticipate that
this will _strengthen_ the CI/CD marketplace as a whole.

The perceived benefits of this commoditization are many:

1. No lock-in:

    1. With little or no effort, developers can leave slow or expensive
       drakespec-compliant providers for faster or more cost-effective
       drakespec-compliant providers.

    1. Developers can try new or different drakespec-compliant providers with
       minimal effort and no risk.

1. Better providers emerge:

    1. New providers can enter the marketplace more easily and with less risk.
       Supporting the drakespec means pipelines compatible with your new service
       already exist in the wild.

    1. With providers supporting a common pipeline model, advantage in the
       marketplace tilts toward providers that are fastest and most
       cost-effective.

## Meaningful Comparisons

The drakespec's mission of ushering in an era of "write once, run anywhere"
pipelines can easily be compared to three familiar success stories-- __Java__,
__Docker__, and __Kubernetes__.

In the 1990s, __Java__ disrupted the market for high level, general-purpose
programming languages, which was dominated at the time by the likes of C and
C++. While C and C++ programs had to be compiled anew for each target operating
system and CPU architecture, Java code could be compiled _once_ into bytecode
that could be executed on any system having a Java Virtual Machine.

Prior to the advent of __Docker__ containers, servers-- whether physical or
virtual-- had to be configured _just so_ to meet the unique requirements of any
applications they hosted. Today, countless servers the world-over are completely
unremarkable-- knowing how to do only one thing and do it well-- host Docker
containers. Complementing that, Docker images have emerged as the _de facto_
standard for packaging and distributing server-side applications. Now any
application than can run in a container can be deployed anywhere there is a
Docker daemon.

__Kubernetes__ has taken Docker's success to the next level. Having emerged as
the _de facto_ standard for container orchestration and having many, many
certified distributions hosted on premises or in public clouds, Kubernetes
manifests (with the help of the __Helm__ package manager) have become the "write
once, run anywhere" option for declaratively describing and deploying complex
"cloud native" applications.

As with Java, Docker, and Kubernetes, the drakespec aims to establish a _de
facto_ standard to promote _portability_ where none previously existed.

## Non-Goals

This section is intended to clarify the drakespec's scope and dispel any
erroneous notions about its goals or market positioning.

### General Purpose; _Not_ All-Purpose

The drakespec describes a domain-specific language for declaratively defining
CI/CD pipelines that are composed _exclusively_ of jobs that are isolated to and
executed within OCI containers. Two practical corollaries follow from this:

1. The constituent jobs of a drakespec-compliant pipeline may execute _any_ task
   that can reasonably be executed within an OCI container, making
   drakespec-compliant pipelines and the CI/CD platforms that execute them
   _general purpose_.

1. The constituent jobs of a drakespec-compliant pipeline _cannot_ execute any
   task that _cannot_ reasonably be executed within an OCI container, which
   precludes drakespec-compliant pipelines and the CI/CD platforms that execute
   them from being classified as _all-purpose_.

Being general pupose, drakespec-compliant pipelines can accommodate a very wide
range of use cases, but that range excludes, for instance, executing builds or
tests that rely on OS kernels other than Linux or Windows, since only those two
OS families are supported by current OCI implementations. If, for example, your
project is an iOS app that can only be compiled and tested on Mac OS, you are
unlikely to find drakespec-compliant pipelines useful.

### Intentional Lack of Language Support

Many existing CI/CD platforms have a legacy of direct support for specific
languages, build tools, etc. Even among those that do not, extensions are often
available to add such support.

As OCI containers have trended toward ubiquity, many existing CI/CD platforms
have _added_ support for containerized jobs, but have also retained their legacy
feature set.

It is the opinion of the drakespec's authors that given the unparalleled
flexibility of OCI containers to support a near-infinite range of languages,
build tools, etc. (simply through selection of appropriate images) any _direct_
support for specific languages, build tools, etc. is a feature that has outlived
its usefulness. As a matter of practicality, the drakespec deliberately
prescribes _no_ direct support for any specific language, build tool, etc.

### Intentional Lack of VM Support

Many existing CI/CD platforms have a legacy of executing entire pipelines or
their constituent jobs in virtual-machine-based "executors."

As OCI containers have trended toward ubiquity, many existing CI/CD platforms
have _added_ support for jobs that are executed within containers instead. Some
platforms have gone as far as promoting container-based executors as a preferred
or default option, but many also have retained their legacy support for VM-based
executors.

It is the opinion of the drakespec's authors that although VMs may support some
use cases that are not supported by containers (for instance, support for OS
families other than Linux and Windows), containers, on their own, cover a _broad
enough_ range of use cases such that complicating the drakespec by prescribing
support for VM-based executors is unwarranted and would only raise the barrier
to adoption for would-be drakespec-compliant CI/CD platforms.

### It's _Not_ a Build Tool

It is, perhaps, too easy to mistake certain implementations of the drakespec
(devdrake in particular) as a "build tool." The drake "origin story" can both
confirm the ease with which this mistake can be made-- because the drakespec
authors once made the same mistake themselves-- and can also dispel any
confusion regarding this matter.

The concept that evolved into the drakespec was born among Go developers who
were consumate Docker and `make` users with a strong preference for isolating
the tasks comprising our local workflows to containers. When typing `make test`
on our laptops, for instance, we wished, quite sensibly, for the underlying `go
test` process to execute identically to how it would in CI-- i.e. in an
identical container. We often wished for a replacement for `make` that was
natively "container-aware" in the same way that our CI platforms were. We would
usually compensate by scripting some rudimentary container orchestration into
our `make` targets, then inevitably encounter an undesired "Docker in Docker"
scenario in CI where _both_ the CI platform and our `make` targets would be
trying to wrap the underlying process in a new container. Our scripts would grow
to include "escape hatches" so CI jobs could signal to a `make` target that the
process was _already_ executing within a container. You can read about this more
[here](https://medium.com/swlh/containerized-development-101-9e9711397f12).
While this approach worked well across many projects, we became disenchanted
with the proliferation of "glue code" that most project contributors didn't
entirely understand and were too afraid to touch.

Convinced as we were (at the time) that `make`'s lack of native
container-awareness was the root of our problem, we dreamt up a hypothetical
container-aware replacement for `make`. The name "drake" is, admittedly, a
portmanteau of "Docker" and `make`.

Over time, however, our understanding of the problem evolved. Before any
prototype was ever devised or any spec drafted, we recognized that our build
tool of choice (`make`) was _not_ deficient in any way. _It was our CI platforms
that were._ There would be no _need_ for our build tool to manage containers if
only our CI pipelines (and the jobs that comprised them) were portable-- most
notably portable to a developer's laptop. With drake still being hypothetical at
the time, we came to envision a world in which a command like `drake run <job or
pipeline name>` could handle container orchestration for a job or entire
pipeline while leaving the tools utilized _within_ those containers to just work
as they normally do with no inherent container-awareness. At that time, the
concept behind drake ceased to involve replacing `make` and became exclusively
about achieving pipeline portability. The name "drake," however, stuck because,
let's be honest-- it's a cool name.

The moral of the origin story is that, in the minds of the drakespec's authors,
implementations of the specification are _not_ intended to replace existing
build tools. If you're using `make` and you like `make`, _keep using it_. If
you're a Ruby developer and you're using `rake` and you like `rake`, _keep using
it_. If you're using Bazel and you like it, _keep using it_. Build tools you are
already happy with should, generally speaking, work just fine with _within_ a
well-designed drakespec-compliant pipeline.

### Relationship to Kubernetes

The drakespec can be said, in a way, to be attempting to do for CI/CD pipelines
what Kubernetes has done for the hosting of "cloud native" applications.

Kuberntes excels at orchestrating OCI containers that execute _long-running_
workloads such as application servers, for instance, and thanks to its ubiquity
and many, many certified distributions hosted on premises or in public clouds,
Kubernetes manifests are highly (if perhaps not perfectly) portable.

CI/CD platforms that adopt the drakespec are intended to excel at orchestrating
OCI containers that execute _directed graphs of comparatively short-running
workloads_, such as builds and tests. Through openness, network effects, and the
allure of pipeline portability, the drakespec aims to make make compliant
pipelines as portable in the near future as Kubernetes manifests are today.

The drakespec is _distantly_ related to Kubernetes in one other way as well,
insofar as Kubernetes is a practical enabling technology for drakespec-compliant
platforms. This is to say it is perfectly reasonable that some
drakepec-compliant platforms (including the drakepec authors' own early
prototypes) might leverage Kubernetes as an "implementation detail."

_It must be explicitly stated that the drakespec's relationship to Kubernetes
runs no deeper than this._

A new generation of CI/CD platforms (including the likes of Argo and Tekton
Pipelines) is emerging that model jobs, pipelines, etc. as Kubernetes resources
(i.e. CRDs-- custom resource definitions). While the drakespec authors stop
short of overtly criticizing this approach, it should be noted that this is an
approach was considered for drakespec and rejected because the authors wished
neither to constrain the implementation of drakespec-compliant platforms by
mandating a dependency on Kubernetes (devdrake, for instance does not rely on
Kubernetes) nor presume or require pipeline authors to be competent Kubernetes
users.
