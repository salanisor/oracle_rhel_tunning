##### Manual Steps

  - Step 1) Enable the optional RPMs repository to install the Oracle tuned profile.

```
subscription-manager repos --enable='rhel-7-server-optional-rpms'
```

  - Step 2) Install the tuned packages required to enable the Oracle tuned profile.

```
yum install tuned-profiles-oracle tuned -y
```

  - Step 3) Set the tuned-profiles-oracle profile

```
tuned-adm profile oracle
```

  - Step 4) Make backup of `/etc/default/grub`

```
cp /etc/default/grub /etc/default/grub-`date +%Y%m%d%s`
```

  - Step 5) Append the following text `numa=off transparent_hugepage=never` to the end of line - starting with: `GRUB_CMDLINE_LINUX=` in file `/etc/default/grub`

  - Step 6) Update the bootloader config from the previous update to `/etc/default/grub`

```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

  - Step 7) Reboot the server to pick up the additional kernel parameters from the previous update.

  - Step 8) Get the active tuned profile to verify changes.

```
tuned-adm active
```

Expected output:

```
Current active profile: oracle
```

  - Step 9)

```
egrep -i 'numa|transparent' /proc/cmdline
```

Expected output:
```
BOOT_IMAGE=/vmlinuz-3.10.0-1160.15.2.el7.x86_64 root=/dev/mapper/rhel_rhel7-root ro crashkernel=auto spectre_v2=retpoline rd.lvm.lv=rhel_rhel7/root rd.lvm.lv=rhel_rhel7/swap rhgb quiet numa=off transparent_hugepage=never
```
