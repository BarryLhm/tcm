# tcm

Termux Container Manager

Provide container management functionality on a **MINIMUM BASIS**
on an android kernel which usually lacks of namespaces support

### Functions

- mount namespace separation to avoid mount pollution
- cgroup management to easily kill all processes in the container

### Terminologies & Concepts

- `root`: geteuid() == 0

States: (booleans, not exclusive)
- `running`: there's program(s) running in the container cgroup
(and mount namespace is kept until all processes exited)

### Requirements

- `root` permission with selinux bypass (of course)
- mount namespace (Android needs this so ought to be available)
- cgroup (either v1 or v2) mounted on /sys/fs/cgroup
 (this is also needed by Android system)
- `busybox` with standalone mode support
 (e.g. busybox from magisk/kernelsu)

### Warnings & Notices

- **NO** user namespace support !!!

This means you **CANNOT** have isolated `root` permission in containers,

`root` in containers **FULLY** equals to `root` in host system with selinux etc. escalation

for enhanced security please install `uid_placeholder.apk`
and add a user with the same uid in the container

Use at your **OWN RISK** !!!

- **NO** pid namespace support !!

This means processes in containers (espacially those running as `root`) can interact with processes on your host system (including **KILL** and debugging),

init and service managers are mostly **UNSUPPORTED** (and may have **SEVERE INTERFERENCE** with the host system which may need a reboot to recover)

### Files & Project Structure

```
/
 |-COMMON: Common parts of the scripts, like a shared library
 |-busybox(not included in repo): busybox binary from magisk/kernelsu etc.
 |-uid_placeholder.apk: An empty apk to create an unprivileged uid
 |-config/: Configuration directory
 |  |-example.conf`: Configuration of container named "example", see Configuration
 |  ...
 |-ns/: namespaces store
 |  |-example.mnt`: mount namespace persistent store of container "example"
 |  ...
 |-switch_root_helper: A simple script to perform pivot_root in container's mnt ns
 |-{run, kill, status}: See "Usage" below
 |-test: A simple wrapper script to test functions in "COMMON"
 ...
```

### Configure
- config/CONTAINER_NAME.conf:
A shell script defining container related variables and functions

Format: (Don't add anything except fields below and follow posix shell syntax!)
(You can certainly do anything **IF YOU HAVE READ THE CODES**, and I am **NOT RESPONSIBLE FOR ANY DAMAGE TO ANYTHING/ANYBODY**)

(Fields with * are mandatory)
```
  container_desc="CONTAINER" # Description
* container_root"/path/to/rootfs" # Container rootfs dir, please manually install system
  container_mount() # Custom mounts
  {
      fs bind "/host/path" "/target/path/relative/to/container/root"
      ...
  }
```
Changes to mounts will take effect after all processes in container exited
(You can use `kill` to force it)

### Usage (may not up to date, read the code!)

# You need to manually copy `busybox` from /data/adb/{magisk, ksu, ...}/bin/busybox
to the place mentioned above

- `run` CONTAINER_NAME [PROG [ARGS ...]]

put a container in `running` state
by separating mount namespace, mount filesystems,
attaching to the container cgroup, and executing specified program

when PROG is not specified, print environment variables (as `env`'s behavior)

after all programs in the container exited, the `running` state will become 0

- `kill` CONTAINER_NAME
kill all process that belong to the container using cgroup kill controller

(turn off container's `running` state)

- `status` CONTAINER_NAME
get overall status about the container
