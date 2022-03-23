# Manage virtual machines with virt-manager

The virt-manager application is a desktop user interface for managing virtual machines through libvirt.
It primarily targets KVM VMs.

## Install virt-manager on Arch Linux

To install virt-manager on Linux you just need to execute:

```bash
sudo pacman -S virt-manager
```
`virt-manager` uses `ibvirt` so you need to start `libvirtd.service` systemd unit.

I prefer to keep it disabled at boot time and only start the service when I'm going to use it:

```bash
systemd start libvirtd.service
```

The easiest way to ensure your user has access to libvirt daemon is to add member to libvirt user group.

Members of the libvirt group have passwordless access to the RW daemon socket by default.

```bash
usermod -a -G libvirt <user>
```
