Multipass
=========

.. _Multipass: https://multipass.run/
.. _Canonical: https://canonical.com/
.. _Vagrant: https://www.vagrantup.com/

`Multipass`_ is `Canonical`_'s solution for easy Virtual Machine
(VM) management. Think of it like `Vagrant`_, but from the Ubuntu makers.

It is super easy to setup and use it.

Installing Multipass
--------------------

Multipass can easily be installed and kept up-to-date using
Canonical's Snap packages. To install Multipass, use
the command ``snap install multipass``.

.. sourcecode:: console

    $ sudo snap install multipass
    [sudo] password for username:
    multipass 1.12.2 from Canonical✓ installed

Now we can see the available images to instantiate a VM.

Looking for images to instantiate a VM
--------------------------------------

To list the available images that can be use to create a new VM,
use the command ``multipass find``.

.. sourcecode:: console

    $ multipass find
    Image                       Aliases           Version          Description
    core                        core16            20200818         Ubuntu Core 16
    core18                                        20211124         Ubuntu Core 18
    core20                                        20230119         Ubuntu Core 20
    core22                                        20230717         Ubuntu Core 22
    20.04                       focal             20231129         Ubuntu 20.04 LTS
    22.04                       jammy,lts         20231130         Ubuntu 22.04 LTS
    23.04                       lunar             20231205         Ubuntu 23.04
    23.10                       mantic            20231011         Ubuntu 23.10
    appliance:adguard-home                        20200812         Ubuntu AdGuard Home Appliance
    appliance:mosquitto                           20200812         Ubuntu Mosquitto Appliance
    appliance:nextcloud                           20200812         Ubuntu Nextcloud Appliance
    appliance:openhab                             20200812         Ubuntu openHAB Home Appliance
    appliance:plexmediaserver                     20200812         Ubuntu Plex Media Server Appliance

    Blueprint                   Aliases           Version          Description
    anbox-cloud-appliance                         latest           Anbox Cloud Appliance
    charm-dev                                     latest           A development and testing environment for charmers
    docker                                        0.4              A Docker environment with Portainer and related tools
    jellyfin                                      latest           Jellyfin is a Free Software Media System that puts you in control of managing and streaming your media.
    minikube                                      latest           minikube is local Kubernetes
    ros-noetic                                    0.1              A development and testing environment for ROS Noetic.
    ros2-humble                                   0.1              A development and testing environment for ROS 2 Humble.

Let's assume we want a Ubuntu 20.04 VM and we now want to instantiate it.

Instantiating a VM
------------------

To instantiate a new VM, use ``multipass launch --name vm_instance_name image_name``.

.. sourcecode:: console

    $ multipass launch --name testvm 20.04
    Launched: testvm

Inspecting a VM
---------------

To inspect the VM configuration, use ``multipass info vm_instance_name``.

.. sourcecode:: console

    $ multipass info testvm 20.04
    Name:           testvm
    State:          Running
    IPv4:           10.142.101.175
    Release:        Ubuntu 20.04.6 LTS
    Image hash:     f5cdf6bf458b (Ubuntu 20.04 LTS)
    CPU(s):         1
    Load:           0.67 0.41 0.16
    Disk usage:     1.5GiB out of 4.8GiB
    Memory usage:   146.1MiB out of 959.4MiB
    Mounts:         --

Very restrictive for our purposes, so let's shut it down and reconfigure
the configuration of the VM.

Stopping a VM
-------------

To stop a VM, use ``multipass stop vm_instance_name``.

.. sourcecode:: console

    $ multipass stop testvm
    $ multipass list
    Name                    State             IPv4             Image
    testvm                  Stopped           --               Ubuntu 20.04 LTS

Now that the VM is stopped, we can proceed to change its configuration.

Reconfigure a VM configuration
------------------------------

So let's add to our test VM some additional cores, memory and disk space.

.. sourcecode:: console

    $ multipass set local.testvm.cpus=8
    $ multipass set local.testvm.memory=10GB
    $ multipass set local.testvm.disk=10GB
    $ multipass start testvm
    $ multipass info testvm
    Name:           testvm
    State:          Running
    IPv4:           10.142.101.175
    Release:        Ubuntu 20.04.6 LTS
    Image hash:     f5cdf6bf458b (Ubuntu 20.04 LTS)
    CPU(s):         8
    Load:           0.21 0.05 0.02
    Disk usage:     1.5GiB out of 9.6GiB
    Memory usage:   218.2MiB out of 9.7GiB
    Mounts:         --

Much better! But how do we access the VM shell?

Getting into a VM
-----------------

For end-users, getting into the VM just requires SSHing into the server
with their credentials. We are going to dump you straight into your
respective VM shell. And if you try escaping from it, we kill your VM.

For administrators, we get there using:

.. sourcecode:: console

    $ multipass exec testvm -- bash
    ubuntu@testvm:~$ exit
    $

Sharing files to/from a VM
--------------------------

Like Docker, Multipass allows us to mount shared volumes.
Let's see it in action.

.. sourcecode:: console

    $ multipass mount ./ testvm:/home/ubuntu/multipass
    $ ls
    $ multipass exec testvm -- bash
    ubuntu@testvm:~/multipass/$ ls
    ubuntu@testvm:~/multipass/$ echo "Hello world!" > test.txt
    ubuntu@testvm:~/multipass/$ ls
    test.txt
    ubuntu@testvm:~/multipass/$ exit
    $ ls
    test.txt
    $ cat test.txt
    Hello world!

As we can see, we have access to the shared directory via both sides.

Unsharing files to/from a VM
----------------------------

Shared directories will persist and need to be manually removed
if sharing is only meant for copying configuration files and such.

To unshare a directory, use the following.

.. sourcecode:: console

    $ multipass info testvm
    Name:           testvm
    State:          Running
    IPv4:           10.142.101.175
    Release:        Ubuntu 20.04.6 LTS
    Image hash:     f5cdf6bf458b (Ubuntu 20.04 LTS)
    CPU(s):         8
    Load:           0.21 0.05 0.02
    Disk usage:     1.5GiB out of 9.6GiB
    Memory usage:   218.2MiB out of 9.7GiB
    Mounts:         /host/directory => /home/ubuntu/multipass
                    UID map: 1000:default
                    GID map: 1000:default
    $ multipass unmount testvm:/home/ubuntu/multipass
    $ multipass info testvm
    Name:           testvm
        State:          Running
        IPv4:           10.142.101.175
        Release:        Ubuntu 20.04.6 LTS
        Image hash:     f5cdf6bf458b (Ubuntu 20.04 LTS)
        CPU(s):         8
        Load:           0.21 0.05 0.02
        Disk usage:     1.5GiB out of 9.6GiB
        Memory usage:   218.2MiB out of 9.7GiB
        Mounts:         --

Pretty neat, ain't it?

Removing a VM
-------------

Removing VMs involve stopping them, then deleting their entry,
then purging its data.

.. sourcecode:: console

    $ multipass stop testvm
    $ multipass list
    Name                    State             IPv4             Image
    testvm                  Stopped           --               Ubuntu 20.04 LTS
    $ multipass delete testvm
    $ multipass list
    Name                    State             IPv4             Image
    testvm                  Deleted           --               Not Available
    $ multipass purge
    $ multipass list
    No instances found.

