# Remote Unlock

## Use case
* If you install Ubuntu 20.04 server, you can choose to lock your disk by a password. However, this will let OpenSSH server run only after the disk being decrypted. Therefore, this repo will demonstrate how to enable an SSH server by Dropbear even if your disk is still encrypted.

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
(Server side)
* `apt-get install dropbear`: From my experience, by install this, `dropbear` and `dropbear-initramfs` will also be installed.
* Add your `id_rsa.pub` to `/etc/dropbear-initramfs/authorized_keys` (i.e. `cat id_rsa.pub >> /etc/dropbear-initramfs/authorized_keys`)
* `update-initramfs -u`
* You are good to go!

If you want to set a static ip with a gateway:
* add line like this in `/etc/initramfs-tools/initramfs.conf`:
```
IP=192.168.11.111::192.168.11.254:255.255.255.0::eth0:off

```
* Note the `IP` format is: `IP=<client-ip>:<server-ip>:<gw-ip>:<netmask>:<hostname>:<device>:<autoconf>`
   <dns0-ip>:<dns1-ip>:<ntp0-ip>
* Remember to update the initramfs: `update-initramfs -u`

(Client side)
* `ssh -i ~/.ssh/id_rsa <your host>`
* `cryptroot-unlock`: this will prompt you to type your password for decrypting the disk.

## Troubleshooting
* I found that after `initramfs` the `IP=` will still remain even after the boot.
    * Check `/run/netplan/eno1.yaml`, I found that `IP` truly was added.
    * The way to solve it is to add a script under `/etc/initramfs-tools/scripts/init-bottom/`. And add commands like: `rm -f /run/netplan/*.yml`. You will find that `IP=` will not remain after `init` ends.
    * Remember to run `update-initramfs -u`!

## References
* <https://unix.stackexchange.com/questions/411945/luks-ssh-unlock-strange-behaviour-invalid-authorized-keys-file>: The most useful one
* <https://unix.stackexchange.com/questions/5017/ssh-to-decrypt-encrypted-lvm-during-headless-server-boot>
* <https://stinkyparkia.wordpress.com/2014/10/14/remote-unlocking-luks-encrypted-lvm-using-dropbear-ssh-in-ubuntu-server-14-04-1-with-static-ipst/>
* `man initramfs.conf`
* `man initramfs-tools`
* <https://unix.stackexchange.com/questions/550021/where-is-the-documentation-for-the-ip-variable-in-initramfs-conf>
* <https://www.eugenemdavis.com/set-static-ip-initramfs.html>
* [What is creating /run/netplan/eth0.yaml?]( https://askubuntu.com/questions/1228433/what-is-creating-run-netplan-eth0-yaml ): `netplan` and `initramfs-tools`
    * Thanks for the hint. I ended up creating `/etc/initramfs-tools/scripts/init-bottom/deconfigure-interfaces` that does `rm -f /run/netplan/eth0.yaml && ip -f inet address flush dev eth0`. (I don't know if flushing the IP addresses is necessary, but it doesn't hurt.)
