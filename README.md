# oracle_rhel_tunning
A list of recommended practices to tune your RHEL system for Oracle workloads.

###### [Disabling_NUMA](https://access.redhat.com/solutions/23216)

NUMA may be required when running a specific type of virtual machine such as one built running on HPE Superdome. Prior to building the OS, you should work with the hardware vendor to receive the recommended settings for your OS version & type of workload.

To disable NUMA in *rhel 7* take the following steps:

**NOTE**: `you should always perform a full OS backup prior to any changes`

- Step 1.) 
