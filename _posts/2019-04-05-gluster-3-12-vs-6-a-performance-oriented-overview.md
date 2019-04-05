---
layout: post
title: Gluster 3.12 vs 6 - A performance oriented overview
author: ezio-auditore 
---
**Gluster 3.12 vs 6 - A performance oriented overview**

With the release of gluster 6 I decided to have a performance benchmark test with gluster 3.12 version to see if there was really any improvement seen. Used [pg-bench](https://www.postgresql.org/docs/10/pgbench.html) tool to measure the performance. 
 
PG-Bench is a simple program for running benchmark tests on PostgreSQL. It runs the same sequence of SQL commands over and over, possibly in multiple concurrent database sessions, and then calculates the average transaction rate (transactions per second). By default, pgbench tests a scenario that is loosely based on TPC-B, involving five `SELECT`, `UPDATE`, and `INSERT` commands per transaction

The methods I used to measure them are very simple. For those who want to know stick around others can jump straight to the results.

I started with installing gluster 3.12 hyper-converged  environment on three bare metal machines using the cockpit-ovirt dashboard. I continued with installing hosted-engine managing the cluster. After the initial setup was complete I created 2 vms, one having disk allocation policy as "Preallocated" and the other as "Thin provisioned".Once the vms were up it was time to setup pgbench.

**Bare metal machine specs** 

OS  : Red Hat Enterprise Linux Server release 7.7 Beta (Maipo)

Processor: 	Intel(R) Xeon(R) CPU E5-1620 v4 @ 3.50GHz

Execution Technology : 4 cores; 8 threads

**Storage specs**

Model	Smart HBA H240

Firmware Version	4.52

Controller Type	HPE Smart Array

Media Type	SSD

Capacity	400 GB

Following are the steps I did for setting pg bench :-



1. yum install [https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm](https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm)
2. yum install postgresql-contrib postgresql-server

Postgres stores its data files at `/var/lib/pgsql/data` thus on the two vms I mounted the device (/dev/sdb here) used for pgsql data. 



Following are the steps :-



1. `pvcreate /dev/sdb && vgcreate postgresql /dev/sdb && lvcreate -n pg_data -l 100%PVS postgresql /dev/sdb && mkfs.xfs /dev/postgresql/pg_data`

2. ` mount /dev/postgresql/pg_data /var/lib/pgsql/data`

3. ` chown postgres:postgres  /var/lib/pgsql/data`

4. ` postgresql-setup initdb`

5. ` vi /var/lib/pgsql/data/postgresql.conf `

shared_buffers=2048MB # approx 250 MB (1/4 memory of VM) 
effective_cache_size=4096MB # approx 250 MB (1/2 memory of VM) 
checkpoint_segments=8
6. ` systemctl restart postgresql`

Steps for preparing the test database:-



1. ` su - postgres`
2. `bash-4.2$ createdb pgbench`
3. Creating a 60G database 

`bash-4.2$ time pgbench -i -s 4000 pgbench`

**Time required for creating the database (Gluster 3.12 vs Gluster 6) Thinly Provisioned**

`time pgbench -i -s 4000 pgbench`

**Thinly Provisioned**

<table border="1">
  <tr>
   <th>Gluster ver with/out options
   </th>
   <th>real
   </th>
  </tr>
  <tr>
   <td>&nbsp;3.12
   </td>
   <td>&nbsp;20m40.080s
   </td>
  </tr>
  <tr>
   <td>&nbsp;6 (without cache-invalidation)&nbsp;&nbsp;
   </td>
   <td>&nbsp;25m24.868s
   </td>
  </tr>
  <tr>
   <td>&nbsp;6 (defaults)
   </td>
   <td>&nbsp;25m56.818s
   </td>
  </tr>
</table>

**Time required for creating the database (Gluster 3.12 vs Gluster 6) Preallocated**

`time pgbench -i -s 4000 pgbench`

**Preallocated**


<table border="1" cellpadding="5">
  <tr>
   <th>Gluster ver with/out options
   </th>
   <th>real
   </th>
  </tr>
  <tr>
   <td>&nbsp;3.12
   </td>
   <td>&nbsp;20m30.481s&nbsp;
   </td>
  </tr>
  <tr>
   <td>&nbsp;6 (without cache-invalidation)&nbsp; &nbsp; 
   </td>
   <td>24m8.149s&nbsp;
   </td>
  </tr>
  <tr>
   <td>&nbsp;6 (defaults)
   </td>
   <td>25m45.599s&nbsp;
   </td>
  </tr>
</table>




![time-required-to-create-db](images/gluster-3.12vs-6/time-required-to-create-db.png "image_tooltip")


*Time taken to create the db seems to be more in gluster-6 still investigating why.*

Now let us look at some pg-bench results. Have created two disks *(Preallocated & Thinly-Provisioned)* . Added various parameters to check the performance. These volume options are set from ovirt-ui. Before running tests for stat-prefetch have run
`echo 3 > /proc/sys/vm/drop_caches && sync` on the hypervisor hosts.
 

For  running auto-invalidations:false test have set the mount options for the storage domains in the ovirt-ui.

Have run the following commands three times on the vms for both the disk types and captured the results.

`pgbench -c 10 -t 500 -r pgbench`

`pgbench -c 10 -t 1000 -r pgbench`

`pgbench -c 10 -t 2000 -r pgbench`

`pgbench -c 10 -t 4000 -r pgbench`

Arguments to pgbench as -c and -t where -c=number of clients , -t= number of transactions per client

The raw data for the below results are located in : [gluster-pg-bench-test-results](https://github.com/ezio-auditore/gluster-pg-bench-test-results)

<table border="1" cellpadding="10">
  <tr>
   <th>Thinly Provisioned disk
   </th>
   <th colspan="4" >Transactions per sec
   </th>
  </tr>
  <tr>
   <th>Gluster ver with/out options
   </th>
   <td>500
   </td>
   <td>1000
   </td>
   <td>2000
   </td>
   <td>4000
   </td>
  </tr>
  <tr>
   <td>3.12
   </td>
   <td>792.38&nbsp;&nbsp;
   </td>
   <td>432.49&nbsp;&nbsp;
   </td>
   <td>407.42&nbsp;&nbsp;
   </td>
   <td>353.82&nbsp;&nbsp;
   </td>
  </tr>
  <tr>
   <td>6 (cache-invalidation=false)
   </td>
   <td>902.91&nbsp;&nbsp;
   </td>
   <td>612&nbsp;&nbsp;
   </td>
   <td>562&nbsp;&nbsp;
   </td>
   <td>596.93&nbsp;&nbsp;
   </td>
  </tr>
  <tr>
   <td>6 (defaults)
   </td>
   <td>806.86&nbsp;&nbsp;
   </td>
   <td>729.71&nbsp;&nbsp;
   </td>
   <td>618.55&nbsp;&nbsp;
   </td>
   <td>526&nbsp;&nbsp;
   </td>
  </tr>
  <tr>
   <td>6 stat-prefetch:off
   </td>
   <td>544.63&nbsp;&nbsp;
   </td>
   <td>455.17&nbsp;&nbsp;
   </td>
   <td>443.49&nbsp;&nbsp;
   </td>
   <td>437.03&nbsp;&nbsp;
   </td>
  </tr>
  <tr>
   <td>6 auto-invalidations:false,stat-prefetch:off&nbsp;&nbsp;
   </td>
   <td>714.5&nbsp;&nbsp;
   </td>
   <td>623.94&nbsp;&nbsp;
   </td>
   <td>587.63&nbsp;&nbsp;
   </td>
   <td>524.3&nbsp;&nbsp;
   </td>
  </tr>
</table>

![txn-per-sec-thinly-provisioned](images/gluster-3.12vs-6/txn-per-sec-thinly-provisioned.png "txn-per-sec-thinly-provisioned")
<table border="1">
  <tr>
   <th>Preallocated
   </th>
   <th colspan="4" >Transactions per sec
   </th>
  </tr>
  <tr>
   <th>Gluster ver with/wout options
   </th>
   <td>500
   </td>
   <td>1000
   </td>
   <td>2000
   </td>
   <td>4000
   </td>
  </tr>
  <tr>
   <td>3.12
   </td>
   <td>522.46 &nbsp; &nbsp;
   </td>
   <td>646.91 &nbsp;&nbsp;
   </td>
   <td>524.79&nbsp;&nbsp;
   </td>
   <td>650.52&nbsp;&nbsp;
   </td>
  </tr>
  <tr>
   <td>6 (cache-invalidations=false)
   </td>
   <td>1180.74&nbsp;&nbsp;
   </td>
   <td>1021.76&nbsp;&nbsp;
   </td>
   <td>890.18&nbsp;&nbsp;
   </td>
   <td>995&nbsp;
   </td>
  </tr>
  <tr>
   <td>6 defaults
   </td>
   <td>1044.89
   </td>
   <td>785.81
   </td>
   <td>725.06
   </td>
   <td>727.44
   </td>
  </tr>
  <tr>
   <td>6 stat-prefetch:off
   </td>
   <td>937.31
   </td>
   <td>783.92
   </td>
   <td>761.32
   </td>
   <td>741.33
   </td>
  </tr>
  <tr>
   <td>6 auto-invalidations:false,stat-prefetch:off&nbsp;
   </td>
   <td>1100.84
   </td>
   <td>793.44
   </td>
   <td>757.88
   </td>
   <td>727.38
   </td>
  </tr>
</table>
![txn-per-sec-preallocated.png](images/gluster-3.12vs-6/txn-per-sec-preallocated.png "txn-per-sec-preallocated")

There is indeed a performance improvement seen with gluster-6 for certain workloads especially without cache-invalidations, although there is a ~12% performance drop while creating the db.




Hope this data helps in understanding the various improvements and bottlenecks seen in the gluster-6. I am also working on making these tests automated for running  with each new gluster release. Comments and suggestions are welcome.