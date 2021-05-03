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

  - Step 8) Get active tuned profile to verify changes.

```
tuned-adm active
```

Expected output:

```
Current active profile: oracle
```
