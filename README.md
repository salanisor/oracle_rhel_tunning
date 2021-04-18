# oracle_rhel_tunning
A list of recommended practices to tune your RHEL system for Oracle workloads.

#### [Disabling_NUMA](https://access.redhat.com/solutions/23216)

NUMA may be required when running a specific type of virtual machine such as one built running on HPE Superdome. Prior to building the OS, please consult with your hardware vendor to receive the recommended settings for your OS version & workload.

To disable NUMA and transparent huge pages in *rhel 7* take the following steps:

**NOTE**: `you should always perform a full OS backup prior to any changes`

- Step 1.) Disable Numa by adding the command line option `numa=off` to grub2 config.
- Step 2.) Disable transparent huge pages by adding the command line option `transparent_hugepage=never` to grub2 config.
- Step 3.) Rebuild the grub2 menu.
- Step 4.) Reboot system to have settings take affect.
- Step 5.) Verify settings.


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

Expected output:
```
BOOT_IMAGE=/vmlinuz-3.10.0-1160.15.2.el7.x86_64 root=/dev/mapper/rhel_rhel7-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=rhel_rhel7/root rd.lvm.lv=rhel_rhel7/swap rhgb quiet numa=off transparent_hugepage=never
``` 
