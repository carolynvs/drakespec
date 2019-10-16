# DrakeSpec

The __DrakeSpec__ is an _open specification_ that describes:

1. A domain-specific language for declaratively defining CI/CD pipelines

1. Requirements for software products that produce, consume, or execute those
   pipelines

Open source and commercial CI/CD platforms (and related tools) are both
permitted and encouraged to adopt support for this specification.

Together, we will usher in a new era of pipeline portability!

## THIS SPEC IS HIGHLY VOLATILE!

__This specification is currently under heavy development.__

__Adopters should be warned that breaking changes to this specification are
likely with ANY minor release prior to 1.0.0 and MUST NOT assume compliance with
a recent pre-1.0.0 release implies compliance with any _earlier_ pre-1.0.0
releases.__

## Table of Contents

1. [Purpose and Goals](purpose-and-goals.md)
1. [Overview and Terminology](overview-and-terminology.md)
1. [The Drake DSL](drake-dsl.md)
1. [Jobs](jobs.md)
1. [Pipelines](pipelines.md)
1. [Pipeline Triggers](pipeline-triggers.md)
1. [Design Principles](design-principles.md)
1. [Examples](examples.md)

## Notational Conventions

Throughout this specification, the key words "MUST", "MUST NOT", "REQUIRED",
"SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
"MAY", and "OPTIONAL" are to be interpreted as described in [RFC 2119][rfc2119].

The key words "unspecified", "undefined", and "implementation-defined" are to be
interpreted as described in the [rationale for the C99
standard][c99-unspecified].

An implementation IS compliant if it satisfies all the MUST, REQUIRED, and SHALL
requirements.

An implementation IS NOT compliant if it fails to satisfy one or more of the
MUST, REQUIRED, or SHALL requirements.

[c99-unspecified]:
http://www.open-std.org/jtc1/sc22/wg14/www/C99RationaleV5.10.pdf#page=18
[rfc2119]: http://tools.ietf.org/html/rfc2119

## Contributing

This project accepts contributions via GitHub pull requests. The
[Contributing](CONTRIBUTING.md) document outlines the process to help get your
contribution accepted.

## Code of Conduct

Although not a CNCF project, this project abides by the
[CNCF Code of Conduct](https://github.com/cncf/foundation/blob/master/code-of-conduct.md).
