# DockEnv

Wrapper for the docker client to run docker images configuring by use of environment variables. Can be extremely useful for use in systemd or build scripts. This project is still under development and may be prone to bugs as not all supported commands and options have undergone full testing. Please report any bugs found (or create a PR fixing).

The `dockenv` command is designed to be used as a replacement for the `docker` command and should support any command which can be executed with `docker` (warning when the specified command is not supported). For all commands, [DOCKER_OPTS](#docker_opts) are supported, and for select commands which have been implemented, additional [DOCKER_CMDOPTS](#docker_cmdopts) can be set. The one caveat when using as a replacement for `docker` is that all options should be specified as a single positional argument (i.e. instead of `--opt value`, use `--opt=value`).

## Options

There are two sets of options which can be configured when using `dockenv`: [DOCKER_OPTS](#docker_opts) and [DOCKER_CMDOPTS](#docker_cmdopts). [DOCKER_OPTS](#docker_opts) are the command line options provided to the docker engine and [DOCKER_CMDOPTS](#docker_cmdopts) are the options specific to the command being run.

### Option Types

`dockenv` supports 3 option types which align with the [3 option types used by the Docker Engine CLI](https://docs.docker.com/engine/reference/commandline/cli/#/option-types).

#### Boolean Value

The behaviour of boolean options (or flags) in `dockenv` is to add the flag when the associated environment variable is set to `true` or `yes`. For some options, where the Docker default is true, the environment variable has been negated (e.g. rather than providing `DOCKER_SIG_PROXY`, `DOCKER_DISABLE_SIG_PROXY` is provided) and so setting the environment variable to `true` or `yes` will add the flag in the `--flag=false` format.

By requiring an explicit `true` or `yes` value, it allows the option to be blanked out in situations where unsetting the variable is not possible (e.g. systemd units). All other values are considered to be `false` and are accepted without warning.

#### Single Value

For single value options, the value of the environment variable is used as the value for the option. Since environment variable names must be unique, single value environment variables can only specify the a single value for the option. The value provided would be in the same format that would have been used for the flag if given on the command line unless otherwise specified. Variables with empty values are ignored, allowing an environment variable set by a higher environment to be disabled with a blank value.

#### Multi Value

These are set in the same way as [single value options](#single-value) are provided by environment variables that match a pattern, typically of a wildcard format (e.g. `DOCKER_VOLUME_.*`). Since variable names are unique, a specific multi value option can be overridden or disabled by setting the value to an empty value. Naming conventions for these wildcard environments are specified in the comments for the environment variable documentation.

Currently, setting the single value and multi value for an option will result in the highest precedence single value all multi-value options being used, but it is planned that setting the single value, will disable all multi value options provided and provide a warning. Single value options are typically provided for all multi value options which have a simple value (that cannot be split or seen as an assignment).

### Precedence

The precedence for options specified via the various means are as outlined here:

1. Options offered through `DOCKER_OPTS` and `DOCKER_CMDOPTS`. If either of these environment variables is set, they are used as the initial value of the lists internally which are appended to by specific options and so will be overridden if a matching option is specified by any other means. It is typically not recommended to set these variables explicitly and to use the explicit environment variables provided, however they are provided for any options which may not be covered by the current implementation of `dockenv`: if you do find a flag that is not supported, please open an issue or add the option and create a PR.
1. Options provided by explicit environment variables. The available environment variables are outlined in the [DOCKER_OPTS](#docker_opts) and [DOCKER_CMDOPTS](#docker_cmdopts) sections. These options are appended to the `DOCKER_OPTS` and `DOCKER_CMDOPTS` options as appropriate and so take precedence over any options specified by either of those. Flags (or [Boolean Options](#boolean-value)) are the one exception to this as if they are not specified or specified as anything that is not `true` or `yes`, nothing will be added to and so it is currently not possible to set a flag to false that has been specified true by a method of lower precedence.
1. Options specified on the command line take the highest precedence and will override anything specified by other means. The command line provided options are always appended last to the `DOCKER_OPTS` and `DOCKER_CMDOPTS` variables and so will override any value of lower precedence.

One notable case is for multi value options; for these options the precedence defines the order of the multiple values being given and previously declared values will not be overridden unless the value makes docker perform overriding (e.g. specifying two volume definitions for the same location within the container).

### DOCKER_OPTS

`DOCKER_OPTS` are the options passed to the Docker Engine (before any particular command). If set, the `DOCKER_OPTS` environment variable should be a list of [command line flags supported by the Docker Engine](https://docs.docker.com/engine/reference/commandline/cli/). Typically these define configuration for docker and connecting to the Docker Daemon. Since the Engine supports most of the flags from [environment variables natively](https://docs.docker.com/engine/reference/commandline/cli/#/environment-variables), we only provide custom environment variables for those which are not made available through the native environment variable support:

Environment Variable | Associated flag(s) | Comment
-------------------- | ------------------ | -------
`DOCKER_DEBUG`       | `-D`,`--debug`     |
`DOCKER_LOG_LEVEL`   | `-l`,`--log-level` |
`DOCKER_TLS`         | `--tls`            | Implied by `DOCKER_TLS_VERIFY` provided by [Docker Engine's native environment variable support](https://docs.docker.com/engine/reference/commandline/cli/#/environment-variables).
`DOCKER_TLS_CACERT`  | `--tlscacert`      | You may instead consider setting `DOCKER_CERT_PATH` provided by [Docker Engine's native environment variable support](https://docs.docker.com/engine/reference/commandline/cli/#/environment-variables).
`DOCKER_TLS_CERT`    | `--tlscert`        | You may instead consider setting `DOCKER_CERT_PATH` provided by [Docker Engine's native environment variable support](https://docs.docker.com/engine/reference/commandline/cli/#/environment-variables).
`DOCKER_TLS_KEY`     | `--tlskey`         | You may instead consider setting `DOCKER_CERT_PATH` provided by [Docker Engine's native environment variable support](https://docs.docker.com/engine/reference/commandline/cli/#/environment-variables).

### DOCKER_CMDOPTS

`DOCKER_CMDOPTS` are the options which are specific to the command being executed: these are inserted after the command and before any positional arguments provided to that command. If set, the `DOCKER_CMDOPTS` environment variable should be a list of options for the specific command being executed via `dockenv`. The full list of options for any command can be seen in the [Docker Engine Command Line Reference](https://docs.docker.com/engine/reference/commandline/) or by using the `--help` flag.

The supported commands are outlined in this section:

- [dockenv run](#dockenv-run)
- [dockenv network create](#dockenv-network-create)

#### dockenv run

Full documentation of the environment variables provided will be added shortly. In the meantime, the source can be inspected to see the options.

#### dockenv network create

Full documentation of the environment variables provided will be added shortly. In the meantime, the source can be inspected to see the options.
