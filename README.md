# Home Assistant Provisioning

This repository contains notes and hints on provisioning a
[Home Assistant](https://home-assistant.io) deployment.

## Provisioning a VM (libvirt/KVM)

### [Optional] Create a Home Assistant Network

A sample XML for a libvirt network definition is provided in
[homeasst0.xml](homeasst0.xml). To create a persistent network that is started
automatically on boot:

```bash
$ virsh net-define homeasst0.xml 
Network homeasst0 defined from homeasst0.xml
$ virsh net-autostart homeasst0
Network homeasst0 marked as autostarted
 $ virsh net-list
 Name              State    Autostart   Persistent
----------------------------------------------------
 homeasst0         active   yes         yes
```

### Download the `qcow2` image

Navigate to [this page](https://www.home-assistant.io/installation/linux) and
download the `KVM (.qcow2)` image.

After having downloaded and uncompressed (using `xz -d`) the
[qcow2 image](https://github.com/home-assistant/operating-system/releases/download/12.3/haos_ova-12.3.qcow2.xz)
the Home Assistant VM can be created using the command shown below.
However, before proceeding, make sure the image is readable to the `libvirt-qemu`
user.

```bash
$ sudo mv haos_ova-12.3.qcow2 /var/lib/libvirt/images/
```

### Create the VM using predefined network

In this example, `virt-install` is using the network defined above.

```bash
$ virt-install --name haos \
               --description "Home Assistant OS" \
               --os-variant=generic \
               --ram=4096 \
               --vcpus=2 \
               --disk /var/lib/libvirt/images/haos_ova-12.3.qcow2,bus=scsi \
               --controller type=scsi,model=virtio-scsi \
               --import \
               --graphics none \
               --network network=homeasst0,model=virtio \
               --boot uefi
```

### Create the VM using tun/tap (direct) network

`virt-install` allows VMs to "hook" into the host's physical interface using
tun/tap. to make use of this, `type=direct,source=<INTERFACE NAME>` should be
specified.

```bash
$ virt-install --name haos \
               --description "Home Assistant OS" \
               --os-variant=generic \
               --ram=4096 \
               --vcpus=2 \
               --disk /var/lib/libvirt/images/haos_ova-12.3.qcow2,bus=scsi \
               --controller type=scsi,model=virtio-scsi \
               --import \
               --graphics none \
               --network type=direct,source=enp1s0,source_mode=bridge,model=virtio \
               --boot uefi
```

### [Optional] Resize memory

Turns out the VM was using too much memory on a small dev laptop, so it got
downsized to 2G RAM:

```bash
$ virsh setmem haos 2048M --config
$ virsh setmaxmem haos 2048M --config
```

### Start the VM

Fire up the defined VM, if it's not yet running.

```bash
$ virsh start haos --console
```

Once started, the prompt will show 

```
Welcome to Home Assistant
homeassistant login: 
```

However, there are no users created, yet. Visit `<IP of VM>:8123` with your
browser and follow the prompts (as described in the [onboarding](https://www.home-assistant.io/getting-started/onboarding/) docs).


## References and Links

### Home Assistant
- [Home Assistant OS VM Image Download](https://www.home-assistant.io/installation/alternative#download-the-appropriate-image)
- [Home Assistant Releases](https://github.com/home-assistant/operating-system/releases/)
- [Provisioning a Home Assistant VM](https://www.home-assistant.io/installation/alternative#create-the-virtual-machine)
- [Home Assistant Onboarding (setup)](https://www.home-assistant.io/getting-started/onboarding/)
- [Home Assistant Concepts and Terminology](https://www.home-assistant.io/getting-started/concepts-terminology/)

### `libvirt`

- [Using `virt-install` with `direct` network option](https://gist.github.com/smurugap/163b3e2be7676a46c835339f8ba0710f)
- [Virtual NIC attachment modes](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/virtualization_administration_guide/sect-attch-nic-physdev#sect-attch-nic-physdev)
