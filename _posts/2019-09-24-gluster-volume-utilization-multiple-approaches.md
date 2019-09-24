---
layout: post
title: Gluster Volume utilization - Multiple approaches
author: aravindavk
---

Gluster Volume utilization is one of the critical metrics which
everybody interested to know. This blog discusses the multiple
approaches available to get the Volume utilization and comparisons
between them.

## Approach 1 - Fuse mount

Use fuse mount to mount the Gluster Volume and get the `df` output(Or
use `os.statvfs` in case of Python script).

For example,

```
$ mount -t glusterfs localhost:gvol_rep3 /mnt/gvol_rep3
$ df -B1 /mnt/gvol_rep3
$ umount /mnt/gvol_rep3
```

## Approach 2 - libgfapi mount

Use libgfapi mount and run `os.statvfs` to get the Volume utilization.

The following Python example uses
[libgfapi-python](https://github.com/gluster/libgfapi-python) project.

```python
from gluster import gfapi

volume = gfapi.Volume('localhost', 'gvol_rep3')
volume.mount()
st = volume.statvfs("/")
print("Total Size: %s" % (st.f_blocks * st.f_bsize))
volume.umount()
```

## Approach 3 - Gluster CLI

Volume status detail command already provides brick-level
utilization. Combine status detail with Volume info to group the
bricks into subvolumes. Get the storage reserve by running `gluster
volume get <volname> storage.reserve` and subtract the same from
`size_free` or add to `size_used`.

Refer Posix storage reserve feature
[here](https://github.com/gluster/glusterfs/issues/236)

Once we get the sub volume grouping, get the sub volume utilization
from the bricks utilization as follows.

```
volume_capacity_total = 0
volume_capacity_used = 0
volume_inodes_total = 0
volume_inodes_used = 0
for subvol in subvols {
    volume_capacity_total += subvol.capacity_total
    volume_capacity_used += subvol.capacity_used
    volume_inodes_total += subvol.inodes_total
    volume_inodes_used += subvol.inodes_used
}
```

Subvolume utilization is dependent on its type. If subvolume type is
Replicate then take the maximum of `capacity_used` of all bricks in
that subvolume and take a minimum of `capacity_total` of all
bricks. This method is because all replica bricks will have the same
data and total capacity of a subvolume is dependent on the brick
having least capacity (In most cases capacity of all bricks in
subvolume will be identical).

In case of subvolume type is Disperse Volume then,

```
subvol_capacity_total = any_one_brick_capacity_total * number_of_data_bricks
subvol_capacity_used = any_one_brick_capacity_used * number_of_data_bricks
subvol_inodes_total = any_one_brick_inode_total
subvol_inodes_used = any_one_brick_inode_used
```

## Conclusion:

- Results of all approaches look similar, feel free to use the method
  comfortable to the use case.
- Mount cleanup required in case of first and second approaches. If
  the application crashes before unmounting the volume, then we may
  end up in stale mounts.
- If the application is collecting the utilization details in regular
  interval, either mount needs to be persisted or unmount is required
  after collecting each time. Mounting and unmounting steps will
  become too cumbersome to manage. Since mounting all volumes will
  consume extra memory and mount can't persist if every time
  utilization collected from different nodes.
- Approach 3 is straightforward to implement since it is just running
  the CLI command and parsing its output to get the required
  results. The only downside of this approach is, the aggregation
  logic implemented in the application side. So the application can go
  out of sync if Gluster changes the way to calculate the utilization
  then application.
- Directory utilization is not possible with approach 3. If Gluster
  Quota feature used and utilization of a subdirectory required, then
  it is not possible using Gluster CLI commands.
- Gluster status detail command itself can subtract the storage
  reserve instead of returning the raw value(Bug: To be opened)


