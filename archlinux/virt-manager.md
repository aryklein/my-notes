# Manage virtual machines with virt-manager

The virt-manager application is a desktop user interface for managing virtual machines through libvirt.
It primarily targets KVM VMs.

## Install virt-manager on Arch Linux

To install virt-manager on Linux you just need to execute:

```bash
sudo pacman -S virt-manager dnsmasq
```
`virt-manager` uses `ibvirt` so you need to start `libvirtd.service` systemd unit.

**Note**: virt-manager uses dnsmasq in NAT network mode but you don't need to start the `dnsmasq`
service.

I prefer to keep it disabled at boot time and only start the service when I'm going to use it:

```bash
sudo systemd start libvirtd.service
```

The easiest way to ensure your user has access to libvirt daemon is to add member to libvirt user group.

Members of the libvirt group have passwordless access to the RW daemon socket by default.

```bash
sudo usermod -a -G libvirt <user>
```
## Running VMs

In NAT network mode, you need to define and start a network in libvirt. By default `libvirt` has a
network defined in `/etc/libvirt/qemu/networks/default.xml`. You can use it by running:

```bash
sudo virsh
virsh # net-start default
```
