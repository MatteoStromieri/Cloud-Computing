# Introduction to Kubernetes
Questa è la lezione di cozzini, scrivi i tuoi appunti o studia dalle slide
# Introduction to Containers
## Linux Namespaces
**Links:**  
- [namespaces - Linux manual](https://www.man7.org/linux/man-pages/man7/namespaces.7.html)
- [Namespaces in operation: namespaces overview](https://lwn.net/Articles/531114/)
### Introduction
The purpose of a namespace is to wrap a particular global system resource in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource.
There are different types of namespaces:
- Cgroup
- IPC (Inter Process Communication)
- Network
- Mount
- PID
- Time
- User
- UTS
### Most useful syscalls for namespaces
- [*clone()*](https://www.man7.org/linux/man-pages/man2/clone.2.html): creates a new process. You can set some flags to create some new namespaces. The child process is made a member of those namespaces.
- [setns()](https://www.man7.org/linux/man-pages/man2/setns.2.html): allows the calling process to join an existing namespace.  The namespace to join is specified via a file descriptor that refers to one of the _/proc/_pid_/ns_ files described below.
- [unshare()](https://www.man7.org/linux/man-pages/man2/unshare.2.html): moves the calling process to a new namespace. If you set some flags new namespaces are created for each flag, and the calling process is made a member of those namespaces.
### **The** _/proc/_**pid**_/ns/_ directory
Each process has a _/proc/_pid_/ns/_ subdirectory containing one entry for each namespace.
```
$ ls -l /proc/$$/ns | awk '{print $1, $9, $10, $11}'
total 0
lrwxrwxrwx. cgroup -> cgroup:[4026531835]
lrwxrwxrwx. ipc -> ipc:[4026531839]
lrwxrwxrwx. mnt -> mnt:[4026531840]
lrwxrwxrwx. net -> net:[4026531969]
lrwxrwxrwx. pid -> pid:[4026531836]
lrwxrwxrwx. pid_for_children -> pid:[4026531834]
lrwxrwxrwx. time -> time:[4026531834]
lrwxrwxrwx. time_for_children -> time:[4026531834]
lrwxrwxrwx. user -> user:[4026531837]
lrwxrwxrwx. uts -> uts:[4026531838]
```
In this output you see that the numbers inside the brackets are *i-numbers*.

> [!NOTE]  
> Bind mounting (see [mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html)) one of the files in this directory to somewhere else in the filesystem keeps the corresponding namespace of the process specified by _pid_ alive even if all processes currently in the namespace terminate.  
>
> A _bind mount_ is an alternate view of a directory tree. Classically, mounting creates a view of a storage device as a directory tree. A bind mount instead takes an existing directory tree and replicates it under a different point. The directories and files in the bind mount are the same as the original. Any modification on one side is immediately reflected on the other side, since the two views show the same data.

 As long as the file descriptor of a namespace remains open, the namespace will remain alive, even if all processes in the namespace terminate.  The file descriptor can be passed to [setns(2)](https://www.man7.org/linux/man-pages/man2/setns.2.html).
### The _/proc/sys/user_ directory
The files in the _/proc/sys/user_ directory expose limits on the number of namespaces of various types that can be created.
- The limits are per-user.  Each user in the same user namespace can create namespaces up to the defined limit.
- The limits apply to all users, including UID 0.
- Each user namespace has a creator UID.
- When a namespace is created, it is accounted against the creator UIDs in each of the ancestor user namespaces, and the kernel ensures that the corresponding namespace limit for the creator UID in the ancestor namespace is not exceeded.
	- The aforementioned point ensures that creating a new user namespace cannot be used as a means to escape the limits in force in the current user namespace.

Tutti i file hanno il nome _max_ns_namespaces_ dove *ns* può essere sostituito con qualsiasi tipo di namespace. Per esempio _max_cgroup_namespaces_.
### Namespace lifetime
Absent any other factors, a namespace is automatically torn down when the last process in the namespace terminates or leaves the namespace.
However, there are a lot of factors that may pin a namespace into existence even though it has no member processes, now we list some of them:
- An open file descriptor or a bind mount exists for the corresponding _/proc/_pid_/ns/*_ file.
- The namespace is hierarchical (i.e., a PID or user namespace), and has a child namespace.
- It is a user namespace that owns one or more nonuser namespaces.
- ...
### Capabilities
**Link:** [capabilities(7) - Linux manual](https://www.man7.org/linux/man-pages/man7/capabilities.7.html)
For the purpose of performing permission checks, traditional UNIX implementations distinguish two categories of processes: _privileged_ processes (whose effective user ID is 0, referred to as superuser or root), and _unprivileged_ processes (whose effective UID is nonzero).
Starting with Linux 2.2, Linux divides the privileges traditionally associated with superuser into distinct units, known as _capabilities_, which can be independently enabled and disabled.

>[!NOTE]
>Capabilities are per-thread attributes. You can check them by looking at the */proc/[PID]/task/[LWP]/status* file. The *task* directory lists the threads for that process.

Linux defines more than 20 capabilities.
#### Threads capabilities
Each thread has the following capability sets containing zero or more of the above capabilities:
- ***Permitted***: This is a limiting superset for the effective capabilities that the thread may assume.
- ***Inheritable***:  set of capabilities preserved across an [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html). inheritable capabilities are not generally preserved across [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html) when running as a non-root user, applications that wish to run helper programs with elevated capabilities should consider using ambient capabilities, described below.
- ***Effective***: This is the set of capabilities used by the kernel to perform permission checks for the thread.
- ***Bounding***: is a mechanism that can be used to limit the capabilities that are gained during [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html).
- ***Ambient***: This is a set of capabilities that are preserved across an [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html)of a program that is not privileged.  The ambient capability set obeys the invariant that no capability can ever be ambient if it is not both permitted and inheritable.

>[!OBSERVATION]
>A child created via [fork(2)](https://www.man7.org/linux/man-pages/man2/fork.2.html) inherits copies of its parent's capability sets.
#### File capabilities
The file capability sets, in conjunction with the capability sets of the thread, determine the capabilities of a thread after an [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html).
The three file capability sets are:
- ***Permitted***: These capabilities are automatically permitted to the thread, regardless of the thread's inheritable capabilities.
- ***Inheritable***:This set is ANDed with the thread's
- ***Effective***: This is not a set, but rather just a single bit.  If this bit is set, then during an [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html) all of the new permitted capabilities for the thread are also raised in the effective set.  If this bit is not set, then after an [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html), none of the new permitted capabilities is in the new effective set.
#### Transformation of capabilities during execve()
During an [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html), the kernel calculates the new capabilities of the process using the following algorithm:
```
P'(ambient) = (file is privileged) ? 0 : P(ambient)
P'(permitted)   = (P(inheritable) & F(inheritable)) |
                  (F(permitted) & P(bounding)) | P'(ambient)
P'(effective)   = F(effective) ? P'(permitted) : P'(ambient)
P'(inheritable) = P(inheritable)    [i.e., unchanged]
P'(bounding)    = P(bounding)       [i.e., unchanged]
```
where 
- *P()* denotes the value of a thread capability set before the [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html)
- *P'()* denotes the value of a thread capability set after the [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html)
- *F()* denotes a file capability set
#### The securebits flags
Linux implements a set of per-thread _securebits_ flags that can be used to disable special handling of capabilities for UID 0 (_root_).
- **SECBIT_KEEP_CAPS**: Setting this flag allows a thread that has one or more 0 UIDs to retain capabilities in its permitted set when it switches all of its UIDs to nonzero values.  If this flag is not set, then such a UID switch causes the thread to lose all permitted capabilities.
- **SECBIT_NO_SETUID_FIXUP**: Setting this flag stops the kernel from adjusting the process's permitted, effective, and ambient capability sets when the thread's effective and filesystem UIDs are switched between zero and nonzero values. 
- **SECBIT_NOROOT**: If this bit is set, then the kernel does not grant capabilities when a set-user-ID-root program is executed, or when a process with an effective or real UID of 0 calls [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html).
- **SECBIT_NO_CAP_AMBIENT_RAISE**: Setting this flag disallows raising ambient capabilities via the [prctl(2)](https://www.man7.org/linux/man-pages/man2/prctl.2.html) **PR_CAP_AMBIENT_RAISE** operation.
Each of the above "base" flags has a companion "locked" flag. Setting any of the "locked" flags is irreversible, and has the effect of preventing further changes to the corresponding "base" flag.

The _securebits_ flags are inherited by child processes.  During are [execve(2)](https://www.man7.org/linux/man-pages/man2/execve.2.html), all of the flags are preserved, except **SECBIT_KEEP_CAPS** which is always cleared.
### The User Namespace
User namespaces isolate security-related identifiers and attributes, in particular, user IDs and group IDs, the root directory, keys and capabilities. A process's user and group IDs can be different inside and outside a user namespace.  In particular, a process can have a normal unprivileged user ID outside a user namespace while at the same time having a user ID of 0 inside the namespace;
#### Nested namespaces
Each user namespace except the initial ("root") namespace has a parent user namespace, and can have zero or more child user namespaces. The
parent user namespace is the user namespace of the process that creates the user namespace via a call to [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html) or [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) with the **CLONE_NEWUSER** flag.
- Each process is a member of exactly one user namespace.  
- A process created via [fork(2)](https://www.man7.org/linux/man-pages/man2/fork.2.html) or [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) without the **CLONE_NEWUSER** flag is a member of the same user namespace as its parent.
#### Capabilities
The child process created by [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) with the **CLONE_NEWUSER** flagstarts out with a complete set of capabilities in the new user namespace. ikewise, a process that creates a new user namespace using [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html) or joins an existing user namespace using [setns(2)](https://www.man7.org/linux/man-pages/man2/setns.2.html) gains a full set of capabilities in that namespace.

That process has no capabilities in the parent (in the case of [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html)) or previous (in the case of [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html) and [setns(2)](https://www.man7.org/linux/man-pages/man2/setns.2.html)) user namespace, even if the new namespace is created or joined by the root user (i.e., a process with user ID 0 in the root namespace).

A call to [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) or [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html) using the **CLONE_NEWUSER** flag ora call to [setns(2)](https://www.man7.org/linux/man-pages/man2/setns.2.html) that moves the caller into another user namespace sets the "securebits" flags (see [capabilities(7)](https://www.man7.org/linux/man-pages/man7/capabilities.7.html)) to their default values (all flags disabled) in the child (for [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html)) or caller (for [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html) or [setns(2)](https://www.man7.org/linux/man-pages/man2/setns.2.html)).  Note that because the caller no longer has capabilities in its original user namespace after a call to [setns(2)](https://www.man7.org/linux/man-pages/man2/setns.2.html), it is not possible for a process to reset its "securebits" flags while retaining its user namespace membership by using a pair of [setns(2)](https://www.man7.org/linux/man-pages/man2/setns.2.html) calls to move to another user namespace and then return to its original user namespace.

The rules for determining whether or not a process has a capability in a particular user namespace are as follows:
- A process has a capability inside a user namespace if it is a member of that namespace and it has the capability in its effective capability set.
- If a process has a capability in a user namespace, then it has that capability in all child (and further removed descendant) namespaces as well.
- When a user namespace is created, the kernel records the effective user ID of the creating process as being the "owner" of the namespace.  A process that resides in the parent of the user namespace and whose effective user ID matches the owner of the namespace has all capabilities in the namespace.

Having a capability inside a user namespace permits a process to perform operations (that require privilege) only on resources governed by that namespace. In other words, having a capability in a user namespace permits a process to perform privileged operations on resources that are governed by (nonuser) namespaces owned by (associated with) the user namespace. On the other hand, there are many privileged operations that affect resources that are not associated with any namespace type, only a process with privileges in the _initial_ user namespace can perform such operations.

Holding **CAP_SYS_ADMIN** within the user namespace that owns a process's mount namespace allows that process to create bind mounts and mount the following types of filesystems:
-  _/proc_ (since Linux 3.8)
- _/sys_ (since Linux 3.8)
- _devpts_ (since Linux 3.9)
- [tmpfs(5)](https://www.man7.org/linux/man-pages/man5/tmpfs.5.html) (since Linux 3.9)
- _ramfs_ (since Linux 3.9)
- _mqueue_ (since Linux 3.9)
- _bpf_ (since Linux 4.4)
- _overlayfs_ (since Linux 5.11)

Holding **CAP_SYS_ADMIN** within the user namespace that owns a process's cgroup namespace allows (since Linux 4.6) that process to the mount the cgroup version 2 filesystem and cgroup version 1 named hierarchies.

Holding **CAP_SYS_ADMIN** within the user namespace that owns a process's PID namespace allows (since Linux 3.8) that process to mount _/proc_ filesystems.

Mounting block-based filesystems can be done only by a process that holds **CAP_SYS_ADMIN** in the initial user namespace.
#### Interaction of user namespaces and other types of namespaces
Unprivileged processes can create user namespaces, and the other types of namespaces can be created with just the **CAP_SYS_ADMIN** capability in the caller's user namespace.

When a nonuser namespace is created, it is owned by the user namespace in which the creating process was a member at the time of the creation of the namespace. Privileged operations on resources governed by the nonuser namespace require that the process has the necessary capabilities in the user namespace that owns the nonuser namespace.

hen a new namespace (other than a user namespace) is created via [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) or [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html), the kernel records the user namespace of the creating process as the owner of the new namespace.  (This association can't be changed.) When a process in the new namespace subsequently performs privileged operations that operate on global resources isolated by the namespace, the permission checks are performed according to the process's capabilities in the user namespace that the kernel associated with the new namespace.
#### User and group ID mappings: uid_map and gid_map
When a user namespace is created, it starts out **without** a mapping of user IDs (group IDs) to the parent user namespace. The _/proc/_pid_/uid_map_ and _/proc/_pid_/gid_map_ files expose the mappings for user and group IDs inside the user namespace for the process _pid_.

The description in the following paragraphs explains the details for _uid_map_; _gid_map_ is exactly the same.

Each line in the _uid_map_ file specifies a 1-to-1 mapping of a range of contiguous user IDs between two user namespaces.  (When a user namespace is first created, this file is empty.)  The specification in each line takes the form of three numbers delimited by white space.  
1. The start of the range of user IDs in the user namespace of the process _pid_.
2.  The start of the range of user IDs to which the user IDs specified by field one map.
3. The length of the range of user IDs that is mapped between the two user namespaces.

When a process accesses a file, its user and group IDs are mapped into the initial user namespace for the purpose of permission checking and assigning IDs when creating a file.  When a process retrieves file user and group IDs, the IDs are mapped in the opposite direction, to produce values relative to the process user and group ID mappings.

After the creation of a new user namespace, the _uid_map_ file of _one_ of the processes in the namespace may be written to _once_ to define the mapping of user IDs in the new user namespace.  An attempt to write more than once to a _uid_map_ file in a user namespace fails with the error **EPERM**.  Similar rules apply for _gid_map_ files.

In order for a process to write to the _/proc/_pid_/uid_map_ (_/proc/_pid_/gid_map_) file, all of the following permission requirements must be met:
- The writing process must have the **CAP_SETUID** (**CAP_SETGID**) capability in the user namespace of the process _pid_.
- The writing process must either be in the user namespace of the process _pid_ or be in the parent user namespace of the process _pid_.
- The mapped user IDs (group IDs) must in turn have a mapping in the parent user namespace.

### The Mount Namespace
**Links:**[mount_namespaces(7) - Linux manual](https://www.man7.org/linux/man-pages/man7/mount_namespaces.7.html)
Mount namespaces provide isolation of the list of mounts seen by the processes in each namespace instance. Thus, the processes in
each of the mount namespace instances will see distinct single-directory hierarchies.

The views provided by the _/proc/_pid_/mounts_, _/proc/_pid_/mountinfo_, and _/proc/_pid_/mountstats_ files (all described in [proc(5)](https://www.man7.org/linux/man-pages/man5/proc.5.html)) correspond to the mount namespace in which the process with the PID _pid_ resides.

A new mount namespace is created using either [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) or [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html) with the **CLONE_NEWNS** flag.  When a new mount namespace is created, its mount list is initialized as follows:
- If the namespace is created using [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html), the mount list of the child's namespace is a copy of the mount list in the parent process's mount namespace.
- If the namespace is created using [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html), the mount list of the new namespace is a copy of the mount list in the caller's previous mount namespace.
#### Shared Subtrees
This feature allows for automatic, controlled propagation of [mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html) and [umount(2)](https://www.man7.org/linux/man-pages/man2/umount.2.html) _events_ between namespaces (or, more precisely, between the mounts that are members of a _peer group_ that are propagating events to one another).

Each mount is marked (via [mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html)) as having one of the following _propagation types_:
- **MS_SHARED**: [mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html) and [umount(2)](https://www.man7.org/linux/man-pages/man2/umount.2.html) events immediately under this mount will propagate to the other mounts that are members of the peer group.  _Propagation_ here means that the same [mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html) or [umount(2)](https://www.man7.org/linux/man-pages/man2/umount.2.html) will automatically occur under all of the other mounts in the peer group.
- **MS_PRIVATE**: This mount is private; it does not have a peer group.[mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html) and [umount(2)](https://www.man7.org/linux/man-pages/man2/umount.2.html) events do not propagate into or out of this mount.
- **MS_SLAVE**: [mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html) and [umount(2)](https://www.man7.org/linux/man-pages/man2/umount.2.html) events propagate into this mount from a (master) shared peer group.  [mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html) and [umount(2)](https://www.man7.org/linux/man-pages/man2/umount.2.html)events under this mount do not propagate to any peer.
- **MS_UNBINDABLE**: This is like a private mount, and in addition this mount can't be bind mounted.  Attempts to bind mount this mount ([mount(2)](https://www.man7.org/linux/man-pages/man2/mount.2.html) with the **MS_BIND** flag) will fail.

The propagation type is a **per-mount-point setting**; some mounts may be marked as shared (with each shared mount being a member of a distinct peer group), while others are private (or slaved or unbindable).

The propagation type of the mounts in a mount namespace can be discovered via the "optional fields" exposed in _/proc/_pid_/mountinfo_. The following tags can appear in the optional fields for a record in that file:
- *shared:X*: This mount is shared in peer group _X_.
- *master:X*: This mount is a slave to shared peer group _X_.
- *propagate_from:X*: This tag will always appear in conjunction with a _master:X_ tag.
- *unbindable*: This is an unbindable mount.
>[!NOTE]
>Each peer group has a unique ID that is automatically generated by the kernel, and all mounts in the same peer group will show the same ID.  (These IDs are assigned starting from the value 1, and may be recycled when a peer group ceases to have any members.)


### The PID Namespace
**Links**: [pid_namespaces(7) - Linux manual](https://www.man7.org/linux/man-pages/man7/pid_namespaces.7.html)
PID namespaces isolate the process ID number space, meaning that processes in different PID namespaces can have the same PID.

PIDs in a new PID namespace start at 1, somewhat like a standalone system, and calls to [fork(2)](https://www.man7.org/linux/man-pages/man2/fork.2.html), [vfork(2)](https://www.man7.org/linux/man-pages/man2/vfork.2.html), or [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) will produce processes with PIDs that are unique within the namespace.
#### The Namespace init process
The first process created in a new namespace (i.e., the process created using [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) with the **CLONE_NEWPID** flag, or the first child created by a process after a call to [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html) using the **CLONE_NEWPID** flag) has the PID 1, and is the "init" process for the namespace (see [init(1)](https://www.man7.org/linux/man-pages/man1/init.1.html)).

If the "init" process of a PID namespace terminates, the kernel terminates all of the processes in the namespace via a **SIGKILL** signal.  This behavior reflects the fact that the "init" process is essential for the correct operation of a PID namespace.

Only signals for which the "init" process has established a signal handler can be sent to the "init" process by other members of the PID namespace.  This restriction applies even to privileged processes, and prevents other members of the PID namespace from accidentally killing the "init" process.

Likewise, a process in an ancestor namespace can—subject to the usual permission checks described in [kill(2)](https://www.man7.org/linux/man-pages/man2/kill.2.html)—send signals to the "init" process of a child PID namespace only if the "init" process has established a handler for that signal.  (Within the handler, the _siginfo_t si_pid_ field described in [sigaction(2)](https://www.man7.org/linux/man-pages/man2/sigaction.2.html) will be zero.)  **SIGKILL** or **SIGSTOP** are treated exceptionally: these signals are forcibly delivered when sent from an ancestor PID namespace.  Neither of these signals can be caught by the "init" process, and so will result in the usual actions associated with those signals (respectively, terminating and stopping the process).
#### Nested PID Namespaces
PID namespaces can be nested: each PID namespace has a parent, except for the initial ("root") PID namespace.  The parent of a PID namespace is the PID namespace of the process that created the namespace using [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) or [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html).

A process is visible to other processes in its PID namespace, and to the processes in each direct ancestor PID namespace going back to the root PID namespace. In this context, "visible" means that one process can be the target of operations by another process using system calls that specify a process ID.

A process has one process ID in each of the layers of the PID namespace hierarchy in which is visible, and walking back though each direct ancestor namespace through to the root PID namespace. System calls that operate on process IDs always operate using the process ID that is visible in the PID namespace of the caller.  A call to [getpid(2)](https://www.man7.org/linux/man-pages/man2/getpid.2.html) always returns the PID associated with the namespace in which the process was created.

Calls to [getppid(2)](https://www.man7.org/linux/man-pages/man2/getppid.2.html) for those processes that have their parents outside the PID namespace return 0.
#### /proc and PID namespaces
A _/proc_ filesystem shows (in the _/proc/_pid directories) only processes visible in the PID namespace of the process that performed the mount, even if the _/proc_ filesystem is viewed from processes in other namespaces.

After creating a new PID namespace, it is useful for the child to change its root directory and mount a new procfs instance at _/proc_ so that tools such as [ps(1)](https://www.man7.org/linux/man-pages/man1/ps.1.html) work correctly.  If a new mount namespace is simultaneously created by including **CLONE_NEWNS** in the _flags_ argument of [clone(2)](https://www.man7.org/linux/man-pages/man2/clone.2.html) or [unshare(2)](https://www.man7.org/linux/man-pages/man2/unshare.2.html), then it isn't necessary to change the root directory: a new procfs instance can be mounted directly over _/proc_.

From a shell, the command to mount _/proc_ is:
```
$ mount -t proc proc /proc
```
>[! /proc/sys/kernel/ns_last_pid]
>This file is virtualized per PID namespace and displays the last PID that was allocated in this PID namespace.  When the next PID is allocated, the kernel will search for the lowest unallocated PID that is greater than this value, and when this file is subsequently read it will show that PID.


### The Network Namespace
**Links:**
- [network_namespaces(7) - Linux manual](https://www.man7.org/linux/man-pages/man7/network_namespaces.7.html#:~:text=Network%20namespaces%20provide%20isolation%20of%20the%20system%20resources,under%20%2Fproc%2Fsys%2Fnet%2C%20port%20numbers%20%28sockets%29%2C%20and%20so%20on.)
- [veth(4) - Linux manual](https://www.man7.org/linux/man-pages/man4/veth.4.html)

Network namespaces provide isolation of the system resources associated with networking. In addition, network namespaces isolate the UNIX domain abstract socket namespace (see [unix(7)](https://www.man7.org/linux/man-pages/man7/unix.7.html))

A physical network device can live in exactly one network namespace.  When a network namespace is freed (i.e., when the last process in the namespace terminates), its physical network devices are moved back to the initial network namespace (not to the namespace of the parent of the process).

A virtual network ([veth(4)](https://www.man7.org/linux/man-pages/man4/veth.4.html)) device pair provides a pipe-like abstraction that can be used to create tunnels between network namespaces, and can be used to create a bridge to a physical network device in another namespace.  When a namespace is freed, the [veth(4)](https://www.man7.org/linux/man-pages/man4/veth.4.html) devices that it contains are destroyed.
#### VETH devices
The **veth** devices are virtual Ethernet devices.  They can act as tunnels between network namespaces to create a bridge to a physical network device in another namespace, but can also be used as standalone network devices.

**veth** devices are always created in interconnected pairs. To create a pair of veth use the command
```
ip link add <p1-name> type veth peer name <p2-name>
```
Packets transmitted on one device in the pair are immediately received on the other device.  When either device is down, the link state of the pair is down.
#### TUN/TAP interfaces
**TUN** and **TAP** are [kernel](https://en.wikipedia.org/wiki/Kernel_(operating_system) "Kernel (operating system)") [virtual network](https://en.wikipedia.org/wiki/Network_virtualization "Network virtualization") devices. Being network devices supported entirely in software, they differ from ordinary network devices which are backed by physical [network adapters](https://en.wikipedia.org/wiki/Network_interface_controller "Network interface controller").

TUN, namely [network TUNnel](https://en.wikipedia.org/wiki/Tunneling_protocol "Tunneling protocol"), simulates a [network layer](https://en.wikipedia.org/wiki/Network_layer "Network layer") device and operates in [layer 3](https://en.wikipedia.org/wiki/OSI_model#Layer_3:_Network_layer "OSI model") carrying [IP](https://en.wikipedia.org/wiki/Internet_Protocol "Internet Protocol") packets. TAP, namely [network TAP](https://en.wikipedia.org/wiki/Network_tap "Network tap"), simulates a [link layer](https://en.wikipedia.org/wiki/Link_layer "Link layer") device and operates in [layer 2](https://en.wikipedia.org/wiki/OSI_model#Layer_2:_Data_link_layer "OSI model") carrying [Ethernet](https://en.wikipedia.org/wiki/Ethernet "Ethernet")frames. TUN is used with [routing](https://en.wikipedia.org/wiki/Routing "Routing"). TAP can be used to create a [user space](https://en.wikipedia.org/wiki/User_space "User space") [network bridge](https://en.wikipedia.org/wiki/Network_bridge "Network bridge"). 

Packets sent by an [operating system](https://en.wikipedia.org/wiki/Operating_system "Operating system") via a TUN/TAP device are delivered to a user space program which attaches itself to the device. A user space program may also pass packets into a TUN/TAP device. In this case the TUN/TAP device delivers (or "injects") these packets to the operating-system [network stack](https://en.wikipedia.org/wiki/Network_stack "Network stack") thus emulating their reception from an external source.
#### Making namespaces communicate
A particularly interesting use case is to place one end of a **veth** pair in one network namespace and the other end in another netwo namespace, thus allowing communication between network namespaces.

To do this, one can provide the **netns** parameter when creating the interfaces:
```
ip link add <p1-name> netns <p1-ns> type veth peer <p2-name> netns <p2-ns>
```
>[!Observation]
>Assign a pair of IP addresses, and you can ping and communicate between the two namespaces.

![[Pasted image 20231207132439.png]]
>[!Bridges]
>A Linux bridge is a kernel module that behaves like a network switch, forwarding packets between interfaces that are connected to it. It's usually used for forwarding packets on routers, on gateways, or between VMs and network namespaces on a host.
>
 It also supports STP, VLAN filter, and multicast snooping.

If the veth interfaces are not connected (state $DOWN$) you can turn them on with the following command on **both sides of the cable**: 
```bash
ip link set dev <interface_name> up
```
##### Practice
Consider a complete set of namespaces
```shell
unshare -UinpmrC -f /bin/bash
```
Consider the following variables defined inside our complete set of namespaces
```shell
namespace1=client 
namespace2=server 
ip_address1="10.10.10.10/24" 
ip_address2="10.10.10.20/24" 
interface1=veth-client 
interface2=veth-server 
command="python3 -m http.server 80"
```
The first thing that we have to do in order to let two namespaces communicate is to mount the **proc** and **run** filesystems:
```shell
mount -t tmpfs /run
mount -t proc proc /proc
```
>[!NOTE]
>The **run** folder contains the data about the resources that all the applications uses during their execution 

Now we can create two sub-network namespaces 
```shell
ip nets add $namespace1
ip nets add $namespace2
```
We could have done this via unshare but, since we are working with several namespaces, this is the best practice. Now there are two files called ```client``` and ```server``` in the ```/run/nets``` folder

Both of these two namespaces only contains the loopback interface.

The next step is to create the VETH interfaces in both the namespaces
```shell
ip link add ptp-$interface1 type veth peer name ptp-$interface2
```
If we run ```ip a``` we get the following output
```
2: ptp-veth-server@ptp-veth-client: mtu 1500 qdisc noop state DOWN    group default qlen 1000 link/ether d6:75:55:cc:c0:76 brd  ff:ff    :ff:ff:ff:ff
3: ptp-veth-client@ptp-veth-server: mtu 1500 qdisc noop state DOWN    group default qlen 1000 link/ether f6:95:31:21:19:ea brd ff:ff:    ff:ff:ff:ff
```
Now let's move these into the two namespaces
```shell
ip link set ptp-$interface1 netns $namespace1
ip link set ptp-$interface2 netns $namespace2
```
Now let's give these interfaces an ip
```shell
ip nets exec $namespace1 ip addr add $ip_address1 dev ptp-$interface1
ip nets exec $namespace2 ip addr add $ip_address2 dev ptp-$interface2
```
Now we want to turn these interfaces up
```
ip -all nets exec ip addr
```
Now the two namespaces can communicate 
#### Giving Internet to the Namespace
**Links**: [How to connect internet in network namespace? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/156847/linux-namespace-how-to-connect-internet-in-network-namespace)

Think of a network namespace as another computer. Think of a veth pair as two Ethernet cards with a crossover cable between them. There are three main ways to connect a network namespace to the Internet, NAT, conventional IP routing, Ethernet bridging.

We have to mask the traffic as if a regular unprivileged user generated it, this masking is what is done by podman. It generates a tap interface in the target networking namespace and intercepts all the output from that interface redirecting it to the regular connections. This masking is what is done by Podman.
##### slirp4nets

Getting access to the internet from a namespace without privileges is possible. However, with a caveat: We must mask the traffic as if a regular unprivileged user generated it.

Podman achieves this using a piece of software called slirp4netns, it generates a tap interface in the target networking namespace and intercepts all the output from that interface redirecting it to the regular connections.

1) find the pid in the target namespace (for example 2646)
2) run the following command
```
slirp4netns --configure --mtu=65520 2646 tap0
```
To understand how slirp4nets works at low level, give a look to this article [slirp4netns — How does it work](https://mcastelino.medium.com/slirp4netns-how-does-it-work-5c0bd31200ce)
### Other Namespaces
#### UTS Namespace
UTS namespaces provide isolation of two system identifiers: the hostname and the NIS domain name. These identifiers are set using sethostname(2) and setdomainname(2), and can be retrieved using uname(2), gethostname(2), and getdomainname(2). Changes made to these identifiers are visible to all other processes in the same UTS namespace, but are not visible to processes in other UTS namespaces.
#### IPC Namespace
IPC namespaces isolate certain IPC resources.
#### Time Namespace
Is to play with time, the only usage I see is to prank people about your uptime.
#### Cgroup Namespace
Add a cgroup fs, very useful when live migrating containers, or having to convince a process that he is pid1 (systemd), or for security.



## Preliminary definitions
- **Container**
	- runtime instantiation of a Container Image
	- standard Linux process typically created through a clone() system call instead of fork() or execvp(). Also, containers are often isolated.
- **Container Image**
	- file which is pulled down from a Registry Server and used locally as a mount point when starting Containers.
- **Container Engine**
	- software that accepts user requests and from the end-user’s perspective runs the container.
	- docker, podman, CRI-O...
- **Container Runtime**
	- a lower level component typically used in a Container Engine but can also be used by hand for testing.
	- The main implementation is called *runc* but other container engines rely on *crun*, which is an alternative implementation
- **Pod**
	- A pod is a logical collection of one or more containers that share the same network namespace, storage, and other specifications.
	- Containers within a pod communicate with each other using localhost, as they share the same network stack.
## crun
Now that we have all the elements it is possible to substitute our unashare call with crun, the OCI runtime, which does all the unshare and mounting steps for us. 

Consider the following script
```bash
mkdir -p rootfs 
bash -c 'podman export $(podman create busybox) | tar -C rootfs -xvf -'
```
The complex command at line two can be decomposed as
1) spawning a bash process which creates a new container ready for running
2) exports the produced filesystem in a *tar* format
3) *tar* then decompress the filesystem into the *rootfs* directory
>[!Observation]
>**podman create *image*** creates a writable container layer over the specified image
>
>**podman export *container*** exports the filesystem of a container and saves it as a tarball on the local machine.

Then, if you run
```bash
crun spec --rootless 
```
- spec: generate a configuration file
- rootless: generates a *config.json* file that is suitable for an unprivileged user 

and then, the following command runs the container *cont1*
```bash
crun run cont1
```
- "run" performs a two-step process: first, create the container and then starts it.

Note that this command performs by itself all the steps that we introduced before
>[!WARNING]
>"cont1" itself is not a JSON file, it is associated with a container specification file that is likely in JSON or another supported format.
>
>If you have the container configuration file (e.g., `cont1-config.json`), and you want to generate the container without running it, you can use the `crun create` command.

So, why did we export the filesystem into the *rootfs* directory? Basically because it will be the root of our container. Give a look to the following configuration file:
```json
{ 
	"ociVersion": "1.0.0", 
	"process": { 
		"args": ["sh"], 
		"env": [ 
			"PATH=/usr/sbin:/usr/bin:/sbin:/bin", 
			], 
		"capabilities": { 
			"effective": [ 
					"CAP_AUDIT_WRITE", "CAP_KILL", "CAP_NET_BIND_SERVICE"
					], 
				}, 
			}, 
	"root": { 
		"path": "rootfs", 
		"readonly": true 
		}, 
	"mounts": [
		 { 
		 "destination": "/proc", 
		 "type": "proc", 
		 "source": "proc" 
		 }, 
	], 
	"linux": { 
		"namespaces": [ 
			{"type": "pid"}, 
			{"type": "network"},
			{"type": "ipc"}, 
			{"type": "uts"}, 
			{"type": "user"}, 
			{"type": "cgroup"}, 
			{"type": "mount"}
		]
	}
}
```
- **args** passes the executable
- **env** the environment variables
- The root folder is mounted as **read-only**
	- If you set the root filesystem inside the container as read-only (`"readonly": true`), processes running inside the container can still write to specific locations within the container, such as mounted volumes or writable directories. However, these changes won't impact the underlying real root filesystem of the host system.
- crun **mounts the proc folder** as done before by hand, and all the Linux **namespaces are generated**

the runtime does not provide any networking other than the loopback interface, but it is possible to inject a **tap interface** as it was done before. This operation is usually done by other components like **podman** itself or **cri-o**.
## Conmon
Conmon is a monitoring program and communication tool between a container manager (like [Podman](https://podman.io/) or [CRI-O](https://cri-o.io/)) and an OCI runtime (like [runc](https://github.com/opencontainers/runc) or [crun](https://github.com/containers/crun)) for a single container.

Upon being launched, conmon (usually) double-forks to daemonize and detach from the parent that launched it. It then launches the runtime as its child. This allows managing processes to die in the foreground, but still be able to watch over and connect to the child process (the container).

Conmon is a shim, a small adapter that abstracts the container engine. It is used by podman and cri-o, and takes care of operations such as logs and container monitoring.
## Podman/CRI-O
- CRI-O is the equivalent of Podman inside Kubernetes.

Podman is pod manager, takes care of managing pods and container lifecycle inside pods. It has an API compatible with docker and offers the same functionality of monitoring logging, and resource isolation. Network is handled in a simple enough manner that can be reproduced with little effort.

It uses conmon for container monitoring and crun as the default runtime. podman also handles image lifecycle, from the build phase to the test and publication phase

cri-o is only a Container Runtime Interface (container engine) Daemon. It does not offer any image lifecycle step other than download. Being a daemon to interact with it, we need an external program like Kubernetes or crictl. crictl has some commands that are very similar to Podman’s ones:
![[Pasted image 20231208125425.png]]
Obviously, the cri-o daemon must be up and running.
>[!Observation]
>cri-o and contaienrd are container runtime daemons that show the same API that can be access thorough a socket. The fact that these APIs are similar allows crictl to work with both.

If you want to check how cri-o is configured in your system you can check the ```/etc/crio/``` directory which contains the crio configuration file, while the ```/etc/cni/``` contains the network configuration file.
## Overlay filesystem
TBD
## Exploring a container image
Consider the following Docker file
```
FROM alpine:latest 
RUN apk add curl 
RUN apk add go 
COPY ./testfile.txt /opt
ENTRYPOINT ["/bin/bash"]
```
After building it with 
```bash 
podman build . -t my-alpine
``` 
we obtain a grand total of 4 images
![[Pasted image 20231207223112.png]]
Some best practices are positioning changes only on top of the chain; in this way, fewer layers are affected by change.
## Podman
Consider the following script to run two containers in a single pod
```bash
podman ps -a --pod 
podman pod create my-pod 
podman ps -a --pod 
```
You can see that your pod contains a container *"podman pause"* running a *catonit* application, this is because a pod can never be empty.

Now we can inject containers inside this pod:
```shell
podman run -it --pod my-pod docker.io/library/alpine:latest 
podman ps -a --pod
```
Default settings:
![[Pasted image 20231207224046.png]]
### Volumes
Volumes are the tools used to share files among containers within the same pod.
- The content of a volume disappears when you delete the pod

Consider the following command
```shell
podman volume inspect <volume_name>
```
It will return you some informations about the input volume; Among these informations there is the volume mount point inside the default file system.
- if you check its content when there is no pod running it is empty
### Intra-pod communication
Beside the VETH strategy that we already introduced, there are several ways to let two containers communicate within the same pod. Suring our course the professor showed us two examples:
-  Since the containers inside a pod shares the loopback interface, they can communicate using the localhost ip 
- using socket files located inside a shared volume
## Kubernetes

>[!Kubernetes Documentation]
>**Configuration files** - Written in YAML or JSON - these files describe the desired state of your application in terms of Kubernetes API objects. A file can include one or more API **object descriptions (manifests)**.

In a ```yaml```file that describes an object of Kubernetes (manifest) there are some fields that can not be absent:
- **API version**
- **Kind**: kind of object that your are going to describe with this manifest file
	- The types of resources is provided by the **Kubernetes API** 
	- A Pod is a type of resource
- **Metadata**: descriptive information about the object, there must be at least the ```name``` key which must be unique
	- Beside the ```name```, you can add an arbitrary amount of ```labels```
- **Spec**: describes the desired state of the cluster relatively to this specific resource
	- The fields that this section includes depend on the type of resource

When the your object instances are running, a **status** field is added to the manifest file. This field includes informations about the status of the system with respect to our resources and is managed by the Kubernetes system.
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: my-pod
spec:
	containers:
	- name: server
	  image: python:3.11.6-alpine
	  command: ["sh","-c","python /opt/server.py"]
	  volumeMounts:
	  - mountPath: /opt/
	    name: py-sources
	  - moutPath: /workdir/
	    name: data-sharing 
	- name: client
      image: python:3.11.6-alpine
	  command: ["sh","-c","python /opt/client.py"]
	  volumeMounts:
	  - mountPath: /opt/
	    name: py-sources
	  - moutPath: /workdir/
	    name: data-sharing

	volumes:
	- name: data-sharing
	  emptyDir: {}
	- hostPath:
		  path: /home/mstromieri/uni/msc/cloud_advanced/pod_communication
		  type: Directory
	  name: py-sources
	  ```
To run a manifest on Kubernetes we can run the following command
```bash
$ podman kube play <manifest>.yaml
```
The standard output of Kubernets pods gets redirected to logs. You can visualize it by using the following command:
```bash
$ podman logs
```
>[!NOTE]
>Everytime that the program in your container finishes the execution, your container dies. If you don't want this you have to add an infinite loop inside the code. 
>
>If you don't do this, the Pod will continuously restart and re-execute the containers that have just finished their execution.

To turn-off the pods related to a certain manifest file
```bash
podman kube down <manifest>.yaml
```
### Kubernetes architecture
What do we want from our cluster?
- **Accessible** by many users
- **Contantly available**:  distributed on several nodes of your architecture
- **Scalable**
- (Load) **Balanced**
![[Pasted image 20231208161526.png]]

>[!Who is the end user?]
>We define the end user as every possible external request to our cluster

#### Overview
**Kubelet** is a regular application (not a container) that has the duty to communicate to the other elements the current state of the node
- If you add another node it only runs Kubelet and the proxy

The **Control plane** is a set of resources that are needed to run Kubernetes. The interface to these resources is the **kube apiserver** which provides a set of REST primitives to let **all clients** interact with them. 

A **scheduler** watches for newly created Pods that have no Node assigned. For every Pod that the scheduler discovers, the scheduler becomes responsible for finding the best Node for that Pod to run on.

Since containers in pods - and pods themselves - can have different requirements, the scheduler filters out any nodes that don't meet a Pod's specific scheduling needs. The scheduler finds feasible Nodes for a Pod and then runs a set of functions to score the feasible Nodes and picks a Node with the highest score among the feasible ones to run the Pod. The scheduler then notifies the API server about this decision in a process called _binding_.

The **Control Manager** supervises the status of the cluster and steer the current one toward the desired one

**etcd** is an object storage of key-value pairs (NoSQL DBMS) for persisting cluster state. It stores objects and config information

>[!WARNING]
>All these elements run inside different containers except for the Kubelet one, which is a regular application

#### Node architecture
The following image depicts the default Kubernetes' node architecture
![[Pasted image 20231208163430.png]]
**kubelet** manages the lifecycle of every pod on a certain node and doesn't run inside a container.

**kube-proxy** manages the network rules on each node and performs connection forwarding or load balancing for Kubernetes cluster services.

Lastly, we have a **Container Runtime Engine** which is the application that manages and executes the containers. Containerd (docker) and CRI-O are Container Runtime Engines.
