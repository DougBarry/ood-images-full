# Setup

    git clone https://github.com/OSC/ood-images-full.git
    cd ood-images-full

## Vagrant

The RPM to use for OOD packages is at the top of the vagrant file. This is set to the latest version at time of writing (20210430).  

```URL_ONDEMANDRPM = "https://yum.osc.edu/ondemand/1.8/ondemand-release-web-1.8-1.noarch.rpm"```


Plugins are installed by the vagrant file:
- vagrant-vbguest
- vagrant-hosts
- vagrant-cachier

To launch and setup the VMs:
```bash
vagrant up
```

# Tested...

Working on Windows 10 host via Vagrant V2.2.16 on VirtualBox V6.1.18 @ 20210430

# Usage

Access to OpenOnDemand is via the `ood` user with password `ood`.

## Vagrant

Once the VM or container is online, the Open OnDemand interface can be accessed at localhost:8080

## VMware

The VM image defaults to use DHCP.  If DHCP is not setup for the imported VM, an IP must be set.  Below is an example.

    ip addr add <IP>/<NETMASK> dev eth0
    ip route add default via <GATEWAY>

The root password for the image is `ood`.

# Development

## Vagrant
The code to overcome vbguest install issues with this aging version of Centos 7, comes from information on this [thread](https://github.com/dotless-de/vagrant-vbguest/issues/351) and this [vagrant template](https://github.com/carlosefr/vagrant-templates/blob/master/vm-centos/Vagrantfile#L33) by Carlos Rodrigues
[carlosefr](https://github.com/carlosefr). Thanks.