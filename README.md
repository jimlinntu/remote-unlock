# Remote Unlock

## Use case
* If you install Ubuntu 20.04 server, you can choose to lock your disk by a password. However, this will let OpenSSH server run only after the disk being decrypted. Therefore, this repo will demonstrate how to enable a SSH server by Dropbear even if your disk is still encrypted.

## Environment
* `uname -a`:
```
Linux <machine name> 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```
* `lsb_release -a`:
```
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.1 LTS
Release:	20.04
Codename:	focal
```

## Installation
* `apt-get install dropbear`: From my experience, by install this, `dropbear` and `dropbear-initramfs` will also be installed.
* Add your `id_rsa.pub` to `/etc/dropbear-initramfs/authorized_keys` (i.e. `cat id_rsa.pub >> /etc/dropbear-initramfs/authorized_keys`)
* `update-initramfs -u`
* You are good to go!

## References
* <https://unix.stackexchange.com/questions/411945/luks-ssh-unlock-strange-behaviour-invalid-authorized-keys-file>: The most useful
* <https://unix.stackexchange.com/questions/5017/ssh-to-decrypt-encrypted-lvm-during-headless-server-boot>
* <https://stinkyparkia.wordpress.com/2014/10/14/remote-unlocking-luks-encrypted-lvm-using-dropbear-ssh-in-ubuntu-server-14-04-1-with-static-ipst/>
