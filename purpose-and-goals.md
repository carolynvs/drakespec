# Purpose and Goals

__drakespec__ is an _open specification_ that describes a domain-specific
language for declaratively defining CI/CD pipelines. All manner of open source
and commercial CI/CD platforms and related tools are both permitted and
encouraged to adopt support for this specification.

By adopting support for this specification, implementors will advance the spec
authors' two primary (and related) objectives:

1. Enabling pipeline portability
1. Commoditizing the CI/CD marketplace

## Pipeline Portability

In the current state of the industry, developers typically define CI/CD
pipelines using syntax specific to a given CI/CD platform. By way of example,
CircleCI configuration does not work with Travis CI. Travis CI configuration
does not work with Jenkins. A `Jenkinsfile` won't work with CircleCI _or_ Travis
CI. And worse, it may not even work on _all_ Jenkins instances because there is
so much variability in how Jenkins itself can be configured.

Syntax differences aside, different CI/CD platforms are often not _semantically_
equivalent to one another either. Again, by way of example, Circle CI models
each job in a pipeline (workflow) as one or more co-operating containers whose
lifespan is scoped to the _job_. Jenkins, on the other hand, models entire
_pipelines_ as one or more co-operating containers whose lifespan is scoped to
the _pipeline_. Each job simply executes commands in containers that will
outlive it and persist until the _entire pipeline_ completes. Neither approach
is inherently superior to the other, but the contrast between the two is
noteworthy.

These inherent syntactic and semantic difference between existing (and popular)
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

Placeholder