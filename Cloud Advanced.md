[Course's repo](https://github.com/Foundations-of-HPC/Cloud-advanced-2023/tree/main)
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

![Screenshot](images/Pastedimage20231207132439.png)
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
## Docker VS Podman 
Podman is more secure and lightweight than Docker. Docker relies on a daemon running in the background of your system. Whenever you access the Docker CLI or API to run and manage containers, you are, in effect, communicating with that daemon. Podman is daemonless! If you execute a command with the Podman CLI, it will execute those commands and run the containers directly on the system. Thus, Podman doesn't rely on a Single Point of Failure, and, equally important, you can run containers rootless. The Docker daemon runs in the background with root privileges. In effect:
1. Podman containers run as a non-root user by default
2. Users can run their own containers, and while doing that, the containers run in a user namespace where they are strictly isolated and not accessible to other users
3. Containers are daemonless and run on top of the lightweight CRI-o container runtime

Note rootless containers do not have an IP address, can only bind to a nonprivileged port and must be the owner of the directory they use for storage.

The following image depicts the main components of Docker an Podman 
![[Pasted image 20231209151718.png]]
While the following image depicts and describes the main components of the Docker architecture
![[Pasted image 20231209151932.png]]
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
**Kubelet** is a regular application (not a pod) that has the duty to communicate to the other elements the current state of the node
- If you add another node it only runs Kubelet and the proxy

The **Control plane** is a set of resources that are needed to run Kubernetes. The interface to these resources is the **kube apiserver** which provides a set of REST primitives to let **all clients** interact with them. 
- We need at least one control plane in the whole cluster
- You want your infrastructure to be consistent $\rightarrow$ you define the control plane on several nodes
	- If you have multiple control planes, they are all synchronized (They only synch the etcd database)

A **scheduler** watches for newly created Pods that have no Node assigned. For every Pod that the scheduler discovers, the scheduler becomes responsible for finding the best Node for that Pod to run on.

Since containers in pods - and pods themselves - can have different requirements, the scheduler filters out any nodes that don't meet a Pod's specific scheduling needs. The scheduler finds feasible Nodes for a Pod and then runs a set of functions to score the feasible Nodes and picks a Node with the highest score among the feasible ones to run the Pod. The scheduler then notifies the API server about this decision in a process called _binding_.

The **Control Manager** is an infinite loop that supervises the status of the cluster (via the API server) and steers the current one toward the desired one

**etcd** is an object storage of key-value pairs (NoSQL DBMS) for persisting cluster state. It stores objects and config information
- stores informations about the current state and the older ones

>[!WARNING]
>All these pods share the same network namespace as pid 1
#### Node architecture
The following image depicts the default Kubernetes' node architecture
![[MSc/Cloud Computing/images/Pasted image 20231208163430 1.png]]

![[Pasted image 20231209152036.png]]

**kubelet** manages the lifecycle of every pod on a certain node and doesn't run inside a container.
- Its backend is a container engine (CRI-O) which will depend on several plugins: like the network plugin (CNI: Container Network Interface)
	- CRI-O uses crun/runc
	- Therefore, you don't need kubelet to interact  with the node, you can directly interact with CRI-O via crictl
- Kubelet is the only application that Kubernetes gives you

**kube-proxy** manages the network rules on each node and performs connection forwarding or load balancing for Kubernetes cluster services.
- deployed as a container and managed by kubernetes
- Present in every node
- defines how informations move between services and pods
- **kproxy** manages networking via *iptables* and *bridges*

Lastly, we have a **Container Runtime Engine** which is the application that manages and executes the containers. Containerd (docker) and CRI-O are Container Runtime Engines.
>[!Networking]
>Every pod that we deploy will have its own network space except for the control plane which will live in the main network namespace 
### Kubernetes installation
To make things easier and to simulate a real cluster we will install Kubernetes on a Vagrant Virtual Machine

**Minimum spec**: You are not allowed to run kubernetes if you don't have at least two cpus and 2GB of RAM for each node.
- Therefore, our virtual machines will satisfy this contraint
#### Vagrant
>[!Vagrant]
>Abstraction layer on top of the virtualization engine. It allows VMs to be ran without caring about the underlying virtualization software. In practice, you can run a Vagran VM regardless of the fact that you are using VirtualBox, VMware or whatever.

All the informations about the Virtual Machine that we are launching are inside a Vagrant (Ruby) file. The following is an example:
```ruby
servers = [
  { :hostname => "k01", :ip => "192.168.133.80" },
  # { :hostname => "k02", :ip => "192.168.133.81" },
  # { :hostname => "k03", :ip => "192.168.133.83" },
]

Vagrant.configure("2") do |config|
  config.vm.box = "fedora/39-cloud-base"

  config.vm.provider :libvirt do |lv|
    # lv.qemu_use_session = true
    lv.memory = 2048
    lv.cpus = 2
  end

  servers.each do |conf|
    config.vm.define conf[:hostname] do |node|
      node.vm.hostname = conf[:hostname]
      node.vm.synced_folder ".", "/vagrant", disabled: true

      node.vm.network :private_network,
                      :libvirt__network_name => 'kub-devel'

      node.vm.provision :shell, 
                        :path => './0_bash_common_provisioning.sh',
                        :args => [ conf[:ip] ]

    end
  end
end
```
Here we are launching three different servers
- for each server you set some parameters

Then you do some basic provisioning by runnign the following bash script
>[!Provisioning]
>The *provisioning* script provides a set of command that each server ran during its setup
```bash
#! /bin/bash

if [[ $(nmcli -t c show  | grep "Wired connection 2" | wc -l) -ne 0 ]]; then
nmcli c del "Wired connection 2";
fi

if [[ $(nmcli -t c show  | grep hpc | wc -l) -ne 0 ]]; then
nmcli c del "hpc-net";
fi

nmcli con add type ethernet \
    con-name hpc-net \
    ifname eth1 \
    ip4 $1/24 \
    gw4 192.168.132.1 \
    ipv4.method manual \
    autoconnect yes;

nmcli con up hpc-net;

echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxToLxTa5zzD6EboxuHsuLcnJ5XK5gUjthGbYRayO7k3GCF0m2IxMP3X3ji00+QY0I7SNmg20uv1Yf6Pz0qHHe8XPhT2A4t0i/ERgSqncwmd272R26UGcITxoWFc3dM6daFJrso/VbHDhAsjr1zJm51s0/aE8SImWuzNwdD5vM37J8oayLgNrd3HslIgFuEVp+4L2/wEbl9QwP94GIGpQ6wSgN33eHuX4oFj7brAnACaprQVJ20DbPpzlRhEUHc8gqSFqx4PERAQoSddfbhuZNv1wNB7+J50jybPbLRqFl2BN763i90xLahncZrb867ETPG7n8OZHqAleIAdw3iH1cy0gtkKU/fLTakFn7rxTIYwMDMG8soPL1N7B+PFC+1pMKCesBFWJpLQtiYXOmYQFETBYV0puOUrB7m2M9nb9LJFTlHxTj7sPFxuyDOqf8HuT5Wwf1dxEzjuvw/P/+zUFnQshd/NlczBzork2qaUwf14qchmM/ZX0EsOU5U6+D2e8= hpc-devel" >> /root/.ssh/authorized_keys

chmod 600 /root/.ssh/authorized_keys;
```
this provisioning script performs several tasks related to network configuration and SSH key setup. It checks and removes existing network connections, creates a new Ethernet connection, brings it up, and adds an SSH public key to the root user's authorized_keys file.

Then, if you run ```vagrant up``` you deploy the virtual machine
##### The Vagrant file
```ruby 
servers = [ 
	{ :hostname => "k01", :ip => "192.168.133.80" }, 
	# { :hostname => "k02", :ip => "192.168.133.81" }, 
	# { :hostname => "k03", :ip => "192.168.133.83" }, ]
```
Defines an array of server configurations with hostnames and IP addresses.
```Ruby
Vagrant.configure("2") do |config|
```
This block initializes the Vagrant configuration. The argument "2" specifies the version of the Vagrant configuration.
```Ruby
config.vm.box = "fedora/39-cloud-base"
```
This line sets the base box for the virtual machines. In this case, it's using the "fedora/39-cloud-base" box.
```Ruby
config.vm.provider :libvirt do |lv| 
	lv.memory = 2048 
	lv.cpus = 2 
end
```
These lines configure the Libvirt provider settings, specifying the amount of memory and the number of CPUs for each virtual machine.
```Ruby
servers.each do |conf| 
	config.vm.define conf[:hostname] do |node| 
		node.vm.hostname = conf[:hostname]
		node.vm.synced_folder ".", "/vagrant", disabled: true
		node.vm.network :private_network,
                :libvirt__network_name => 'kub-devel'
		node.vm.provision :shell,
                  :path => './0_bash_common_provisioning.sh',
                  :args => [ conf[:ip] ]
	end
end
```
This loop iterates over each server configuration and defines a virtual machine for each one.

Inside the loop, for each virtual machine, it sets the hostname, disables synced folders, configures a private network using Libvirt, and provisions the virtual machine using a shell script (`0_bash_common_provisioning.sh`) with the specified IP address as an argument.

##### The provisioning file
```bash
if [[ $(nmcli -t c show  | grep "Wired connection 2" | wc -l) -ne 0 ]]; then
  nmcli c del "Wired connection 2";
fi
```
This block checks if a network connection named "Wired connection 2" exists using `nmcli` (NetworkManager Command-Line Interface). If it exists, the script deletes the connection.
```bash
if [[ $(nmcli -t c show  | grep hpc | wc -l) -ne 0 ]]; then
  nmcli c del "hpc-net";
fi
```
Similar to the previous block, this one checks if a network connection named "hpc-net" exists, and if it does, the script deletes that connection.
```bash
nmcli con add type ethernet \
    con-name hpc-net \
    ifname eth1 \
    ip4 $1/24 \
    gw4 192.168.132.1 \
    ipv4.method manual \
    autoconnect yes;
```
This block uses `nmcli` to add a new Ethernet connection named "hpc-net." It specifies the interface (`eth1`), the IPv4 address and subnet mask passed as arguments, the gateway, and sets the IPv4 method to manual. The connection is set to autoconnect.
```bash
nmcli con up hpc-net;
```
This line brings up the newly created network connection "hpc-net."
```bash
echo "ssh-rsa ... hpc-devel" >> /root/.ssh/authorized_keys
```
This line appends an SSH public key to the `/root/.ssh/authorized_keys` file. The SSH key provided is for the user "hpc-devel."
```bash
chmod 600 /root/.ssh/authorized_keys;
```
This line sets the correct permission on the *authorized_keys* file

#### Starting up the Virtual Machine
Now run the virtual machine and we can se that it does some setup operations
```bash
# Load modules
modprobe overlay
modprobe br_netfilter
```
This loads kernel modules (`overlay` and `br_netfilter`) that are commonly used in containerization and networking, especially in the context of Kubernetes.
- [Overlay fs](https://docs.kernel.org/filesystems/overlayfs.html)
- [Netfilter](https://en.wikipedia.org/wiki/Netfilter)
```bash
# make load permanent
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
This ensures that the kernel modules (`overlay` and `br_netfilter`) are loaded automatically at system boot by adding them to the `k8s.conf` file in the `/etc/modules-load.d/` directory.
```bash
# change kernel parameters
cat <<EOF |  tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
```
This sets kernel parameters related to networking and IP forwarding, ensuring proper communication between containers and enabling IP forwarding.
- `net.bridge.bridge-nf-call-iptables = 1`: This setting enables iptables to process bridged IPv4 traffic. In a Kubernetes environment, iptables is often used for network address translation (NAT) and other network-related tasks.
- `net.bridge.bridge-nf-call-ip6tables = 1`: Similar to the previous setting, this one enables iptables to process bridged IPv6 traffic.
- `net.ipv4.ip_forward = 1`: This setting enables IP forwarding for IPv4 traffic. IP forwarding allows the system to forward packets from one network interface to another. In the context of Kubernetes, enabling IP forwarding is essential for communication between containers and the external network.
	- it is often required by the container runtime and network plugins.
```bash
# Load kernel parameters at runtime
sysctl --system
```
This command loads the kernel parameters specified in the `k8s.conf` file without requiring a system reboot.
```bash
# disable zram
touch /etc/systemd/zram-generator.conf
swapoff -a
```
This disables [zram](https://en.wikipedia.org/wiki/Zram) (compressed RAM), a feature that uses compression to store data in RAM more efficiently. Disabling ZRAM might be done for specific requirements or compatibility with certain applications.
1. **`touch /etc/systemd/zram-generator.conf`**: This command creates an empty file at `/etc/systemd/zram-generator.conf`. The purpose of this file is to signal to the system that ZRAM should not be automatically configured or enabled. It effectively disables the ZRAM generator, which is responsible for setting up compressed swap space in RAM.
2. **`swapoff -a`**: This command turns off all swap devices. In this context, it's used to deactivate any existing ZRAM swap devices that might be active. The option `-a` stands for "all," indicating that all swap devices should be turned off.
>[!Swap devices]
>A swap device, often referred to as swap space or a swap partition, is an area on a storage device (usually a hard disk or SSD) that the operating system uses as virtual memory. The purpose of swap space is to temporarily store data that doesn't fit into the computer's physical RAM (Random Access Memory). When the physical RAM is fully utilized, the operating system can move less frequently used data from RAM to the swap space to free up memory for more immediate needs.
```bash
# Security? What security? Disable SElinux.
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```
This temporarily disables [SELinux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux) by setting it to permissive mode. SELinux (Security-Enhanced Linux) is a security feature in Linux that provides mandatory access control. Disabling it might be done for simplicity during the initial setup, but it's generally recommended to understand and configure SELinux for security reasons.

Now we have the modules to perform netfilter codes on the bridges between pods whitin the same node but we don't have 
#### Installation
The following script snippet adds the Kubernetes repository configuration to the YUM package manager on a Linux system.
- **`exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni`**: Excludes specific packages from being installed from this repository.
```bash
# attention to exclude!!
# you have to add the kubernetes repository
cat << EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.28/rpm/repodata/repomd.xml.key
# we say that these app cannot be found inside this repo
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```
We install iproute (utility to check routes), wget (Utility used to download stuff), CRI-O, kubelet, kubeadm (makes the installation easier) and kubectl
- CRI-o will also download the network manager plugin which manages the interpod communication
>[!Kubectl]
>Kubectl is not really needed but is the way we interact with our cluster

>[!Warning]
> We assume that the resources are managed with cgroup in systemd. If you are not managing resources with systemd you have to specify it in the *crio.conf* file
```bash
# utils
dnf install iproute-tc wget -y
# CRI-o
dnf install crio -y
dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
#### CRI-O configuration
- */etc* is the folder for configuration files in linux
- */etc/crio/crio.conf* contains the configuration info for CRI-o

The most important informations in the **crio.conf** file are 
- The informations about the **network plugin**
- The **sleep container**
	- You can create an empty pod container which does nothing 
	- There are situations in which the standard default sleep container is not working and you might want to change it with something else
		- container are not portable, they are compiled against a certain architecture and therefore they do not work if tried to execute on a different architecture

In *etc/cni/net.d/* there are two configuratio files about the network plugin
- in of them is 100-crio-bridge.conflist
	- it contains the ip range of the pods that are connecting to the bridge

This `sed` command modifies a configuration file by replacing a specific IP address range.
```
sed -i 's/10.85.0.0\/16/10.17.0.0\/16/' /etc/cni/net.d/100-crio-bridge.conflist
```
Another interesting directory is */etc/containers*
- It contains some informations about the containers (for example the registries)

Now we can finallly enable crio and kubelet

```bash
systemctl enable --now crio
systemctl enable --now kubelet
```
- *enable* starts the application everytime I start the (virtual) machine
- *--now* please start it now
>[!Note]
If you do *crictl ps* between the first and the second command you can already start seeing that there are not cointainers running 

The `kubeadm init` command is used to initialize a Kubernetes control-plane node.
```bash
kubeadm init --pod-network-cidr=10.17.0.0/16
# --services-cidr=10.96.0.0/12 /default
# --control-plane-endpoint 192.168.132.80 /needed for HA
```
-**`--pod-network-cidr=10.17.0.0/16`**: Specifies the range of IP addresses to allocate for Pod networking. Pods in the cluster will be assigned IPs from this CIDR range.
	- This range must coincide with the one written inside the CRIO configuration file 
- **`--services-cidr=10.96.0.0/12` (commented out)**: This option sets the CIDR range for service IPs in the cluster.
	- they do not have to overlap with your infrastructure
- **`--control-plane-endpoint 192.168.132.80` (commented out)**: This option sets the endpoint for the control plane.
	- if you don't set this parameter, your kubernetes thinks that there is only the current node's control plane

Now, if you use *crictl* to see which pods are running you see
- *coredns*: internal service in order to find resources
- *kube-proxy*
- *kube-scheduler*
- *kube-controller-manager*
- *etcd*
- *kube-apiserver*

Since kubelet is not running inside a container, to know its status you have to ask to *systemd* with 
```bash
systemctl status kubelet
```
And it will tell you that kubelet is up and running

Remind that we said that we want to use kubectl to manage our cluster. To make it run we have to copy the credentials. 

These commands set up the Kubernetes configuration for the current user by copying the admin configuration file (`admin.conf`) to the user's home directory and creating an alias for the `kubectl` command.
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
alias k=kubectl
```
In this way we created a config file in  .kube/config. It contains:
- **`apiVersion`**: The version of the Kubernetes configuration API. In this case, it's `v1`.
- **`clusters`**: Defines the cluster information.
    - **`certificate-authority`**: The path to the certificate authority (CA) certificate file used to verify the API server's certificate.
    - **`server`**: The URL of the Kubernetes API server.
- **`contexts`**: Specifies the context information.
    - **`cluster`**: The name of the cluster to use.
    - **`user`**: The name of the user to use.
- **`current-context`**: The name of the current context. It refers to one of the contexts defined in the `contexts` section
- **`kind`**: The type of the configuration. It is set to `Config`.
- **`users`**: Defines the user information.
    - **`name`**: The name of the user.
    - **`user`**: Specifies the user's credentials.
        - **`client-certificate`**: The path to the user's client certificate file.
        - **`client-key`**: The path to the user's private key file.
#### The cluster's status
```bash
kubectl get --raw=/<keywork>
```
Here keywork can be:
- healthz: (liveness probe) tests if the deployed app is running correctly
- livez: (startup probe) tests if booting has ended cirrectly
- readyz: (readiness probe) checks if the app is ready to accept traffic 
It is also possible to add a query string
```bash
kubectl get --ray=/healthz?verbose
```
Every application within our cluster will expose some of these addresses (it can also rename them) and will be used by the control manager to decide how to steer the cluster toward a certain state
- The specs for each of these calls is defined inside the object specification for each pod

>[!note]
>Kubectl just does GET and POST requests to the API server

If we look inside the file *app5.yaml* (configuration file for a pod) we can see the livenessProbe part
```
livenessProbe:
	httpGet:
		path: /healthy
		port: 8080
	InitialDelaySeconds: 5
	timeoutSeconds: 1
	periodSeconds: 10
	failureThreshold: 3
```
So, the get request will be sent with 5 seconds delay, if we don't get the answer in 1 second the test is considered failde and we try another time after 10 seconds. If the test fails 3 times the app is rebooted
>[!Note]
>Kubernetes performs the livenessProbe every *"period seconds"* independently of the status of the pod
#### Useful (Kubectl) commands
The following commnd prints the available nodes
```bash
kubectl get nodes
```
The following command gives you a description of the node
```bash
kubectl describe node
```
The output contains the role that your node is covering in your cluster
	- Not very significant because it is inferred by the labels that are applied to our cluster

>[!labels]
>_Labels_ are key/value pairs that are attached to [objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/#kubernetes-objects) such as Pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. 
>
>The key is everything that is at the left of the equal(=) and the value is the right side. The key  can have two parts, a prefix and the name of the key, they are separated by a slash (/)

This node uses the label ```node-role.kubernetes.io/control-plane=```so kubernetes assumes that is the control plane node, another common label is ```node-role.kubernetes.io/worker=``` 

>[!Annotations]
>You can use either labels or annotations to attach metadata to Kubernetes objects. Labels can be used to select objects and to find collections of objects that satisfy certain conditions. In contrast, annotations are not used to identify and select objects.

So, while labels attach identifying metadata to a K8s object, annotations attach additional information that is not identifying.

**Taints** are a way to repel certain pods from nodes:
- **Taints:**
    - A taint is a key-value pair associated with a node in the Kubernetes cluster.
    - Each taint has an effect, which can be either `NoSchedule`, `PreferNoSchedule`, or `NoExecute`.
        - `NoSchedule`: Pods that do not tolerate this taint will not be scheduled on the node.
        - `PreferNoSchedule`: Kubernetes will try to avoid scheduling pods that do not tolerate this taint but may do so if necessary.
        - `NoExecute`: Existing pods on the node that do not tolerate this taint will be evicted.
- **Tolerations:**
    - Tolerations are specified in the pod's definition and indicate that the pod can tolerate certain taints.
    - A pod can have multiple tolerations, allowing it to be scheduled on nodes with different taints.
- **Use Cases:**
    - Taints are commonly used in scenarios where nodes have specific characteristics or resources (e.g., GPU, specialized hardware, control plane).
    - They can be used to repel pods from nodes during maintenance or to ensure that critical workloads are isolated on dedicated nodes.

In Kubernetes, _namespaces_ provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces.
- Kubernetes namespaces are just folders
- This is a way to compartimentarize your kubernetes application workflow

The following command shows you all the running pods
```bash
kubectl get pod
```
The following command shows you all the running pods in all the namespaces
```bash
kubectl get pod --all-namespces
``` 

You can see that, in this case, the kube-system pods are showed too.
The output provides some informations about every pod:
- **ready** is the readiness check
- the **name** of the pod (the name that you assign inside the metadata file)
- **namespace**
- ...
>[!Note]
>If you addd ```-o wide``` you get a larger output with more informations. Like the IP of every pod

Note that since all the kube-system pods runs in the original network namespace they have the IP of the machine while every pod inside a node (A virtual machine in our case) lives within its own network namepsace and has a single network interface

Another important keywork is the `kubectl describe pod` command. It is used to display detailed information about a specific pod. When you run this command, it provides a comprehensive overview of the selected pod's configuration, status, events, and other relevant details.
```bash
kubectl describe pod <pod_name>
```
The `kubectl explain` command in Kubernetes is used to display documentation for API resources, including pods. It helps you understand the structure and possible configurations for a particular resource type. For example, if you want informations about the Pod resource you can type
```bash
kubectl explain pods
```
To view the logs of a given pod in Kubernetes, you can use the `kubectl logs` command.
```bash
kubectl logs <pod>
```
>[!Note]
>If your pod has multiple containers you can also specify a single on of them with ````kubectl logs mypod -c mycontainer```` 

To execute a shell inside the single containers within *mypod* pod 
```bash
kubectl exec -it mypod -- /bin/bash
```
If *mypod* contains multiple containers you have to specify the container
```bash
kubectl exec -it kuard -c <container_name> -- /bin/bash
```
To stop running pods in Kubernetes, you generally don't "stop" them like you would with traditional processes. Instead, you delete the pods with one of the following commands
```bash
kubectl delete pods/kuard
kubectl delete -f kube-installation/home/app2.yaml
```
- ```app2.yaml``` is the file used to create the pod

When a Pod is deleted, it is not immediately killed. Instead, if you run kubectl get pods you will see that the Pod is in the Terminating state. All Pods have a termina‐ tion grace period. By default, this is 30 seconds. When a Pod is transitioned to Terminating it no longer receives new requests. In a serving scenario, the grace period is important for reliability because it allows the Pod to finish any active requests that it may be in the middle of processing before it is terminated. It’s important to note that when you delete a Pod, any data stored in the containers associated with that Pod will be deleted as well. If you want to persist data across mul‐ tiple instances of a Pod, you need to use *PersistentVolumes*.
#### Kubernetes namespaces
In the real world Kubernetes applications there are several namespaces and several users

Kubernetes uses namespaces to organize objects in the cluster. You can think of each namespace as a folder that holds a set of objects. By default, the kubectl commandline tool interacts with the **default namespace**.

Use the following flag to interact with resources contained by a specific namespace
```bash
--namespace=mystuff
```
while, the following one can be used to refer to all the namespaces 
```bash
--all-namespaces
```
If you want to change the default namespace more permanently, you can use a con‐ text. This gets recorded in a kubectl configuration file, usually located at $HOME/.kube/config. This configuration file also stores how to both find and authen‐ ticate to your cluster. The following commands first create a new context and then start using it 
```bash
kubectl config set-context my-context --namespace=mystuff
kubectl config use-context my-context
```
#### Networking inside Kubernetes
![[Pasted image 20231214130240.jpg]]
Every Pod will have its own network namespace and containers inside that Pod share the same IP address and ports. All communication between these containers would happen via the localhost as they are all part of the same namespace. ( Represented by the green line in the diagram )

In Kubernetes, every node has a designated CIDR range of IPs for Pods. This would ensure that every Pod gets a unique IP address that can be seen by other Pods in the cluster and also ensures that when a new Pod is created, the IP address never overlaps. Unlike Container-to-Container networking, Pod-to-Pod communication happens using real IPs, whether the Pod is deployed on the same node or a different node in the cluster.

You can notice from the diagram above that, for Pods to communicate with each other, the traffic has to flow between the Pod network namespace and the Root network namespace. This is achieved by connecting both the Pod namespace and the root namespace by a virtual ethernet device or a veth pair (veth0 to Pod namespace 1 and veth1 to Pod namespace 2 in the diagram). Both these virtual interfaces would be connected via a virtual network bridge which will then allow traffic to flow between them using the ARP protocol.

So if data is sent from Pod 1 to Pod 2, the flow of events would like this ( refer to diagram above )
1. Pod 1 traffic flows through eth0 to the root network namespaces virtual interface veth0.
2. Then traffic goes via veth0 to the virtual bridge which is connected to veth1.
3. Traffic goes via the virtual bridge to veth1.
4. Finally, traffic reaches eth0 interface of Pod 2 via veth1.

The inter-node communication is mainly managed by the CNI-plugin inside CRI-o, therefore, there is no standard. However, the main concept is that when the virtual bridge receives a packet/frame (The layer ot which the bridge works depends on the underlying plugin) whose recipient is not a pod inside this node, will be forwarded to the repient node by the CNI plugin using a VXLAN interface
##### VXLAN
At its most basic level, VXLAN is a tunnelling protocol. In the case of VXLAN specifically, the tunnelled protocol is Ethernet. In other words, it’s merely another Ethernet frame put into a UDP packet, with a few extra bytes serving as a header — or a transport mechanism that supports the software controlling the devices that use the VXLAN.

VXLAN is a formal internet standard, specified in RFC 7348. If we go back to the OSI model, VXLAN is another application layer-protocol based on UDP that runs on port 4789.

In terms of VXLAN, the underlay is the Layer 3 (L3) IP network that routes VXLAN packets as normal IP traffic. The overlay refers to the virtual Ethernet segment created by this forwarding.
![[1_lXF5U-aUj1eNW7e3eknVcA.webp]]
The term VTEP (VXLAN Tunnel Endpoint) generally refers to any device that originates or terminates VXLAN traffic. There are two major types, based on how the encapsulation or de-encapsulation of VXLAN packets is handled: hardware VTEP devices handle VXLAN packets in hardware, while software VTEP devices handle VXLAN packets in software.
#### Making services available
If we run *ngix* (A simple web server) inside a container in our node, how can we make it accessible from the outside?

The simplest way is to ask kubernetes to map the port 80 of the pod to the ethernet port (let's say 30000) of the node and we will connect 
```bash
kubectl port-forwarding my-first-pod 30000:80 --address 0.0.0.0 
```
a secure tunnel is created from your local machine, through the Kubernetes master, to the instance of the Pod running on one of the worker nodes.

The `--address 0.0.0.0` flag in the `kubectl port-forward` command specifies the IP address to which the local port forwarding should be bound. In this context, `0.0.0.0` is a special value that means "bind to all available network interfaces."

When you use `--address 0.0.0.0`, it allows external access to the port-forwarded service from any network interface, making it accessible from outside the machine where the `kubectl port-forward` command is running. This can be particularly useful if you want to access the forwarded port from a different machine or if you want to make the service available on the network.
#### Resource management
A resource metric for a pod specifies the amount of resource that is assigned to that pod.    

Kubernetes allows users to specify two different resource metrics. Resource requests specify the minimum amount of a resource required to run the application. Resource limits specify the maximum amount of a resource that an application can consume. 

For example, to request that the *kuard* container lands on a machine with half a CPU free and gets 128 MB of memory allocated to it, we define the Pod as shown in the following code
```yaml
apiVersion: v1 
kind: Pod 
metadata: 
	name: kuard 
spec: 
	containers: 
	- image: gcr.io/kuar-demo/kuard-amd64:blue 
	- name: kuard 
	- resources: 
		requests: 
			cpu: "500m" 
			memory: "128Mi" 
		ports: 
			- containerPort: 8080 
				name: http 
				protocol: TCP
```
>[!Note]
>We are only specifying the minimum amount of resources for this pod. So, if more resources are available the pod can use them.

The Kubernetes scheduler will ensure that the sum of all requests of all Pods on a node does not exceed the capacity of the node. Therefore, a Pod is guaranteed to have at least the requested resources when running on the node. If your nodes does not satisfy the requests (in terms of resources) of a certain pod, the pod will not be scheduled untill a new node with enpigh resources will be turned on.

You can also set a maximum on a Pod’s resource usage via resource *limits*.
```yaml
apiVersion: v1 
kind: Pod 
metadata: 
	name: kuard 
spec: 
	containers: 
	- image: gcr.io/kuar-demo/kuard-amd64:blue 
	- name: kuard 
	- resources: 
		requests: 
			cpu: "500m" 
			memory: "128Mi"
		limits:
			cpu: "1000m"
			memory: "256Mi" 
		ports: 
			- containerPort: 8080 
				name: http 
				protocol: TCP
```
When you establish limits on a container, the kernel is configured to ensure that con‐ sumption cannot exceed these limits
# Lectures - 15 Dec 2023
*- The notes about these lectures have not been reviewed yet, stay tuned*

Today we will expose the capabilities of the Kubernetes infrastructure to expose a pod to the outside world $\rightarrow$ services, load balancer

How do we find things (pods)? Once the DNS solves your reference, you get the IP of a load balancer service, not the real computing node

The first thing that we will do is to perform load balancing inside Kubernetes
- Inside Kubernetes you have a DNS (kubeDNS)
- Every service has cluster IP $\rightarrow$ every service is linked to a set of pod that run that application

The idea is that the kubeDNS is populated by the names taken from the service objects

If your app wants to send a certain app via name
1. ask the name translation to the dns
2. the dns answers with the service ip
3. the service dispaches the traffic to one of the available pod

>[!Node]
>kubeDNS itself is a load-balanced service

Now, the first thing that we want to do is to make our app accessible from the outside

The simplest way to expose a set of pod is the **nodeport**
- it takes a set of pods, and assigns them a cluster up 
- When the outside app want to access a service asks a DNS to translate the name and gets the ip of every node containing the app
- Asks to the first node and, via kube-proxy, the first node redirects the traffic to the node that ows the application and then redirect the answer to the outside app
With this mechanism if one node goes down, the outside app can use the other IPs

Inside this framework, the app is running on a single node, so when that nide crashes we have to wait that the app spawns on another node $\rightarrow$ we want more replicas of the same app on multiple node

We use the replica set to do this
- every app in the same replica set has the same label 
- The service is aware of this label and manages the receiving traffic taking into account the running pods with that specific label

In this framework, we are able to get both HA and Load Balancing

Assume that we have one pod for a single service 
```bash
apiVersion: v1
kind: Pod
metadata:
	name: blabla
```

Everytime that we create a service, Kubernetes will assign it a ClusterIP

What is an Endpoint in a nodeport node?

Now that we created a service, it will automaticaly inserted inside the KubeDNS

Now we talk about Load Balancer
- Not a component of Kubernetes
	- The Load Balancer type is provided by ?

We have our external IP and want to bind it to a set of pods
- We have a service in the middle with a certain ClusterIP
- So the external IP is not owned by any machine
	- We assume that we already have the IP of the service
- If our machine is connected to the cluster via a switch, we get the MAC address that owns the service via the ARP protocol
- For every service running on multiple nodes we have a leader node which owns the service pod and the kube proxy, these redirect the traffict to the node that has the running requested pod
- The main problem is that all the traffic goes throught the leader node $\rightarrow$ bandwidth is a bottleneck
- DNS -> Load balancer IP -> leader ip that owns kube proxy -> pod
	- the load balancer elects the leader
>[!Real case]
>In real cases, the switch is subsititued with a router and all your backend gets hidden. Traffic is self-balanced thanks to the BGP protocol

How do you install components in kubernetes? We use pods -> we can install metallb and use it
- Installing this application creates some roles, some new resouces and installing some pods that will take care of the new resources
>[!Installing something in Kubernetes]
>Installating something in Kubernetes means creating a lot of resources and installing some pods that manage them

```bash
```
The next thing that we want to do is adding a reverse proxy on top of the load balancer
### Kubernetes storage
On-disk files in a container are ephemeral, which presents some problems for non-trivial applications when running in containers.

Possible problems:
- Container state is not saved so all of the files that were created or modified during the lifetime of the container are lost if the container crashes.
-  Multiple containers are running in a `Pod` and need to share files.  It can be challenging to setup and access a shared filesystem across all of the containers.

The kubernetes **Volume** solves both these problems.

Kubernetes supports many types of volumes. A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) can use any number of volume types simultaneously. [Ephemeral volume](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/) types have a lifetime of a pod, but [persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) exist beyond the lifetime of a pod. When a pod ceases to exist, Kubernetes destroys ephemeral volumes; however, Kubernetes does not destroy persistent volumes. 

>[!Observation]
>For any kind of volume in a given pod, data is preserved across container restarts.

At its core, **a volume is a directory**, possibly with some data in it, which is accessible to the containers in a pod.

To use a volume, specify the volumes to provide for the Pod in `.spec.volumes` and declare where to mount those volumes into containers in `.spec.containers[*].volumeMounts`. A process in a container sees a filesystem view composed from the initial contents of the [container image](https://kubernetes.io/docs/reference/glossary/?all=true#term-image), plus volumes (if defined) mounted inside the container.

Volumes mount at the [specified paths](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath) within the image. For each container defined within a Pod, you must independently specify where to mount each volume that the container uses.

>[!Note]
>- Volumes cannot mount within other volumes
>- a volume cannot contain a hard link to anything in a different volume

Now we will explore some specific kinds of volume that are available in Kubernetes
##### ConfigMap
A [ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) provides a way to inject configuration data into pods. The data stored in a ConfigMap can be referenced in a volume of type `configMap` and then consumed by containerized applications running in a pod.

When referencing a ConfigMap, you provide the name of the ConfigMap in the volume. You can customize the path to use for a specific entry in the ConfigMap. The following configuration shows how to mount the `log-config` ConfigMap onto a Pod called `configmap-pod`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox:1.28
      command: ['sh', '-c', 'echo "The app is running!" && tail -f /dev/null']
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```
The `log-config` ConfigMap is mounted as a volume, and all contents stored in its `log_level` entry are mounted into the Pod at path `/etc/config/log_level`. Note that this path is derived from the volume's `mountPath` and the `path` keyed with `log_level`.
>[!Note]
>- You must create a [ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) before you can use it.
>- A ConfigMap is always mounted as `readOnly`.
>- A container using a ConfigMap as a [`subPath`](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath) volume mount will not receive ConfigMap updates.
##### emptyDir
For a Pod that defines an `emptyDir` volume, the volume is created when the Pod is assigned to a node. As the name says, the `emptyDir` volume is initially empty. All containers in the Pod can read and write the same files in the `emptyDir` volume, though that volume can be mounted at the same or different paths in each container. When a Pod is removed from a node for any reason, the data in the `emptyDir` is deleted permanently.
>[!Note]
> A container crashing does _not_ remove a Pod from a node. The data in an `emptyDir` volume is safe across container crashes.

The `emptyDir.medium` field controls where `emptyDir` volumes are stored. By default `emptyDir` volumes are stored on whatever medium that backs the node such as disk, SSD, or network storage, depending on your environment. If you set the `emptyDir.medium` field to `"Memory"`, Kubernetes mounts a tmpfs (RAM-backed filesystem) for you instead. While tmpfs is very fast be aware that, unlike disks, files you write count against the memory limit of the container that wrote them.

A size limit can be specified for the default medium, which limits the capacity of the `emptyDir` volume. The storage is allocated from [node ephemeral storage](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#setting-requests-and-limits-for-local-ephemeral-storage). If that is filled up from another source (for example, log files or image overlays), the `emptyDir` may run out of capacity before this limit.

emptyDir configuration example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLim
```
##### hostPath
A `hostPath` volume mounts a file or directory from the host node's filesystem into your Pod. This is not something that most Pods will need, but it offers a powerful escape hatch for some applications.
```yaml
# This manifest mounts /data/foo on the host as /foo inside the
# single container that runs within the hostpath-example-linux Pod.
#
# The mount into the container is read-only.
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example-linux
spec:
  os: { name: linux }
  nodeSelector:
    kubernetes.io/os: linux
  containers:
  - name: example-container
    image: registry.k8s.io/test-webserver
    volumeMounts:
    - mountPath: /foo
      name: example-volume
      readOnly: true
  volumes:
  - name: example-volume
    # mount /data/foo, but only if that directory already exists
    hostPath:
      path: /data/foo # directory location on host
      type: Directory # this field is optional
```
#### local
A `local` volume represents a mounted local storage device such as a disk, partition or directory.

Local volumes can only be used as a statically created PersistentVolume. Dynamic provisioning is not supported.

`local` volumes are subject to the availability of the underlying node and are not suitable for all applications. If a node becomes unhealthy, then the `local` volume becomes inaccessible by the pod. The pod using this volume is unable to run. Applications using `local` volumes must be able to tolerate this reduced availability, as well as potential data loss, depending on the durability characteristics of the underlying disk.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```
You must set a PersistentVolume `nodeAffinity` when using `local` volumes. The Kubernetes scheduler uses the PersistentVolume `nodeAffinity` to schedule these Pods to the correct node.

PersistentVolume `volumeMode` can be set to "Block" (instead of the default value "Filesystem") to expose the local volume as a raw block device.
##### nfs
An `nfs` volume allows an existing NFS (Network File System) share to be mounted into a Pod. Unlike `emptyDir`, which is erased when a Pod is removed, the contents of an `nfs` volume are preserved and the volume is merely unmounted. This means that an NFS volume can be pre-populated with data, and that data can be shared between pods. NFS can be mounted by multiple writers simultaneously.
##### secret
A `secret` volume is used to pass sensitive information, such as passwords, to Pods. You can store secrets in the Kubernetes API and mount them as files for use by pods without coupling to Kubernetes directly. `secret` volumes are backed by tmpfs (a RAM-backed filesystem) so they are never written to non-volatile storage.
>[!Note]
>A secret is always mounted as readOnly
#### Using subPath
Sometimes, it is useful to share one volume for multiple uses in a single pod. The `volumeMounts.subPath` property specifies a sub-path inside the referenced volume instead of its root.

The following example shows how to configure a Pod with a LAMP stack (Linux Apache MySQL PHP) using a single, shared volume. This sample `subPath` configuration is not recommended for production use.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-lamp-site
spec:
    containers:
    - name: mysql
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "rootpasswd"
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: site-data
        subPath: mysql
    - name: php
      image: php:7.0-apache
      volumeMounts:
      - mountPath: /var/www/html
        name: site-data
        subPath: html
    volumes:
    - name: site-data
      persistentVolumeClaim:
        claimName: my-lamp-site-data
```
#### PersistentVolumes
>[!Storage classes]
>A StorageClass provides a way for administrators to describe the _classes_ of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators. Kubernetes itself is unopinionated about what classes represent.

The PersistentVolume subsystem provides an API for users and administrators that abstracts details of how storage is provided from how it is consumed. To do this, we introduce two new API resources: PersistentVolume and PersistentVolumeClaim.

A **_PersistentVolume_** (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/). It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.

A **_PersistentVolumeClaim_** (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes.

There are two ways PVs may be provisioned: statically or dynamically.
##### Static
A cluster administrator creates a number of PVs. They carry the details of the real storage, which is available for use by cluster users. They exist in the Kubernetes API and are available for consumption.
##### Dynamic
When none of the static PVs the administrator created match a user's PersistentVolumeClaim, the cluster may try to dynamically provision a volume specially for the PVC. This provisioning is based on StorageClasses: the PVC must request a [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/) and the administrator must have created and configured that class for dynamic provisioning to occur.
##### Binding
A user creates, or in the case of dynamic provisioning, has already created, a PersistentVolumeClaim with a specific amount of storage requested and with certain access modes. A control loop in the control plane watches for new PVCs, finds a matching PV (if possible), and binds them together. If a PV was dynamically provisioned for a new PVC, the loop will always bind that PV to the PVC. Otherwise, the user will always get at least what they asked for, but the volume may be in excess of what was requested. Once bound, PersistentVolumeClaim binds are exclusive, regardless of how they were bound. A PVC to PV binding is a one-to-one mapping, using a ClaimRef which is a bi-directional binding between the PersistentVolume and the PersistentVolumeClaim.

Claims will remain unbound indefinitely if a matching volume does not exist. Claims will be bound as matching volumes become available. For example, a cluster provisioned with many 50Gi PVs would not match a PVC requesting 100Gi. The PVC can be bound when a 100Gi PV is added to the cluster.
##### Using
Pods use claims as volumes. The cluster inspects the claim to find the bound volume and mounts that volume for a Pod. For volumes that support multiple access modes, the user specifies which mode is desired when using their claim as a volume in a Pod.

Once a user has a claim and that claim is bound, the bound PV belongs to the user for as long as they need it. Users schedule Pods and access their claimed PVs by including a `persistentVolumeClaim` section in a Pod's `volumes` block.
##### Reclaiming
The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim. Currently, volumes can either be Retained, Recycled, or Deleted.
###### Retain
When the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered "released". But it is not yet available for another claim because the previous claimant's data remains on the volume.
###### Delete
For volume plugins that support the `Delete` reclaim policy, deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure. Volumes that were dynamically provisioned inherit the [reclaim policy of their StorageClass](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy), which defaults to `Delete`.
###### Recycle (Deprecated)
If supported by the underlying volume plugin, the `Recycle` reclaim policy performs a basic scrub (`rm -rf /thevolume/*`) on the volume and makes it available again for a new claim.

>[!Observation]
>Se guardiamo i file creati dal container, dal punto punto di vista del file system del nodo sono file dell'utente root

##### Types of Persistent Volumes
PersistentVolume types are implemented as plugins. Kubernetes currently supports several plugins, during the lecture we have seen the following:
- [`hostPath`](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) - HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using `local` volume instead)
- [`local`](https://kubernetes.io/docs/concepts/storage/volumes/#local) - local storage devices mounted on nodes.
- [`nfs`](https://kubernetes.io/docs/concepts/storage/volumes/#nfs) - Network File System (NFS) storage
##### Spec and Status
Each PV contains a spec and status, which is the specification and status of the volume. The name of a PersistentVolume object must be a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```
Generally, a PV will have a specific **storage capacity**. This is set using the PV's `capacity` attribute which is a [Quantity](https://kubernetes.io/docs/reference/glossary/?all=true#term-quantity) value.
##### Volume Mode
`volumeMode` is an optional API parameter.

Kubernetes supports two `volumeModes` of PersistentVolumes: `Filesystem`(default) and `Block`.

A volume with `volumeMode: Filesystem` is _mounted_ into Pods into a directory. If the volume is backed by a block device and the device is empty, Kubernetes creates a filesystem on the device before mounting it for the first time.

You can set the value of `volumeMode` to `Block` to use a volume as a raw block device. Such volume is presented into a Pod as a block device, without any filesystem on it. This mode is useful to provide a Pod the fastest possible way to access a volume, without any filesystem layer between the Pod and the volume. On the other hand, the application running in the Pod must know how to handle a raw block device.
##### Access Modes
A PersistentVolume can be mounted on a host in any way supported by the resource provider. As shown in the table below, providers will have different capabilities and each PV's access modes are set to the specific modes supported by that particular volume.

Each PV gets its own set of access modes describing that specific PV's capabilities.
- `ReadWriteOnce`: the volume can be mounted as read-write by a single node. ReadWriteOnce access mode still can allow multiple pods to access the volume when the pods are running on the same node.
- `ReadOnlyMany`: the volume can be mounted as read-only by many nodes.
- `ReadWriteMany`: the volume can be mounted as read-write by many nodes.
##### Node Affinity
>[!Note]
>This field must be set only for local volumes

A PV can specify node affinity to define constraints that limit what nodes this volume can be accessed from. Pods that use a PV will only be scheduled to nodes that are selected by the node affinity. To specify node affinity, set `nodeAffinity` in the `.spec` of a PV.

A PersistentVolume will be in one of the following phases:

- `Available`: a free resource that is not yet bound to a claim
- `Bound` the volume is bound to a claim
- `Released`: the claim has been deleted, but the associated storage resource is not yet reclaimed by the cluster
- `Failed`: the volume has failed its (automated) reclamation

You can see the name of the PVC bound to the PV using `kubectl describe persistentvolume <name>`.
#### PersistentVolumeClaims
Each PVC contains a spec and status, which is the specification and status of the claim. The name of a PersistentVolumeClaim object must be a valid [DNS subdomain name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

Claims use [the same conventions as volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) when requesting storage with specific access modes.

Claims use [the same convention as volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-mode) to indicate the consumption of the volume as either a filesystem or block device.

Claims, like Pods, can request specific quantities of a resource. In this case, the request is for storage. The same [resource model](https://git.k8s.io/design-proposals-archive/scheduling/resources.md) applies to both volumes and claims.

Claims can specify a [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) to further filter the set of volumes. Only the volumes whose labels match the selector can be bound to the claim.

A claim can request a particular class by specifying the name of a [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) using the attribute `storageClassName`. Only PVs of the requested class, ones with the same `storageClassName` as the PVC, can be bound to the PVC.

PVCs don't necessarily have to request a class. A PVC with its `storageClassName` set equal to `""` is always interpreted to be requesting a PV with no class, so it can only be bound to PVs with no class (no annotation or one set equal to `""`).
>[!Claims as Volumes]
>Pods access storage by using the claim as a volume. Claims must exist in the same namespace as the Pod using the claim. The cluster finds the claim in the Pod's namespace and uses it to get the PersistentVolume backing the claim. The volume is then mounted to the host and into the Pod.

### Workload resources
#### Deployment
A _Deployment_ provides declarative updates for [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).

You describe a _desired state_ in a Deployment, and the Deployment [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

The following is an example of a Deployment. It creates a ReplicaSet to bring up three `nginx` Pods:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
- The Deployment creates a ReplicaSet that creates three replicated Pods, indicated by the `.spec.replicas` field.
- The `.spec.selector` field defines how the created ReplicaSet finds which Pods to manage. In this case, you select a label that is defined in the Pod template (`app: nginx`). However, more sophisticated selection rules are possible, as long as the Pod template itself satisfies the rule.

Per aggiornare lo stato del ReplicaSet associato al Deployment mi basta accedere al suo ManifestFile e modificarlo; Kubernetes allora porterà il cluster nel nuovo stato desiderato
>[!Observation]
>Se un Deployment viene usato per aggiornare la versione di un'applicazione abbiamo che Kubernetes emigra gradualmente il traffico verso quell'applicazione ai nuovi pod, piano piano cancella i vecchi, fino ad avere solo pod aggiornati. Chiaramente la nostra applicazione deve tenere conto di questo comportamento e, quindi, deve gestire lo stesso formato di richieste che gestiva la vecchia versione.
#### ReplicaSet
A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

A ReplicaSet is defined with fields, including a selector that specifies how to identify Pods it can acquire, a number of replicas indicating how many Pods it should be maintaining, and a pod template specifying the data of new Pods it should create to meet the number of replicas criteria. A ReplicaSet then fulfills its purpose by creating and deleting Pods as needed to reach the desired number. When a ReplicaSet needs to create new Pods, it uses its Pod template.

A ReplicaSet is linked to its Pods via the Pods' [metadata.ownerReferences](https://kubernetes.io/docs/concepts/architecture/garbage-collection/#owners-dependents) field, which specifies what resource the current object is owned by. All Pods acquired by a ReplicaSet have their owning ReplicaSet's identifying information within their ownerReferences field. It's through this link that the ReplicaSet knows of the state of the Pods it is maintaining and plans accordingly.

A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the OwnerReference is not a [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.

Example(```frontend.yaml```):
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```
##### Writing a ReplicaSet manifest
When the control plane creates new Pods for a ReplicaSet, the `.metadata.name` of the ReplicaSet is part of the basis for naming those Pods. The name of a ReplicaSet must be a valid [DNS subdomain](https://kubernetes.io/docs/concepts/overview/working-with-objects/names#dns-subdomain-names) value, but this can produce unexpected results for the Pod hostnames. A ReplicaSet also needs a [`.spec` section](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).

The `.spec.template` is a [pod template](https://kubernetes.io/docs/concepts/workloads/pods/#pod-templates) which is also required to have labels in place. In our `frontend.yaml` example we had one label: `tier: frontend`. Be careful not to overlap with the selectors of other controllers, lest they try to adopt this Pod.

The `.spec.selector` field is a [label selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/). As discussed [earlier](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#how-a-replicaset-works) these are the labels used to identify potential Pods to acquire. In our `frontend.yaml` example, the selector was:

```yaml
matchLabels:
  tier: frontend
```

In the ReplicaSet, `.spec.template.metadata.labels` must match `spec.selector`, or it will be rejected by the API.

You can specify how many Pods should run concurrently by setting `.spec.replicas`. The ReplicaSet will create/delete its Pods to match this number.

If you do not specify `.spec.replicas`, then it defaults to 1.
##### Alternatives to ReplicaSet
###### Deployment (recommended)[](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#deployment-recommended)
[`Deployment`](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is an object which can own ReplicaSets and update them and their Pods via declarative, server-side rolling updates. While ReplicaSets can be used independently, today they're mainly used by Deployments as a mechanism to orchestrate Pod creation, deletion and updates. When you use Deployments you don't have to worry about managing the ReplicaSets that they create. Deployments own and manage their ReplicaSets. As such, it is recommended to use Deployments when you want ReplicaSets.
###### Bare Pods[](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#bare-pods)
Unlike the case where a user directly created Pods, a ReplicaSet replaces Pods that are deleted or terminated for any reason, such as in the case of node failure or disruptive node maintenance, such as a kernel upgrade. For this reason, we recommend that you use a ReplicaSet even if your application requires only a single Pod. Think of it similarly to a process supervisor, only it supervises multiple Pods across multiple nodes instead of individual processes on a single node. A ReplicaSet delegates local container restarts to some agent on the node such as Kubelet.
###### Job[](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#job)
Use a [`Job`](https://kubernetes.io/docs/concepts/workloads/controllers/job/) instead of a ReplicaSet for Pods that are expected to terminate on their own (that is, batch jobs).
###### DaemonSet[](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#daemonset)
Use a [`DaemonSet`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) instead of a ReplicaSet for Pods that provide a machine-level function, such as machine monitoring or machine logging. These Pods have a lifetime that is tied to a machine lifetime: the Pod needs to be running on the machine before other Pods start, and are safe to terminate when the machine is otherwise ready to be rebooted/shutdown.
###### ReplicationController[](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/#replicationcontroller)
ReplicaSets are the successors to [ReplicationControllers](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/). The two serve the same purpose, and behave similarly, except that a ReplicationController does not support set-based selector requirements as described in the [labels user guide](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors). As such, ReplicaSets are preferred over ReplicationControllers
