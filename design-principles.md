# Design Principles

This section presents a brief overview of guiding principles that have been
observed by the drakespec's authors in the development of this specification.
The authors hope that by documenting these and committing the rationale for
each to the public record, future contributors will be enabled to evolve and
improve the specification while remaining faithful to the original vision.

## The drakespec Should Make Developers' Lives Easier

Developers' jobs are hard enough. Every new technology that is introduced
increases the cognitive burden on developers who already need to juggle a
dizzying amount of knowledge. The drakespec aspires to contribute as minimally
as possible to that burden. Consequently, the drakespec benefits from being easy
to learn and generally useful. It therefore endeavors to remain simple while
still covering a broad range of use cases. Amendments to the spec must balance
these concerns, never broadening the range of usefulness through a
disproportionate increase in complexity.

## The drakespec Should Be Easy To Adopt

The drakespec aspires to gain ubiquitly quickly through the allure of portable
pipelines and network effects. i.e. The more "places" a pipeline is portable
_to_ increases the value of its _being_ portable. Consequently, the drakespec
benefits from being easy to adopt. It therefore endeavors to remain both simple
and only as prescriptive as is necessary to meet the primary objective of
ensuring pipeline portability.

## The drakespec Should Leave Room For Providers To Innovate

It is, perhaps, easy to presume that in a marketplace saturated with
drakespec-compliant CI/CD platforms, all CI/CD platforms must necessarily be
functionally identical, however, the drakespec aspires to achieve pipeline
portability _without_ stiffling innovation. Consequently, the drakespec is only
as prescriptive as is necessary to meet the primary objective of ensuring
pipeline portability. This is to say, all the requirements _not_ imposed by the
specification are as important as those that are. For instance, the drakespec
explicitly declines to impose any specific requirements with respect to pipeline
triggers, opting instead to treat this as an extension point in the
specification, thereby granting providers the flexibility to support only the
triggers they wish. Similarly, the drakespec is entirely silent on the subject
of a platform's user interfaces or overall developer experience. By prescribing
only that which is necessary to ensure pipeline portability, providers retain
significant latitude to innovate and offer unique platforms that are
differentiated in terms of integrations and developer experience.
