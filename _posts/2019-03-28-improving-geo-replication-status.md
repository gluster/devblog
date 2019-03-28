---
layout: post
title:  Improving Geo-replication Status
author: aravindavk
---

[Geo-replication](https://docs.gluster.org/en/latest/Administrator%20Guide/Geo%20Replication/)
is a feature in Glusterfs to sync data from one Gluster Volume to
another.

Current Geo-replication status has the following limitations.

- Order of bricks is not uniform. Glusterd collects the status details
  from all other nodes and displays in random order. This is very
  confusing to users. Order of bricks in status should be in the same
  order as in Volume info.
- If initiator Glusterd(where the command is run) couldn't reach other
  node's Glusterd then current CLI is skipping that in the status
  output. Status should be shown as "Unknown" without spoiling the
  order.
- Geo-rep status mixes the rows from multiple sessions depending on
  the response received from other node's Glusterds.

Until the above issues are fixed in official Gluster CLI, we can use
`gluster-georep-status` tool from
[gluster-tools](https://github.com/aravindavk/gluster-georep-tools)
project. This tool merges Volume info and Geo-rep status to identify
the Offline status and bricks order.

Installation instructions are available in project
[README](https://github.com/aravindavk/gluster-georep-tools/blob/master/README.md)

This tool also supports filters like `--with-status=active`,
`--with-crawl-status=changelog` etc.

Example output:

```
$ gluster-georep-status
SESSION: photos ==> bkp1.example.com::backup_photos
+-------------------------------------+---------+-----------------+---------------------+---------------------+------------+-----------------+-----------------------+
| MASTER                              | STATUS  | CRAWL STATUS    | SLAVE NODE          | LAST SYNCED         | CHKPT TIME | CHKPT COMPLETED | CHKPT COMPLETION TIME |
+-------------------------------------+---------+-----------------+---------------------+---------------------+------------+-----------------+-----------------------+
| prod1.example.com:/bricks/photos/b1 | Active  | Changelog Crawl | remote1.example.com | 2019-03-27 08:34:40 |    N/A     |       N/A       |          N/A          |
| prod2.example.com:/bricks/photos/b2 | Passive | N/A             | N/A                 | N/A                 |    N/A     |       N/A       |          N/A          |
| prod3.example.com:/bricks/photos/b3 | Passive | N/A             | N/A                 | N/A                 |    N/A     |       N/A       |          N/A          |
+-------------------------------------+---------+-----------------+---------------------+---------------------+------------+-----------------+-----------------------+

SESSION: data ==> geoaccount@bkp2.example.com::backup_data
+-----------------------------------+---------+-----------------+---------------------+---------------------+------------+-----------------+-----------------------+
| MASTER                            | STATUS  | CRAWL STATUS    | SLAVE NODE          | LAST SYNCED         | CHKPT TIME | CHKPT COMPLETED | CHKPT COMPLETION TIME |
+-----------------------------------+---------+-----------------+---------------------+---------------------+------------+-----------------+-----------------------+
| prod1.example.com:/bricks/data/b1 | Stopped | N/A             | N/A                 | N/A                 |    N/A     |       N/A       |          N/A          |
| prod2.example.com:/bricks/data/b2 | Stopped | N/A             | N/A                 | N/A                 |    N/A     |       N/A       |          N/A          |
| prod3.example.com:/bricks/data/b3 | Stopped | N/A             | N/A                 | N/A                 |    N/A     |       N/A       |          N/A          |
+-----------------------------------+---------+-----------------+---------------------+---------------------+------------+-----------------+-----------------------+
```

Comments & Suggestions Welcome.
