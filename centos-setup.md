# CentOS Setup

## Install CentOS with a flash drive
Remember to add the user to administrator group when create an user.

## Setup network

1. List ethernet card
    ```bash
    nmcli d
    ```

2. Open Network manager and edit connection.
    ```bash
    nmtui
    ```
    Choose “Automatic” in IPv4 CONFIGURATION and check Automatically connect check box and press OK and quit from Network manager.

3. Reset network service
    ```bash
    service network restart
    ```
    
## Setup UI (if CentOS minimal)

1. List available package groups
    ```bash
    yum group list
    ```
    
2. Install Gnome GUI packages
    ```bash
    yum groupinstall "GNOME Desktop" "Graphical Administration Tools"
    ```

3. Enable GUI on system startup
    ```bash
    ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target
    ```

4. Reboot the machine to start the server in the graphical mode
    ```bash
    reboot
    ```

## Bridge network

1. Disabling NetworkManager
    ```bash
    chkconfig NetworkManager off
    chkconfig network on
    service NetworkManager stop
    service network start
    ```

2. Open ifcfg-em1 (or ifcfg-eth0 depending on what ethernet card you have) in /etc/sysconfig/network-scripts. Make the following changes.

    ```bash
    #TYPE=Ethernet
    PROXY_METHOD=none
    BROWSER_ONLY=no
    #BOOTPROTO=dhcp <-------------It is very important to change dhcp to static
    BOOTPROTO=static
    #DEFROUTE=yes
    #IPV4_FAILURE_FATAL=no
    #IPV6INIT=yes
    #IPV6_AUTOCONF=yes
    #IPV6_DEFROUTE=yes
    #IPV6_FAILURE_FATAL=no
    #IPV6_ADDR_GEN_MODE=stable-privacy
    NAME=em1
    UUID=7c87469d-30c3-453f-854a-104142f03bee
    DEVICE=em1
    ONBOOT=yes
    BRIDGE=br0   <---------------This line is also very important
    ```

3. Modify/Create ifcfg-br0 in the same folder to define bridge device

    ```bash
    DEVICE="br0"
    BOOTPROTO=static
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    ONBOOT=yes
    TYPE=Bridge
    DELAY=0
    IPADDR=192.168.29.190
    NETMASK=255.255.255.0
    GATEWAY=192.168.29.1
    ```

4. Restart networking

    ```bash
    service network restart
    ```

5. Check shared physical device

    ```bash
    brctl show
    ```

    A table that similar to the following should show up

    | bridge name |     bridge id     | STP enabled | interfaces |
    | ----------- | ----------------- | ----------- | ---------- |
    | br0         | 8000.f8b156c47f7d | no          | em1        |
    |             |                   |             | vnet0      |
    |             |                   |             | vnet1      |
    | virbr0      | 8000.5254006ad49e | yes         | virbr0-nic |

## Setup VM

1. Install KVM
    
    ```bash
    yum install qemu-kvm libvirt libvirt-python libguestfs-tools virt-install
    ```

2. Start the libvirtd service

    ```bash
    systemctl enable libvirtd
    systemctl start libvirtd
    ```

3. Verify kvm installation. Make sure KVM module loaded using lsmod command and grep command.

    ```bash
    lsmod | grep -i kvm
    ```

### Create VM via commandline

1. Direct to /var/lib/libvirt/images, and download CentOS image from web.

    ```bash
    cd /var/lib/libvirt/images
    wget http://mirror.trouble-free.net/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1804.iso
    ```

2. Use <code>virt-install</code> to create VM

    ```bash
    virt-install \
    --virt-type=kvm \
    --name centos7 \
    --ram 2048 \
    --vcpus=1 \
    --os-variant=centos7.0 \    
    --cdrom=/var/lib/libvirt/boot/CentOS-7-x86_64-Minimal-1804.iso \
    --network=bridge=br0,model=virtio \
    --graphics vnc \
    --disk path=/var/lib/libvirt/images/centos7.qcow2,size=40,bus=virtio,format=qcow2
    ```

### Create VM via virt-manager

1. Install virt-manager

    ```bash
    yum install virt-manager
    ```

2. Start virt-manager and create VM

    ```bash
    virt-manager
    ```


## References
Setup network: https://lintut.com/how-to-setup-network-after-rhelcentos-7-minimal-installation/

Setup UI:https://www.itzgeek.com/how-tos/linux/centos-how-tos/install-gnome-gui-on-centos-7-rhel-7.html

Bridge network:
https://wiki.libvirt.org/page/Networking#Bridged_networking_.28aka_.22shared_physical_device.22.29

Create VM:
https://www.cyberciti.biz/faq/how-to-install-kvm-on-centos-7-rhel-7-headless-server/
https://www.linuxtechi.com/install-kvm-hypervisor-on-centos-7-and-rhel-7/

