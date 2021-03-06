---
title: Process Management
description: "How to manage the Vector process: starting, stopping, and reloading."
---

This document covers how to manage the Vector process using various interfaces.
How you manage the Vector process is largely dependent on how you installed
Vector.

## Starting

### Command

<Tabs
block={false}
centered={true}
groupId="interfaces"
placeholder="How did you install Vector?"
select={true}
size={null}
values={[{"group":"Package managers","label":"APT","value":"apt"},{"group":"Package managers","label":"DPKG","value":"dpkg"},{"group":"Platforms","label":"Docker CLI","value":"docker-cli"},{"group":"Platforms","label":"Docker Compose","value":"docker-compose"},{"group":"Package managers","label":"Homebrew","value":"homebrew"},{"group":"Package managers","label":"MSI","value":"msi"},{"group":"Package managers","label":"Nix","value":"nix"},{"group":"Package managers","label":"RPM","value":"rpm"},{"group":"Nones","label":"Vector CLI","value":"vector-cli"},{"group":"Package managers","label":"Yum","value":"yum"}]}>
<TabItem value="apt">

```bash
sudo systemctl start vector
```

</TabItem>
<TabItem value="dpkg">

```bash
sudo systemctl start vector
```

</TabItem>
<TabItem value="docker-cli">

```bash
docker run \
  -v $PWD/vector.toml:/etc/vector/vector.toml:ro \
  timberio/vector:latest-alpine
```

<CodeExplanation>

- The `-v $PWD/vector.to...` flag passes your custom configuration to Vector.
- The `timberio/vector:latest-alpine` is the default image we've chosen, you are welcome to use [other image variants][docs.platforms.docker#variants].

</CodeExplanation>

</TabItem>
<TabItem value="docker-compose">

</TabItem>
<TabItem value="homebrew">

```bash
brew services start vector
```

</TabItem>
<TabItem value="msi">

```bat
.\bin\vector --config config\vector.toml
```

</TabItem>
<TabItem value="nix">

```bash
vector --config /etc/vector/vector.toml
```

<CodeExplanation>

- `vector` is placed in your `$PATH`.
- You must create a [Vector configuration file][docs.setup.configuration] to
  successfully start Vector.

</CodeExplanation>

</TabItem>
<TabItem value="rpm">

```bash
sudo systemctl start vector
```

</TabItem>
<TabItem value="vector-cli">

```bash
vector --config vector.toml
```

</TabItem>
<TabItem value="yum">

```bash
sudo systemctl start vector
```

</TabItem>
</Tabs>

### Flags

| Flag                    | Description                                                                                                         |     |
| :---------------------- | :------------------------------------------------------------------------------------------------------------------ | :-- |
| **Required**            |                                                                                                                     |     |
| `-c, --config <path>`   | Path the Vector [configuration file][docs.setup.configuration].                                                           |     |
| **Optional**            |                                                                                                                     |     |
| `-q, --quiet`           | Raises the log level to `warn`.                                                                                     |     |
| `-qq`                   | Raises the log level to `error`.                                                                                    |     |
| `-qqq`                  | Turns logging off.                                                                                                  |     |
| `-r, --require-healthy` | Causes vector to immediately exit if any sinks fail their healthchecks.                                             |     |
| `-t, --threads`         | Limits the number of internal threads Vector can spawn.                                                             |     |
| `-v, --verbose`         | Drops the log level to `debug`.                                                                                     |     |
| `-vv`                   | Drops the log level to `trace`, the lowest level possible.                                                          |     |
| `-w, --watch-config`    | Vector will watch for changes in [configuration file][docs.setup.configuration], and reload accordingly. (Mac/Linux only) |     |

### Daemonizing

Vector does not _directly_ offer a way to daemonize the Vector process. We
highly recommend that you use a utility like [Systemd][urls.systemd] to
daemonize and manage your processes. Vector provides a
[`vector.service` file][urls.vector_systemd_file] for Systemd.

## Stopping

The Vector process can be stopped by sending it a `SIGTERM` process signal.

### Command

<Tabs
block={false}
centered={true}
groupId="interfaces"
placeholder="How did you install Vector?"
select={true}
size={null}
values={[{"group":"Package managers","label":"APT","value":"apt"},{"group":"Package managers","label":"DPKG","value":"dpkg"},{"group":"Platforms","label":"Docker CLI","value":"docker-cli"},{"group":"Platforms","label":"Docker Compose","value":"docker-compose"},{"group":"Package managers","label":"Homebrew","value":"homebrew"},{"group":"Package managers","label":"MSI","value":"msi"},{"group":"Package managers","label":"Nix","value":"nix"},{"group":"Package managers","label":"RPM","value":"rpm"},{"group":"Nones","label":"Vector CLI","value":"vector-cli"},{"group":"Package managers","label":"Yum","value":"yum"}]}>
<TabItem value="apt">

```bash
sudo systemctl stop vector
```

</TabItem>
<TabItem value="dpkg">

```bash
sudo systemctl stop vector
```

</TabItem>
<TabItem value="docker-cli">

```bash
docker stop timberio/vector
```

</TabItem>
<TabItem value="docker-compose">

</TabItem>
<TabItem value="homebrew">

```bash
brew services stop vector
```

</TabItem>
<TabItem value="msi">

The Vector MSI package does not install Vector into a process manager.
Therefore, you are responsible for stopping Vector based on how you started it.

</TabItem>
<TabItem value="nix">

The Vector Nix package does not install Vector into a process manager.
Therefore, you are responsible for stopping Vector based on how you started it.

</TabItem>
<TabItem value="rpm">

```bash
sudo systemctl stop vector
```

</TabItem>
<TabItem value="vector-cli">

If you are starting Vector directly from the Vector CLI then you are responsible
for stopping Vector depending on how you are managing the process. If you're
in the terminal, hitting `ctrl+c` will exit the process.

</TabItem>
<TabItem value="yum">

```bash
sudo systemctl stop vector
```

</TabItem>
</Tabs>

### Graceful Shutdown

Vector is designed to gracefully shutdown within 20 seconds when a `SIGTERM`
process signal is received. The shutdown process is as follows:

1. Stop accepting new data for all [sources][docs.sources].
2. Gracefully close any open connections with a 20 second timeout.
3. Flush any sink buffers with a 20 second timeout.
4. Exit the process with a `1` exit code.

### Force Killing

If Vector is forcefully killed there is the potential to lose in-flight
data. To mitigate this we recommend enabling on-disk buffers and avoiding
forceful shutdowns whenever possible.

### Exit Codes

If Vector fails to start it will exit with one of the preferred exit codes
as defined by `sysexits.h`. A full list of exit codes can be found in the
[`exitcodes` Rust crate][urls.exit_codes]. The relevant codes that Vector uses
are:

| Code | Description                              |
| :--- | :--------------------------------------- |
| `0`  | No error.                                |
| `78` | Bad [configuration][docs.setup.configuration]. |

## Reloading

Vector can be reloaded, on the fly, to recognize any configuration changes by
sending the Vector process a `SIGHUP` signal.

### Command

<Tabs
block={false}
centered={true}
groupId="interfaces"
placeholder="How did you install Vector?"
select={true}
size={null}
values={[{"group":"Package managers","label":"APT","value":"apt"},{"group":"Package managers","label":"DPKG","value":"dpkg"},{"group":"Platforms","label":"Docker CLI","value":"docker-cli"},{"group":"Platforms","label":"Docker Compose","value":"docker-compose"},{"group":"Package managers","label":"Homebrew","value":"homebrew"},{"group":"Package managers","label":"MSI","value":"msi"},{"group":"Package managers","label":"Nix","value":"nix"},{"group":"Package managers","label":"RPM","value":"rpm"},{"group":"Nones","label":"Vector CLI","value":"vector-cli"},{"group":"Package managers","label":"Yum","value":"yum"}]}>
<TabItem value="apt">

```bash
sudo systemctl stop vector
```

</TabItem>
<TabItem value="dpkg">

```bash
sudo systemctl stop vector
```

</TabItem>
<TabItem value="docker-cli">

```bash
docker stop timberio/vector
```

</TabItem>
<TabItem value="docker-compose">

</TabItem>
<TabItem value="homebrew">

```bash
brew services stop vector
```

</TabItem>
<TabItem value="msi">

The Vector MSI package does not install Vector into a process manager.
Therefore, you are responsible for stopping Vector based on how you started it.

</TabItem>
<TabItem value="nix">

The Vector Nix package does not install Vector into a process manager.
Therefore, you are responsible for stopping Vector based on how you started it.

</TabItem>
<TabItem value="rpm">

```bash
sudo systemctl stop vector
```

</TabItem>
<TabItem value="vector-cli">

If you are starting Vector directly from the Vector CLI then you are responsible
for stopping Vector depending on how you are managing the process. If you're
in the terminal, hitting `ctrl+c` will exit the process.

</TabItem>
<TabItem value="yum">

```bash
sudo systemctl stop vector
```

</TabItem>
</Tabs>

### Automatic Reload On Changes

You can automatically reload Vector's configuration file when it changes by using
the `-w` or `--watch-config` flag when [starting](#starting) Vector. This is
particularly helpful in scenarios where configuration is managed for you, such
as Kubernetes.

### Configuration Errors

When Vector is reloaded it proceeds to read the new configuration file from
disk. If the file has errors it will be logged to `STDOUT` and ignored,
preserving any previous configuration that was set. If the process exits you
will not be able to restart the process since it will try to use the
new, invalid, configuration file.

### Graceful Pipeline Transitioning

Vector will perform a diff between the new and old configuration, determining
which sinks and sources should be started and shutdown and ensures the
transition from the old to new pipeline is graceful.

[docs.setup.configuration]: /docs/setup/configuration/
[docs.platforms.docker#variants]: /docs/setup/installation/platforms/docker/#variants
[docs.sources]: /docs/reference/sources/
[urls.exit_codes]: https://docs.rs/exitcode/1.1.2/exitcode/#constants
[urls.systemd]: https://systemd.io/
[urls.vector_systemd_file]: https://github.com/timberio/vector/blob/master/distribution/systemd/vector.service
