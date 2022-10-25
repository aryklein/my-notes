# Manage virtual machines with virt-manager

The virt-manager application is a desktop user interface for managing virtual machines through libvirt.
It primarily targets KVM VMs.

## Install virt-manager on Arch Linux

To install virt-manager on Linux you just need to execute:

```bash
sudo pacman -S virt-manager dnsmasq qemu-img qemu-system-x86
```
`virt-manager` uses `libvirt` so you need to start `libvirtd.service` systemd unit.

**Note**: virt-manager uses dnsmasq in NAT network mode but you don't need to start the `dnsmasq`
service.

I prefer to keep it disabled at boot time and only start the service when I'm going to use it:

```bash
sudo systemctl start libvirtd.service
```

The easiest way to ensure your user has access to libvirt daemon is to add member to libvirt user group.

Members of the libvirt group have passwordless access to the RW daemon socket by default.

```bash
sudo usermod -a -G libvirt <user>
```
## Running VMs

If you use the NAT network mode, you will need to define and start a network in libvirt. By default
`libvirt` has a network defined in `/etc/libvirt/qemu/networks/default.xml`. You can use it by running:

```bash
sudo virsh
```

Then in the `virsh` CLI you can start it by:

```
virsh # net-start default
```

Alternatively, if you want to have it enabled by default every time you start the `libvirtd` service,
you can run:

```
virsh # net-start default
virsh # net-autostart default
```
