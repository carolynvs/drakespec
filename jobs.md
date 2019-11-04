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
field by job producers and correct interpretation and use of each field by
DrakeSpec-compliant job executors and other job consumers.

### Implied name field

A `Job` object has no intrinsic `name` field nor any other intrinsic
identifier, but does have an extrinsic name that is implied by its key in the
enclosing `Drakefile.yaml`'s `job` map.

### `primaryContainer`

__Field name:__ `primaryContainer`<br/>
__Field type:__ [`Container`](#container)<br/>
__Required:__ Y<br/>

Job producers MUST populate the `primaryContainer` field of a `Job` object with
a `Container` object that describes an OCI container.

DrakeSpec-compliant job executors MUST launch an OCI container corresponding to
the `Container` object in the `primaryContainer` field. DrakeSpec-compliant job
executors MUST consider a `Job` to be complete when the main process of the
primary container completes or an implementation-defined timeout interval has
elapsed. DrakeSpec-compliant job executors MUST determine success or failure of
a `Job` on the basis of the exit code from the primary container's main process,
with an exit code of `0` indicating success and all other values indicating
failure.

### `sidecarContainers`

__Field name:__ `sidecarContainers`<br/>
__Field type:__ `[]`[`Container`](#container)<br/>
__Required:__ N<br/>

Job producers MAY populate the `sidecarContainers` field of a `Job` object with
an ordered list of `Container` objects, each of which describes an OCI container
that plays a supporting role in execution of the `Job`.

Note: Suggested uses for sidecar containers include executing long-running
processes that the main process of the primary container may depend on in some
fashion. For instance, a job that implements a suite of tests requiring access
to a live database might execute the tests themselves in the primary container
while hosting the database in a sidecar container.

For `Job` objects having more than one `Container` object in the
`sidecarContainers` field, DrakeSpec-compliant job executors MUST establish
networking among the primary container and all sidecar container that permits
processes running in each to address open ports on all others using the local
interface.

DrakeSpec-compliant job executors MUST launch OCI containers corresponding to
the `Container` objects in a `Job`'s `sidecarContainers` field _in the order
they are defined_. Although sidecar containers MUST be launched in the order
they were defined, executors MUST NOT make any attempt to delay or defer
launching of any container until the processes running in prior containers are
running or otherwise ready. Ensuring coordination among processes is strictly
the responsibility of job producers.

DrakeSpec-compliant job executors MUST forcefully terminate _all_ sidecar
containers (if any) that remain running upon conclusion of the primary
container's main process.

### `sourceMountMode`

__Field name:__ `sourceMountMode`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>
__Allowed values:__ `RO`, `COPY`, `RW`<br/>
__Default value:__ `RO`<br/>

Job producers MAY populate the `sourceMountMode` field of a `Job` object with
a value that indicates how any applicable application source should be mounted
into any of the job's containers which may require source code.

For any job with one or more containers requiring access to any applicable
source code (per the `sourceMountPath` field), DrakeSpec-compliant job executors
MUST mount that source into the corresponding OCI container in accord with the
following rules:

| `sourceMountMode` | Method |
|-------------------|--------|
| `RO` | For each of a job's applicable containers, the source MUST be mounted to the indicated paths in the corresponding OCI containers _in a read-only fashion_ that prevents mutation of the source. |
| `COPY` | One exact copy of the source MUST be created (through implementation-defined means) for each job. For each of a job's applicable containers, that copy MUST be mounted to the indicated paths in the corresponding OCI containers. Original source MUST NOT be mutated. |
| `RW` | For each of a job's applicable containers, the source MUST be mounted to the indicated paths in the corresponding OCI containers _in a writable fashion_ that permits mutation of the source. Jobs utilizing this source mount mode are ineligible for execution within the context of a pipeline. |

Note: Jobs are not generally intended to mutate source code. Within the context
of a pipeline, this general expectation permits every job to assume it is
operating upon code that has not been altered in any way by "upstream" jobs.
(The DrakeSpec defines a separate mechanism-- shared storage-- that is intended
to facilitate a clean "hand-off" of build artifacts from any job within a
pipeline to "downstream" jobs.) The `RO` and `COPY` source mount modes offer
varying levels of flexibility and performance, while preventing any job from
mutating source code. In the event that it is desired for a job to directly
alter source code, it may utilize the `RW` source mount mode, but by doing so
becomes ineligible for use within a pipeline. Such a job might still be
independently executable by some job executors. This can, for instance, be an
effective means of containerizing workflow element that mutate source code at
development time. For example, the DrakeSpec authors have a habit of "vendoring"
third-party libraries and frequently utilize jobs with an `RW` source mount mode
to effect dependency resolution at development time.

## `Container` Fields

This section describes the fields of a `Container` object that represents an OCI
container, including valid use of each field by job producers and correct
interpretation and use of each field by DrakeSpec-compliant job executors and
other job consumers.

### `name`

__Field name:__ `name`<br/>
__Field type:__ `string`<br/>
__Required:__ Y<br/>

Job producers MUST populate the `name` field of a `Container` with an identifier
that is unique among all the `Container` objects within the enclosing `Job`'s
`primaryContainer` and `sidecarContainers` fields. The name serves as an
end-user-recognizable identifier to assist in correlating any output from a
`Job` to the specific OCI container that produced it.

DrakeSpec-compliant job executors MUST clearly "label," "tag," or otherwise
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

Job producers MUST populate the `image` field of a `Container` object with the
valid, addressable name of the OCI image upon which the corresponding OCI
container must be based.

DrakeSpec-compliant job executors MUST base the corresponding OCI container on
the image specified by the `image` field of a `Container`. DrakeSpec-compliant
job executors MAY support private image registries that require authentication.
The DrakeSpec is intentionally vague about _how_ job executors manage
credentials and authentication, leaving it implementation-defined.
DrakeSpec-compliant job executors MUST NOT implement support for private image
registries through any mechanism that requires credentials to appear within a
`Drakefile.yaml`.

### `environment`

__Field name:__ `environment`<br/>
__Field type:__ `[]string`<br/>
__Required:__ N<br/>

Job producers MAY populate the `environment` field of a `Container` object with
an ordered list of `string` values containing key/value pairs delimited by a `=`
character (equal sign). If multiple `=` characters appear in a single `string`,
DrakeSpec-compliant job consumers MUST treat only the first `=` character as a
delimter and MUST treat all subsequent `=` characters as part of the value. If a
`string` lacks any `=` character, DrakeSpec-compliant job consumers MUST treat
the `string` as a key with an implied empty string value.

DrakeSpec-compliant job executors MUST utilize the key/value pairs derived from
the `environment` field of a `Container` object as environment variables and
their corresponding values in the corresponding OCI container. In constructing
such a container, the environment variable / value pairs MUST be utilized in the
order they were defined. DrakeSpec-compliant job executors MUST offer additional
means of specifying an OCI container's environment variables that are defined
somewhere other than a `Drakefile.yaml`. This is intended to facilitate
management of environment variables with sensitive values. The DrakeSpec is
intentionally vague about _how_ job executors may achieve this, leaving it
implementation-defined, however, DrakeSpec-compliant job executors MUST allow
environment variable / value pairs defined through the `environment` field of a
`Container` object to supercede environment variable / value pairs defined
through other means.

### `workingDirectory`

__Field name:__ `workingDirectory`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>

Job producers MAY populate the `workingDirectory` field of a `Container` object
to specify the working directory for the corresponding OCI container's main
process.

If defined, DrakeSpec-compliant job executors MUST utilize the value of the
`workingDirectory` field of a `Container` object at the working directory for
the corresponding OCI container's main process. If undefined,
DrakeSpec-compliant job executors MUST leave the same undefined when launching
the corresponding OCI container-- in which case the working directory for the
corresponding OCI container's main process is determined by the details of the
OCI image and by the OCI runtime.

### `command`

__Field name:__ `command`<br/>
__Field type:__ `[]string`<br/>
__Required:__ N<br/>

Job producers MAY populate the `command` field of a `Container` object with an
ordered list of `string` values that collectively specify the command for
launching the main process of the corresponding OCI container. The
[`args`](#args) field MAY be used to provide further arguments to the command.

If defined, DrakeSpec-compliant job executors MUST utilize the values of the
`command` field of a `Container` object as the `ENTRYPOINT` of the corresponding
OCI container. If undefined, DrakeSpec-compliant job executors MUST leave the
same undefined when launching the corresponding OCI container-- in which case
behavior with respect to this field is determined by the details of the OCI
image and by the OCI runtime.

### `args`

__Field name:__ `args`<br/>
__Field type:__ `[]string`<br/>
__Required:__ N<br/>

Job producers MAY populate the `args` field of a `Container` object with an
ordered list of arguments to the `ENTRYPOINT` (see [`command`](#command)) of the
corresponding OCI container.

If defined, DrakeSpec-compliant job executors MUST utilize the values of the
`args` field of a `Container` object as the `CMD` of the corresponding OCI
container. If undefined, DrakeSpec-compliant job executors MUST leave the same
undefined when launching the corresponding OCI container-- in which case
behavior with respect to this field is determined by the details of the OCI
image and by the OCI runtime.

### `tty`

__Field name:__ `tty`<br/>
__Field type:__ `boolean`<br/>
__Required:__ N<br/>
__Default:__ `false`

Job producers MAY populate the `tty` field of a `Container` object to indicate
whether a pseudo-TTY should be allocated for the corresponding OCI container's
main process.

If a value is specified for the `tty` field of a `Container` object and the
value is `true`, DrakeSpec-compliant job executors MUST allocate a pseudo-TTY
for the corresponding OCI container's main process. If these conditions are not
met, DrakeSpec-compliant job executors MUST NOT allocate a pseudo-TTY for the
corresponding OCI container's main process

### `privileged`

__Field name:__ `privileged`<br/>
__Field type:__ `boolean`<br/>
__Required:__ N<br/>
__Default:__ `false`

Job producers MAY populate the `privileged` field of a `Container` object to
indicate whether the corresponding OCI container should be privileged. Processes
within privileged containers generally execute with a level of permissions more
similar to that of processes executing directly on the container's host.

__Job producers are recommended to utilize this option sparingly and with
caution. Executing untrusted code within a privileged container opens a
significant and easily exploitable attack vector.__

If a value is specified for the `privileged` field of a `Container` object, and
the value is `true`, _and_ the use of privileged containers is not otherwise
restricted (see below), DrakeSpec-compliant job executors MUST launch a
privileged container. If these conditions are not met, DrakeSpec-compliant job
executors MUST NOT launch a privileged container.

DrakeSpec-compliant job executors SHOULD enforce some restrictions on the use of
this feature. The scope of those restrictions is implementation-defined. By way
of example, some job executors could unconditionally refuse to run privileged
containers. Other job executors which are also pipeline executors could limit
the feature to use in pipelines that are triggered by events that originated
from specific, trusted users. If a privileged container is requested, but not
permitted per implementation-defined restrictions, DrakeSpec-compliant job
executors MUST NOT (silently or otherwise) "downgrade" to an unprivileged
container. Instead, in such cases, DrakeSpec-compliant job executors MUST
generate an appropriate error message and consider execution of the enclosing
`Job` to have failed.

### `mountDockerSocket`

__Field name:__ `mountDockerSocket`<br/>
__Field type:__ `boolean`<br/>
__Required:__ N<br/>
__Default:__ `false`

Job producers MAY populate the `mountDockerSocket` field of a `Container`
object to indicate whether the container host's Docker socket at
`/var/run/docker.sock` should be mounted to the same path within the
corresponding OCI container's file system.

__Job producers are recommended to utilize this option sparingly and with
caution. Executing untrusted code within a container with access to the host's
Docker socket opens a significant and easily exploitable attack vector.__

If a value is specified for the `mountDockerSocket` field of a `Container`
object, and the value is `true`, _and_ mounting the Docker socket into the
corresponding OCI container is not otherwise restricted (see below),
DrakeSpec-compliant job executors MUST mount the host's Docker socket at
`/var/run/docker.sock` to the same location within the corresponding OCI
container's file system. If these conditions are not met, DrakeSpec-compliant
job executors MUST NOT mount the host's Docker socket into the
corresponding OCI container's file system.

DrakeSpec-compliant job executors SHOULD enforce some restrictions on the use of
this feature. The scope of those restrictions is implementation-defined. By way
of example, some job executors could unconditionally refuse to mount the host's
Docker socket into containers. Other job executors which are also pipeline
executors could limit the feature to use in pipelines that are triggered by
events that originated from specific, trusted users. If mounting the host's
Docker socket into the corresponding OCI container is requested, but not
permitted per implementation-defined restrictions, DrakeSpec-compliant job
executors MUST NOT (silently or otherwise) "downgrade" to a container lacking
the requested mount. Instead, in such cases, DrakeSpec-compliant job executors
MUST generate an appropriate error message and consider execution of the
enclosing `Job` to have failed.

__The DrakeSpec authors acknowledge that this section of the spec appears to be
too specific to Docker and may not be sufficiently generic in terms of support
for other OCI container runtimes. Due to this, the authors urge caution as this
section of the specification is very likely to change.__

### `sourceMountPath`

__Field name:__ `sourceMountPath`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>

Job producers MAY populate the `sourceMountPath` field of a `Container` object
to indicate where within the corresponding OCI container's file system any
applicable source code should be mounted.

If a non-empty value is specified for the `sourceMountPath` field of a
`Container`, DrakeSpec-compliant job executors MUST mount applicable source code
to the specified path within the corresponding OCI container's file system and
MUST mount it in a manner that complies with the `sourceMountMode` of the
encloding `Job` object. If the `sourceMountPath` field of a `Container` is
undefined or empty, DrakeSpec-compliant job executors MUST NOT mount source code
into the corresponding OCI container's file system at any location.

The specific method of locating and mounting applicable source code into the
corresponding OCI container's file system is deliberately unspecified and left
implementation-defined. It is useful to examine two very different
implementations of this requirement for the sake of example.

[__DevDrake__](https://github.com/lovethedrake/devdrake) implements this by bind
mounting the working directory (for the `RO` source mount mode) or a copy of the
working directory (for the `COPY` source mount mode) from the the host's file
system into the container's file system.

[__BrigDrake__](https://github.com/lovethedrake/brigdrake) implements this by
utilizing an "init container" to clone a source code as indicated by incoming
GitHub webhook events.

### `sharedStorageMountPath`

__Field name:__ `sharedStorageMountPath`<br/>
__Field type:__ `string`<br/>
__Required:__ N<br/>

Job producers MAY populate the `sharedStorageMountPath` field of a `Container`
object to indicate where within the corresponding OCI container's file system a
shared storage volume should be mounted.

If a non-empty value is specified for the `sharedStorageMountPath` field of a
`Container`, DrakeSpec-compliant job executors MUST mount a shared storage
volume to the specified path within the corresponding OCI container's file
system. For DrakeSpec-compliant job executors that are also DrakeSpec-compliant
pipeline executors, one and only one such volume MUST be provisioned per
pipeline and mounted to the specified path within all applicable containers of
all applicable jobs. For all other DrakeSpec-compliant job executors (i.e. those
executing standalone jobs), such a volume MUST still be provisioned (even though
it will not be shared across multiple jobs) and MUST still be mounted to the
specified path within all applicable containers.

__Note: The DrakeSpec authors anticipate significant changes to the spec as it
relates to shared storage-- namely, they anticipate requiring "layered" storage
that enables re-execution of a failed job by returning to the state it was in
prior to initial execution of that job. Due to this, the authors urge caution as
this section of the specification is very likely to change.__
