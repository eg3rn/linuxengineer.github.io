---
layout: post
title:  "Multipass With Libvirt and Cloud-Init"
---
Multipass is a lightweight Virtual Machine Manager created by Canonical that allows you to set up an Ubuntu mini-cloud on your machine.

<h1>Getting started</h1>

Get the newest packages available
{% highlight shell %}
sudo apt-get update
sudo apt-get upgrade
{% endhighlight %}

Install Virt-Manager 
{% highlight shell %}
sudo apt-get install virt-manager
{% endhighlight %}

Install Multipass (The *\--classic* switch is required)
{% highlight shell %}
sudo snap install multipass --classic
{% endhighlight %}

Set Multipass to use the Libvirt driver
{% highlight shell %}
sudo multipass set local.driver=libvirt
{% endhighlight %}

Launch our first VM
{% highlight shell %}
multipass launch 20.04 --name LE01 --cpus 1 --mem 2048M --disk 10G
{% endhighlight %}

After launch we can connect to the VM
{% highlight shell %}
multipass shell LE01
{% endhighlight %}

&nbsp;
<h1>Cloud-Init</h1>
All Multipass VMs are launched with Cloud-Init, which allows us to customize our VMs on boot.

This is an example of a basic Cloud-Init user-data YAML-file. 
{% highlight yaml %}
#cloud-config
package_upgrade: true

users:
  - name: le
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa {id_rsa.pub}
{% endhighlight %}

```package_upgrade: true``` implies that the VM will run an ```apt-get update``` and ```apt-get upgrade``` on startup.

```users:``` specifies the user that is to be created on the VM. Note that this user will be added in addition to the default Ubuntu user. In this example ```sudo:```  gives the created user rights to run any command without specifying a password. This is very useful for testing.

By specifying a ```ssh-authorized key```, we can login to the VM over SSH.

Cloud-Init is loaded by multipass by specifying the *\--cloud-init* switch on launch
{% highlight shell %}
multipass launch 20.04 --cloud-init user-data --name LE02 --mem 2048M --disk 10G
{% endhighlight %}

&nbsp;
<h1>Other useful things</h1>
Since we use DHCP per default we can see the IP-address assigned to the VM by libvirt (virbr0)
```
virsh net-dhcp-leases default 
```
