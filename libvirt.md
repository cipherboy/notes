# Renewing DHCP Address

After cloning new VMs in quick succession, best to nuke the old dnsmasq
configs in `/var/lib/libvirt/dnsmasq/vibr0.status`, `killall dnsmasq` and
restart `libvird`. Reboot VMs and this should fix it.
