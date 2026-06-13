---
title:  "debian trixie luks unlock with dropbear-initramfs"
categories: debian
---

I have a lot of computers with luks encrypted root drives
Setting them up to allow ssh unlock was less easy than could be hoped


the [debian wiki](https://wiki.debian.org/DropBear) has some information, but it's not complete, and is overly complicated in a few ways

here is what I did:

```
apt install dropbear-initramfs
```


have the kernel do dhcp on boot (docs [online](https://www.kernel.org/doc/Documentation/filesystems/nfs/nfsroot.txt) or at `/usr/share/doc/linux-doc/Documentation/admin-guide/nfs/nfsroot.rst.gz`:
`/etc/initramfs-tools/initramfs.conf`:
```
IP=dhcp
```


configure dropbear
`/etc/dropbear/initramfs/dropbear.conf`:
```
# -I 180 : Disconnect the session if no traffic is transmitted or received in 180 seconds.
# -j : Disable ssh local port forwarding.
# -k : Also disable remote port forwarding.
# -c cryptroot-unlock : Disregard the command provided by the user and always run forced_command.
DROPBEAR_OPTIONS="-I 180 -j -k -c cryptroot-unlock"
```

setup the dropbear keys to be the same as the system keys, and use the user's authorized_keys as the dropbear authorized_keys
```
sudo rm /etc/dropbear/initramfs/dropbear_*
sudo dropbearconvert openssh dropbear /etc/ssh/ssh_host_ecdsa_key /etc/dropbear/initramfs/dropbear_ecdsa_host_key
sudo dropbearconvert openssh dropbear /etc/ssh/ssh_host_rsa_key /etc/dropbear/initramfs/dropbear_rsa_host_key
sudo ln -s $HOME/.ssh/authorized_keys /etc/dropbear/initramfs/
```

update initramfs
```
update-initramfs -c -k all
```
