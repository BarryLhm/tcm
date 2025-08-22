# tcm

Termux Container Manager

Provide container management functionality on a **MINIMUM BASIS**
on an android kernel which usually lacks of namespaces support

### Functions

- mount namespace separation to avoid mount pollution
- cgroup management to easily kill all processes in the container

### Terminology

- `root`: geteuid() == 0
- states (booleans, not exclusive)
`started`: the container's unique cgroup was created
`running`: there's program(s) running in the container cgroup
(and mount namespace is kept until all processes exited)

### Requirements

- `root` permission with selinux bypass (of course)
- mount namespace (Android needs this so ought to be available)
- cgroup (either v1 or v2) mounted on /sys/fs/cgroup
 (this is also needed by Android system)

### Warnings & Notices

- **NO** user namespace support !!!
This means you **CANNOT** have isolated `root` permission in containers,
`root` in containers **FULLY** equals to `root` in host system with selinux etc. escalation
Use at your **OWN RISK** !!!

- **NO** pid namespace support !!
This means processes in containers (espacially those running as `root`) can interact with processes on your host system (including **KILL** and debugging),
init and service managers are mostly **UNSUPPORTED** (and may have **MAJOR INTERFERENCE** with the host system which may need a reboot to recover)

### Usage (may not up to date, read the code!)
- `spawn` CONTAINER_NAME [PROG [ARGS ...]]
put a container in `started` and `running` state
by separating mount namespace, mount filesystems, creating cgroup, attaching the script itself to it, and executing specified program
when PROG is not specified, print environment variables (as `env`'s behavior)
it's currently not allowed to spawn a CONTAINER when it's in `started` state
(an `attach` script will be added later )

- `kill` CONTAINER_NAME
kick a container out of `running` and then `started` state
by using cgroup kill controller and removing the cgroup
