Privilege separation is used to ensure that misbehaving users and/or applications from harming the rest of the system.

# Assumptions
1. The system knows who the user is, i.e. they have authenticated themselves, e.g. with a username / password
2. All requests are directed to the operating system

# Users
There are two types of accounts that have a unique identifier ([[W7N1 - OS concepts#Processes|uid]]):
- User accounts - accounts associated with humans
- Service account - accounts associated with background processes

There is one entry in `/etc/passwd` per account with fields:
`username:password:uid:gid:uid_info:home:shell`
(password is an `x` if there is a password for that account)

# File permissions
The unix operating system manages all resources as files. Every file has 3 defined permissions: reads (`r`), writes (`w`), and executes (`x`). Permissions are defined for the owner, the owner's group, and all other users. Root and the owner can change file permissions,

e.g.
`-rw-r--r-- 1 infernox infernox 1162 Nov  4 23:09 'W7N3 - Privilege separation.md'`
This is a file where the owner (`infernox`) has `rw` permissions, while all other users have `r` only

## Directory permissions
In a directory, the `x` permission allows traversing the directory, while the `r` permission allows a lookup the contents of the directory.

`drwxr-xr-x 10 infernox infernox 4096 Nov  2 22:45 computer_security/`
This is a directory where the owner (`infernox`) has `rwx` permissions, and all others have `rx`.

# Processes
Every process has a unique identifier (the PID). Each process is associated with the user that spawned it. When a user runs a process, it runs with the user's privilege. As a thread is spawned by forking an existing process it will inherit its parent's privileges, though it can be specified otherwise.
All processes are isolated in memory.

Every process has a:
- real user ID (uid): the user ID which spawned that process
- effective user ID (euid): the user ID which determines the process' privileges
- saved user ID (suid): the effective user ID before the last modification

Users can change a process' IDs by:
```c
setuid(x)
// uid = x
// euid = x
// suid = x

seteuid(x)
// uid = uid
// euid = x
// suid = suid
```
Root can change euid/uid to arbitrary values, unprivileged users can only change euid to uid or suid.

## Elevating privileges
Some executables (e.g. `passwd`) have the `setuid` bit set, meaning that if A executes a `setuid` file owned by B, then the euid of that process is B instead of A. These executables must be carefully designed and implemented, as the file will generally give higher privileges to the user.

# Unix permissions are coarse grained
All applications installed by one user have the same privileges. This means that if any of these applications are compromised, then all of them are accessible.