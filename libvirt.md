# Renewing DHCP Address

After cloning new VMs in quick succession, best to nuke the old dnsmasq
configs in `/var/lib/libvirt/dnsmasq/vibr0.status`, `killall dnsmasq` and
restart `libvird`. Reboot VMs.

Also, make sure to update `/etc/machine-id` for `systemd-networkd`-based
DHCP on the guest:

    dd if=/dev/urandom bs=128 count=1 | md5sum | awk '{print $1}' > /etc/machine-id

Then restart `systemd-networkd`.
