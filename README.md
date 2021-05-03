# oracle_rhel_tunning
A list of recommended practices to tune your RHEL system for Oracle workloads.

#### [Disabling_NUMA](https://access.redhat.com/solutions/23216)

NUMA may be required when running a specific type of virtual machine such as one built running on HPE Superdome. Prior to building the OS, please consult with your hardware vendor to receive the recommended settings for your OS version & workload.

To disable NUMA and transparent huge pages in *rhel 7* take the following steps:

**NOTE**: `You should always perform a full OS backup prior to any changes`

- Step 1.) Disable Numa by adding the command line option `numa=off` to grub2 config.
- Step 2.) Disable transparent huge pages by adding the command line option `transparent_hugepage=never` to grub2 config.
- Step 3.) Install and set the tuned profile to `oracle`
- Step 4.) Rebuild the grub2 menu.
- Step 5.) Reboot system to have settings take affect.
- Step 6.) Verify settings.


#### How to run playboook

On the local DB host execute ansible.

```
ansible-playbook -vvv prep_oracle_sys.yaml
```

#### [Verify reboot](https://access.redhat.com/solutions/4968001)

To verify that `numa=off` & `transparent_hugepage=never` settings took effect run the following command:


```
cat /proc/cmdline 
```

Expected output example:

```
BOOT_IMAGE=/vmlinuz-3.10.0-1160.15.2.el7.x86_64 root=/dev/mapper/rhel_rhel7-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=rhel_rhel7/root rd.lvm.lv=rhel_rhel7/swap rhgb quiet numa=off transparent_hugepage=never
``` 

#### [Oracle tuned profile](https://www.mankier.com/7/tuned-profiles-oracle)


`/usr/lib/tuned/oracle/tuned.conf`

#### tuned configuration
```
[main]
summary=Optimize for Oracle RDBMS
include=throughput-performance

[sysctl]
```
Swappiness is defined as a value from `0` to `100` which controls the degree to which the system
favors anonymous memory or the page cache. A high valueimproves file-system performance, while
aggressively swapping less active processes out of memory. A low value avoids swapping processes
out of memory, which usually decreases latency, at the cost of I/O performance.
  
Swapping for Oracle is not ideal and should be avoided as much as possible. The following tunable
will tune the kernel to swap less aggressively.
```
vm.swappiness = 10
```
Contains, as a percentage of total system memory, the number of pages at which the background write
back daemon will start writing out dirty data. The Oracle recommended value is `3`.
```
vm.dirty_background_ratio = 3
```
Contains, as a percentage of total system memory, the number of pages at which a process which is
generating disk writes will itself start writing out dirty data. The default value is `20`. The recommended
value is between `40` and `80`. The reasoning behind increasing the value from the standard Oracle `15`
recommendation to a value between `40` and `80` is because dirty ratio defines the maximum percentage of total
memory that be can be filled with dirty pages before user processes are forced to write dirty buffers themselves
during their time slice instead of being allowed to do more writes. All processes are blocked for writes
when this occurs due to synchronous I/O, not just the processes that filled the write buffers. This can
cause what is perceived as unfair behavior where a single process can hog all the I/O on a system. As the value
of dirty_ratio is increased, it is less likely that all processes will be blocked due to synchronous I/O,
however, this allows for more data to be sitting in memory that has yet to be written to disk. As for all
parameters in this reference architecture, there is no “one-size fits all” value and the recommendation
should be only seen as a starting point.
```
vm.dirty_ratio = 40
```
Defines when dirty in-memory data is old enough to be eligible for writeout. The default value is `3000`,
expressed in hundredths of a second. The Oracle recommended value is `500`.
```
vm.dirty_expire_centisecs = 500
```
Defines the interval of when writes of dirty in-memory data are written out to disk. The default value is `500`,
expressed in hundredths of a second. The Oracle recommended value is `100`.
```
vm.dirty_writeback_centisecs = 100
```
Defines the maximum size (in bytes) of a single shared memory segment allowed by the kernel.
```
kernel.shmmax = 4398046511104
```
Defines the total amount of shared memory pages that can be used on the system at one time. A
page is `4096` bytes on the AMD64 and Intel 64 architecture.
```
kernel.shmall = 1073741824
```

**SHMMNI** is the maximum total amount of shared memory segments. A default installation of
Red Hat Enterprise Linux 7 x86_64 provides a SHMMNI default value of `4096`. By optimizing the
**SHMMAX** value with one shared memory segment per Oracle SGA, this parameter reflects the
maximum number of Oracle and ASM instances that can be started on a system. Oracle
recommends the value of **SHMMNI** to be left at the default value of `4096`.
```
kernel.shmmni = 4096
```
Red Hat Enterprise Linux 7 provides semaphores for synchronization of information between
processes. The kernel parameter sem is composed of four parameters:

**SEMMSL** – is defined as the maximum number of semaphores per semaphore set
**SEMMNI** – is defined as the maximum number of semaphore sets for the entire system
**SEMMNS** – is defined as the total number of semaphores for the entire system

**NOTE**: SEMMNS is calculated by SEMMSL * SEMMNI – is defined as the total number of semaphore operations
performed per semop system call.

**SEMOPM** – is defined as the total number of semaphore operations performed per semop
system call.

The following line is required within the /etc/sysctl.conf file to provide sufficient semaphores
for Oracle:
```
kernel.sem = 250 32000 100 128
```
While the value of the **FS.FILE-MAX** parameter varies upon every environment, this reference
environment sets the value at `6815744`. Oracle recommends a value no smaller than
`6815744`. Due to the calculation in the above example equating to 5050858, the minimum
Oracle recommended value is used. In order to add **FS.FILE-MAX** to `6815744`, modify the
`/etc/sysctl.conf` file on each node of the Oracle RAC cluster as follows:
```
fs.file-max = 6815744
```

The kernel parameter **FS.AIO-MAX** - NR sets the maximum number of current asynchronous I/O
requests. Oracle recommends setting the value to `1048576`. In order to add **FS-AIO-MAX-NR** to
`1048576`, modify the `/etc/sysctl.conf` file on each node of the Oracle RAC cluster as follows:


```
fs.aio-max-nr = 1048576
```

Oracle recommends that the ephemeral default port range be set starting at `9000` to `65500`.
This ensures that all well known ports used by Oracle and other applications are avoided. To
set the ephemeral port range, modify the `/etc/sysctl.conf` file on each node of the Oracle RAC
Database cluster and add the following line.
```
net.ipv4.ip_local_port_range = 9000 65499
```

Optimizing the network settings for the default and maximum buffers for the application
sockets in Oracle is done by setting static sizes to **RMEM** and **WMEM** . The **RMEM** parameter
represents the receive buffer size, while the WMEM represents the send buffer size. The
recommended values by Oracle are configured within the /etc/sysctl.conf file on each node of
the Oracle RAC cluster.
```
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```
The kernel parameter `kernel.panic_on_oops` controls the kernel's behavior when an oops or
bug is encountered. By default the value is set to `1`, however, the oracle installer requires it to
be set within the `/etc/sysctl.conf` file.
```
kernel.panic_on_oops = 1
```

```
[vm]
transparent_hugepages=never
```
