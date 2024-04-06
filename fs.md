## ramfs vs shmem
From `init/Kconfig`:
```
config SHMEM
	bool "Use full shmem filesystem" if EXPERT
	default y
	depends on MMU
	help
	  The shmem is an internal filesystem used to manage shared memory.
	  It is backed by swap and manages resource limits. It is also exported
	  to userspace as tmpfs if TMPFS is enabled. Disabling this
	  option replaces shmem and tmpfs with the much simpler ramfs code,
	  which may be appropriate on small systems without swap.
```
And, from `fs/Kconfig`:
```
config TMPFS
	bool "Tmpfs virtual memory file system support (former shm fs)"
	depends on SHMEM
	select MEMFD_CREATE
	help
	  Tmpfs is a file system which keeps all files in virtual memory.

	  Everything in tmpfs is temporary in the sense that no files will be
	  created on your hard drive. The files live in memory and swap
	  space. If you unmount a tmpfs instance, everything stored therein is
	  lost.

	  See <file:Documentation/filesystems/tmpfs.rst> for details.
```
So, to summarize:
- `!SHMEM && !TMPFS`: tmpfs is provided by ramfs
- `SHMEM && !TMPFS`: tmpfs still appears in `/proc/filesystems`, but it can't be mounted, because it's marked with SB_NOUSER in `mm/shmem.c:shmem_fill_super()`
- `SHMEM && TMPFS`: tmpfs is provided by shmem
