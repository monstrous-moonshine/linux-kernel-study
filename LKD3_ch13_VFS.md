## struct file_system_type
`get_sb` was replaced by `mount` between commits c96e41e92b and 1a102ff925. (Nice Python
reference there, btw.)
<details><summary><code>$ git log -n1 c96e41e92b</code></summary>

```
Author: Al Viro <viro@zeniv.linux.org.uk>
Date:   Sun Jul 25 00:17:56 2010 +0400

    beginning of transtion: ->mount()

    eventual replacement for ->get_sb() - does *not* get vfsmount,
    return ERR_PTR(error) or root of subtree to be mounted.

    Signed-off-by: Al Viro <viro@zeniv.linux.org.uk>
```
</details>
<details><summary><code>$ git log -n1 1a102ff925</code></summary>

```
commit 1a102ff92579edeff5e3d5d3c76ca49977898f00
Author: Al Viro <viro@zeniv.linux.org.uk>
Date:   Wed Mar 16 09:07:58 2011 -0400

    vfs: bury ->get_sb()

    This is an ex-parrot.

    Signed-off-by: Al Viro <viro@zeniv.linux.org.uk>
```
</details>

## struct vfsmount
`struct vfsmount` was put inside `struct mount` defined in `fs/mount.h`. Eventually,
almost all of the former struct would be moved to the latter, leaving the former the
tiny struct it is now.
<details><summary><code>$ git log -n1 7d6fec45a5</code></summary>

```
commit 7d6fec45a5131918b51dcd76da52f2ec86a85be6
Author: Al Viro <viro@zeniv.linux.org.uk>
Date:   Wed Nov 23 12:14:10 2011 -0500

    vfs: start hiding vfsmount guts series

    Almost all fields of struct vfsmount are used only by core VFS (and
    a fairly small part of it, at that).  The plan: embed struct vfsmount
    into struct mount, making the latter visible only to core parts of VFS.
    Then move fields from vfsmount to mount, eventually leaving only
    mnt_root/mnt_sb/mnt_flags in struct vfsmount.  Filesystem code still
    gets pointers to struct vfsmount and remains unchanged; all such
    pointers go to struct vfsmount embedded into the instances of struct
    mount allocated by fs/namespace.c.  When fs/namespace.c et.al. get
    a pointer to vfsmount, they turn it into pointer to mount (using
    container_of) and work with that.

    This is the first part of series; struct mount is introduced,
    allocation switched to using it.

    Signed-off-by: Al Viro <viro@zeniv.linux.org.uk>
```
</details>
<details><summary><code>$ git log -n1 c63181e6b6</code></summary>

```
commit c63181e6b6df89176b3984c6977bb5ec03d0df23
Author: Al Viro <viro@zeniv.linux.org.uk>
Date:   Fri Nov 25 02:35:16 2011 -0500

    vfs: move fsnotify junk to struct mount

    Signed-off-by: Al Viro <viro@zeniv.linux.org.uk>
```
</details>

## struct mnt_namespace
`struct mnt_namespace` was moved from `<linux/mnt_namespace.h>` to `fs/mount.h`.
<details><summary><code>$ git log -n1 0226f4923f</code></summary>

```
commit 0226f4923f6c9b40cfa1c1c1b19a6ac6b3924ead
Author: Al Viro <viro@zeniv.linux.org.uk>
Date:   Tue Dec 6 12:21:54 2011 -0500

    vfs: take /proc/*/mounts and friends to fs/proc_namespace.c

    rationale: that stuff is far tighter bound to fs/namespace.c than to
    the guts of procfs proper.

    Signed-off-by: Al Viro <viro@zeniv.linux.org.uk>
```
</details>

All per-process namespace pointers were moved to `struct nsproxy` defined in `<linux/nsproxy.h>`.
Hence, that's what's inside `task_struct` now.
<details><summary><code>$ git log -n1 ab516013ad</code></summary>

```
commit ab516013ad9ca47f1d3a936fa81303bfbf734d52
Author: Serge E. Hallyn <serue@us.ibm.com>
Date:   Mon Oct 2 02:18:06 2006 -0700

    [PATCH] namespaces: add nsproxy

    This patch adds a nsproxy structure to the task struct.  Later patches will
    move the fs namespace pointer into this structure, and introduce a new utsname
    namespace into the nsproxy.

    The vserver and openvz functionality, then, would be implemented in large part
    by virtualizing/isolating more and more resources into namespaces, each
    contained in the nsproxy.

    [akpm@osdl.org: build fix]
    Signed-off-by: Serge Hallyn <serue@us.ibm.com>
    Cc: Kirill Korotaev <dev@openvz.org>
    Cc: "Eric W. Biederman" <ebiederm@xmission.com>
    Cc: Herbert Poetzl <herbert@13thfloor.at>
    Cc: Andrey Savochkin <saw@sw.ru>
    Signed-off-by: Andrew Morton <akpm@osdl.org>
    Signed-off-by: Linus Torvalds <torvalds@osdl.org>
```
</details>
