# Jobs

Along with [__pipelines__](pipelines.mid), __jobs__ are the other most
fundamental concept in the Drake DSL. `Job` objects represent _containerized_
tasks. In other words, tasks that execute within an OCI container. (Or in some
cases, execute in an OCI container with one or more additional OCI containers
acting in a supporting role.) A `Job` might be used to execute a single,
discrete bit of work such as unit testing, compiling source to a binary, or
uploading a build artifact to object storage. These are common examples, but
`Job`s are open-ended in terms of what they might accomplish. `Pipeline`s are
_compositions_ of `Job`s.

## `Job` Fields

This section describes the fields of a `Job` object, including valid use of each
field by pipeline producers and correct interpretation and use of each field by
DrakeSpec-compliant executors and other consumers.

### Implied name field

A `Job` object has no intrinsic `name` field nor any other intrinsic
identifier, but does have an extrinsic name that is implied by its key in the
enclosing `Drakefile.yaml`'s `job` map.

### `containers`

__Field name:__ `containers`<br/>
__Field type:__ `[]`[`Container`](#container)<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the `containers` field of a `Job` object with an
ordered list of `Container` objects, each of which describes an OCI container
that plays a role in execution of the `Job`. A `Job` object with no values in
the `containers` field is currently considered valid by this specification, but
is, by definition, a trivial `Job` that could serve no purpose. For `Job`
objects having one or more `Container` objects in the `containers` field, the
_last_ `Container` object is special among all others in the same collection and
is counted as the `Job`'s _primary container_. All remaining containers are
considered _supporting containers_. (These are also, colloquially, known as
"sidecar containers.")

Note: Suggested uses for supporting containers include executing long-running
processes that the main process of the main container may depend on in some
fashion. For instance, a job that implements a suite of tests requiring access
to a live database might execute the tests themselves in the primary container
while hosting the database in a supporting container.

For `Job` objects having more than one `Container` object in the `containers`
field, DrakeSpec-compliant pipeline executors MUST establish networking between
the corresponding OCI containers that permits processes running in each to
address open ports on all others using the local interface.

DrakeSpec-compliant pipeline executors MUST launch OCI containers corresponding
to the `Container` objects in a `Job`'s `containers` field _in the order they
are defined_. Necessarily, this means _all_ supporting containers (if any) are
launched prior to the primary container. DrakeSpec-compliant pipeline executors
MUST consider a `Job` to be complete when the main process of the primary
container completes or an implementation-defined timeout interval has elapsed.
DrakeSpec-compliant pipeline executors MUST determine success or failure of a
`Job` on the basis of the exit code from the primary container's main process,
with an exit code of `0` indicating success and all other values indicating
failure. DrakeSpec-compliant pipeline executors MUST forcefully terminate _all_
supporting containers (if any) that remain running upon conclusion of the
primary container's main process.

## `Container` Fields

This section describes the fields of a `Container` object that represents an OCI
container, including valid use of each field by pipeline producers and correct
interpretation and use of each field by DrakeSpec-compliant executors and other
consumers.

### `name`

__Field name:__ `name`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>

Pipeline producers MUST populate the `name` field of a `Container` with an
identifier that is unique among all the `Container` objects within the enclosing
`Job`'s `containers` field. The name serves as an end-user-recognizable
identifier to assist in correlating any output from a `Job` to the specific
OCI container that produced it.

DrakeSpec-compliant pipeline executors MUST clearly "label," "tag," or otherwise
correlate all `Job` output to the specific OCI container that produced it. The
DrakeSpec is intentionally vague about _how_ this requirement is satisfied,
leaving it implementation-defined. For the sake of illustration, it would be
valid to direct all `Job` output to a single stream with every line of output
prefixed with the name of the OCI container that produced it. It would be
equally valid to direct output from each container to its own collection of
streams indexed by name.

### `image`

__Field name:__ `image`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>

Pipeline producers MUST populate the `image` field of a `Container` object with the valid, addressable name of the OCI image upon which the corresponding OCI
container must be based.

DrakeSpec-compliant pipeline executors MUST base the corresponding OCI container
on the image specified by the `image` field of a `Container`.
DrakeSpec-compliant pipeline executors MAY support private image registries that
require authentication. The DrakeSpec is intentionally vague about _how_
pipeline executors may manage credentials and authentication, leaving it
implementation-defined. DrakeSpec-compliant pipeline executors MUST NOT
implement support for private image registries through any mechanism that
requires credentials to appear within a `Drakefile.yaml`.

### `environment`

__Field name:__ `environment`<br/>
__Field type:__ `[]string`<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the `environment` field of a `Container` object
with an ordered list of `string` values containing key/value pairs delimited by
a `=` character (equal sign). If multiple `=` characters appear in a single
`string`, DrakeSpec-compliant pipeline consumers MUST treat only the first `=`
character as a delimter and MUST treat all subsequent `=` characters as part of
the value. If a `string` lacks any `=` character, DrakeSpec-compliant pipeline
consumers MUST treat the `string` as a key with an implied empty string value.

DrakeSpec-compliant pipeline executors MUST utilize the key/value pairs derived
from the `environment` field of a `Container` pipeline as environment variables
and their corresponding values in the OCI container represented by that
`Container` object. In constructing such a container, the environment variable /
value pairs MUST be utilized in the order they were defined. DrakeSpec-compliant
pipeline executors MUST offer additional means of specifying an OCI container's
environment variables that are defined somewhere other than a `Drakefile.yaml`.
This is intended to facilitate management of environment variables with
sensitive values. The DrakeSpec is intentionally vague about _how_ pipeline
executors may achieve this, leaving it implementation-defined, however,
DrakeSpec-compliant pipeline executors MUST allow environment variable / value
pairs defined through the `environment` field of a `Container` object to
supercede environment variable / value pairs defined through other means.

### `workingDirectory`

__Field name:__ `workingDirectory`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the `workingDirectory` field of a `Container`
object to specify the working directory for the corresponding OCI container's
main process.

If defined, DrakeSpec-compliant pipeline executors MUST utilize the value of the
`workingDirectory` field of a `Container` object at the working directory for
the corresponding OCI container's main process. If undefined,
DrakeSpec-compliant pipeline executors MUST leave the same undefined when
launching the corresponding OCI container-- in which case the working directory
for the corresponding OCI container's main process is determined by the details
of the OCI image and by the OCI runtime.

### `command`

__Field name:__ `command`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>

Pipeline producers MAY populate the `command` field of a `Container` object with
a `string` value specifying the command and any applicable arguments that should
be utilized to launch the main process of the corresponding OCI container.

If defined, DrakeSpec-compliant pipeline executors MUST parse the value of the
`command` field of a `Container` object to determine the command and any
applicable arguments used to launch the main process of the corresponding OCI
container. If undefined, DrakeSpec-compliant pipeline executors MUST leave the
same undefined when launching the corresponding OCI container-- in which case
behavior with respect to this field is determined by the details of the OCI
image and by the OCI runtime.

Note: The `command` field of a `Container` object, contrasted with other
familiar methods of declaratively defining OCI containers (e.g. `compose.yaml`,
Kubernetes manifests, etc.), is simple-- being of type `string` instead of an
ordered list of `string` values with (perhaps) a complementary ordered list of
`string` command arguments. This simplicity was a deliberate choice on the part
of the DrakeSpec's authors, who wished to discourage the temptation for pipeline
producers to define complex commands inline using a tangled mess of quotes and
escape characters that may not be intuitively parsable by human users. The
DrakeSpec authors actively encourage pipeline producers to use short, simple
commands in the `command` field of a `Container` object and delegate complexity
to scripts. __This decision may be revisited in future drafts of the spec if
it proves to be an undesireable constraint.__

### `tty`

__Field name:__ `tty`<br/>
__Field type:__ `boolean`<br/>
__Required:__ N<br/>
__Default:__ `false`

Pipeline producers MAY populate the `tty` field of a `Container` object to
indicate whether a pseudo-TTY should be allocated for the corresponding OCI
container's main process.

If a value is specified for the `tty` field of a `Container` object and the
value is `true`, DrakeSpec-compliant pipeline executors MUST allocate a
pseudo-TTY for the corresponding OCI container's main process. If these
conditions are not met, DrakeSpec-compliant pipeline executors MUST NOT allocate
a pseudo-TTY for the corresponding OCI container's main process

### `privileged`

__Field name:__ `privileged`<br/>
__Field type:__ `boolean`<br/>
__Required:__ N<br/>
__Default:__ `false`

Pipeline producers MAY populate the `privileged` field of a `Container` object
to indicate whether the corresponding OCI container should be privileged.
Processes within privileged containers generally execute with a level of
permissions more similar to that of processes executing directly on the
container's host.

__Pipeline producers are recommended to utilize this option sparingly and with
caution. Executing untrusted code within a privileged container opens a
significant and easily exploitable attack vector.__

If a value is specified for the `privileged` field of a `Container` object, and
the value is `true`, _and_ the use of privileged containers is not otherwise
restricted (see below), DrakeSpec-compliant pipeline executors MUST launch a
privileged container. If these conditions are not met, DrakeSpec-compliant
pipeline executors MUST NOT launch a privileged container.

DrakeSpec-compliant pipeline executors SHOULD enforce some restrictions on the
use of this feature. The scope of those restrictions is
implementation-determined. By way of example, some pipeline executors could
unconditionally refuse to run privileged containers. Other pipeline executors
might limit the feature to use in pipelines that are triggered by events that
originated from specific, trusted users. If a privileged container is requested,
but not permitted per implementation-determined restrictions,
DrakeSpec-compliant pipeline executors MUST NOT (silently or otherwise)
"downgrade" to an unprivileged container. Instead, in such cases,
DrakeSpec-compliant pipeline executors MUST generate an appropriate error
message and consider execution of the enclosing `Job` to have failed.

### `mountDockerSocket`

__Field name:__ `mountDockerSocket`<br/>
__Field type:__ `boolean`<br/>
__Required:__ N<br/>
__Default:__ `false`

Pipeline producers MAY populate the `mountDockerSocket` field of a `Container`
object to indicate whether the container host's Docker socket at
`/var/run/docker.sock` should be mounted to the same path within the
corresponding OCI container's file system.

__Pipeline producers are recommended to utilize this option sparingly and with
caution. Executing untrusted code within a container with access to the host's
Docker socket opens a significant and easily exploitable attack vector.__

If a value is specified for the `mountDockerSocket` field of a `Container`
object, and the value is `true`, _and_ mounting the Docker socket into the
corresponding OCI container is not otherwise restricted (see below),
DrakeSpec-compliant pipeline executors MUST mount the host's Docker socket at
`/var/run/docker.sock` to the same location within the corresponding OCI
container's file system. If these conditions are not met, DrakeSpec-compliant
pipeline executors MUST NOT mount the host's Docker socket into the
corresponding OCI container's file system.

DrakeSpec-compliant pipeline executors SHOULD enforce some restrictions on the
use of this feature. The scope of those restrictions is
implementation-determined. By way of example, some pipeline executors could
unconditionally refuse to mount the host's Docker socket into containers. Other
pipeline executors might limit the feature to use in pipelines that are
triggered by events that originated from specific, trusted users. If mounting
the host's Docker socket into the corresponding OCI container is requested, but
not permitted per implementation-determined restrictions, DrakeSpec-compliant
pipeline executors MUST NOT (silently or otherwise) "downgrade" to a container
lacking the requested mount. Instead, in such cases, DrakeSpec-compliant
pipeline executors MUST generate an appropriate error message and consider
execution of the enclosing `Job` to have failed.

__The DrakeSpec authors acknowledge that this section of the spec appears to be
too specific to Docker and may not be sufficiently generic in terms of support
for other OCI container runtimes. Due to this, the authors urge caution as this
section of the specification is very likely to change.__

### `sourceMountPath`

__Field name:__ `sourceMountPath`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>

Pipeline producers MAY populate `sourceMountPath` field of a `Container` object
to indicate where within the corresponding OCI container's file system any
applicable source code should be mounted.

If a value is specified for the `sourceMountPath` field of a `Container` and the
value is `true`, DrakeSpec-compliant pipeline executors MUST mount applicable
source code to the specified path within the corresponding OCI container's file
system. If these conditions are not met, DrakeSpec-compliant pipeline executors
MUST NOT mount source code into the corresponding OCI container's file system at
any location.

The specific method of mounting applicable source code into the corresponding
OCI container's file system is deliberately unspecified and left
implementation-defined. It is useful to examine two very different
implementations of this requirement for the sake of example.

[__DevDrake__](https://github.com/lovethedrake/devdrake) implements this by
mounting the local working directory into the corresponding OCI container's file
system.

[__BrigDrake__](https://github.com/lovethedrake/devdrake) implements this by
cloning applicable source code to a pipeline's network-attached shared storage
at the onset of every pipeline's execution, then mounting that into the file
system of every OCI container within that pipeline having a non-empty value for
the `sourceMountPath` field.

__The DrakeSpec authors readily acknowledge some difficulties with this section
of the specification.__

__For one, the example implementations cited above have significant
shortcomings. Network attached storage, for instance, is notoriously slow and
likely to be a limiting factor for many implementations of DrakeSpec-compliant
pipeline executors.  Both approaches may also permit jobs to mutate and thereby
"contaminate" source code, potentially tainting the results of "downstream" jobs
that may assume "fresh" source code as input.__

__Shortcomings of the example implementations aside, the authors further
acknowledge that the entire jobs/pipelines model is likely easier to rationalize
if jobs execute in complete isolation from one another (with the exception of
well-define shared storage, which is inadequately defined at present). This
being the case, jobs requiring source code to be mounted to their OCI
container(s) file systems likely benefit from always having a "fresh" copy of
the source. The specification does not currently account for this.__

__In acknowledging the shortcomings of the referenced implementations of the of
this requirement, and more specifically, the limitations of the requirement
itself, as currently written, the DrakeSpec authors wish to urge caution as this
section of the spec is likely to evolve signigicantly in subsequent releases.__
