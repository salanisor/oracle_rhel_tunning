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

```
#
# tuned configuration
#

[main]
summary=Optimize for Oracle RDBMS
include=throughput-performance

[sysctl]

# Swappiness is defined as a value from 0 to 100 which controls the degree to which the system favors anonymous memory or the page cache. A high value
improves file-system performance, while aggressively swapping less active processes out of memory. A low value avoids swapping processes out of memory, which usually decreases latency, at the cost of I/O performance.
  
# Swapping for Oracle is not ideal and should be avoided as much as possible. The following tunable will tune the kernel to swap less aggressively.

vm.swappiness = 10

# Contains, as a percentage of total system memory, the number of pages at which the background write back daemon will start writing out dirty data. The Oracle recommended value is 3.

vm.dirty_background_ratio = 3

# Contains, as a percentage of total system memory, the number of pages at which a process which is generating disk writes will itself start writing out dirty data. The default value is 20. The recommended value is between 40 and 80. The reasoning behind increasing the value from the standard Oracle 15 recommendation to a value between 40 and 80 is because dirty ratio defines the maximum percentage of total memory that be can be filled with dirty pages before user processes are forced to write dirty buffers themselves during their time slice instead of being allowed to do more writes. All processes are blocked for writes when this occurs due to synchronous I/O, not just the processes that filled the write buffers. This can cause what is perceived as unfair behavior where a single process can hog all the I/O on a system. As the value of dirty_ratio is increased, it is less likely that all processes will be blocked due to synchronous I/O, however, this allows for more data to be sitting in memory that has yet to be written to disk. As for all parameters in this reference architecture, there is no “one-size fits all” value and the recommendation should be only seen as a starting point.

vm.dirty_ratio = 40
vm.dirty_expire_centisecs = 500
vm.dirty_writeback_centisecs = 100
kernel.shmmax = 4398046511104
kernel.shmall = 1073741824
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
fs.file-max = 6815744
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65499
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
kernel.panic_on_oops = 1

[vm]
transparent_hugepages=never
```
