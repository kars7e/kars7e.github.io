---
title: "Kubernetes cluster on ARM using Asus Tinkerboard"
description: How to build Kubernetes cluster on ARM, using Asus Tinkerboard as a base and OpenFaaS as the serverless platform
date: 2018-01-14T16:13:20-08:00
author: karol_stepniewski
comments: true
tags:
- kubernetes
- arm
- tinkerboard
- armbian
- k8s-1.9
- openfaas
- raspberrypi
---

Over the winter holiday break of 2017 I've spent some time playing with Kubernetes on ARM. I had a lot of fun building this beast:

{{< tweet 948122096969818113 >}}

I finally found some time to write about it in details. Let's go! 
<!--more-->

## Shopping list ##
The most popular choice for ARM cluster is of course [Raspberry Pi](http://amzn.to/2Dfa9V7), well-tested board with a huge community. I've decided to use something else for my cluster, and I based it on [Asus Tinker board](http://amzn.to/2DdTwJH). The most important differences:

* **2GB of RAM** (compared to 1GB in RPi)
* **Gigabit Ethernet** (compared to 100Mbit)

Twice the amount of memory will come very useful when running a bigger number of containers (for example, [Prometheus uses](https://prometheus.io/docs/prometheus/1.8/storage/#memory-usage) a significant amount of memory, subject to tunning).
1 Gbit/s Ethernet link will come useful as I want to expose persistent volumes to [Kubernetes pods via NFS](https://kubernetes.io/docs/concepts/storage/volumes/#nfs) from my [Synology DS 218+](http://amzn.to/2FFpCw7) NAS. 

Here is the complete list of required components:
 
| Part                                                                                                      | Price      | Quantity     | Sum         |
|-------------------------------------------------------------------------------------------------------    |--------    |----------    |---------    |
| [Asus Tinker board](http://amzn.to/2DdTwJH)                                                               | $59.45     | 4            | $237.80     |
| [16GB SD Card](http://amzn.to/2D4kG29)                                                                    | $9.97      | 4            | $39.88      |
| [Anker 3A 63W USB Charger](http://amzn.to/2mu1yDI)                                                        | $35.99     | 1            | $35.99      |
| [1ft microUSB cable, set of 4](http://amzn.to/2D2mpVI)                                                    | $9.99      | 1            | $9.99       |
| [8 Port Gigabit Switch](http://amzn.to/2DvDSa2)                                                           | $25.32     | 1            | $25.32      |
| [Cat 6 Ethernet cable 1ft, set of 4](http://amzn.to/2Dx9wEo)                                              | $8.90      | 1            | $8.90       |
| [RPi stackable Dog Bone Case](http://amzn.to/2rbgZqb)                                                     | $22.93     | 2            | $45.86      |
| [Arctic USB Fan](http://amzn.to/2Dw7mon)                                                                  | $4.12      | 1            | $4.12       |
| [WD PiDrive 375GB](https://www.wdc.com/products/wdlabs/wd-pidrive-foundation-edition.html#WD3750LMCW)     | $29.99     | 4            | $119.96     |
|                                                                                                           |            |              | **$527.82** |

**Woah**, a whopping *$527.82*! Well, no one said it's a cheap hobby... but you can make it less expensive. For example, you could reduce the number of nodes to 3; this will cut the cost by $100 (and a 3-node cluster is still kick-ass!). You could also remove the PiDrives. I'm using them as a mount point for `/var` directory, which will hold logs, and most importantly, docker container file system. These tend to be write-intensive, and storing them on  SD card may result in a much shorter card lifetime. Finally, you could change Tinker board to some cheaper board, even the previously mentioned Raspberry Pi... but this post is about deploying Kubernetes on Tinker board and switching to a different board will make the rest of this post useless for you :-).

There are two differences between the list above and the actual parts I've used:

* I swapped 32GB SD card I've used in my build with 16GB - you don't need that much space if you are using external HDD.
* I've used 5-port gigabit switch instead of 8. However, now that I've assembled everything, I saw I might have made a mistake - I've used all the ports, with no room for expansion! The 5-port switch has one nice advantage; it fits nicely on top of the case. It's up to you!

**NOTE: Make sure the charger you use can deliver at least 2.5 Amp per USB port**. 3 Amps are preferred. Tinker board needs more current than Raspberry Pi, and also that single port will be used to power both disk and the board. The Anker Charger I've included in the list can squeeze out 3 Amps per port, so you're covered. 


## Assembling ##
We've got all parts. **Let's assemble them!**
{{< imgproc all_parts Fit "700x1024" "All parts ready for assembly!">}}

I've started with Tinker board and [RPi stackable Dog Bone Case](http://amzn.to/2rbgZqb). Tinker board is fully compatible (size-wise) with Raspberry Pi, so I thought it could not have been easier. The first time I used the washers, however, I used them in the wrong way... (hint: they are supposed to go *between* the board and the case, as spacers, to ensure proper space left for the bottom side of the board). **Make sure you attached the radiator to the CPU before you mount another layer** - they tend to heat much more than RPi. In fact, I've added even more cooling; I will come back to that soon.

{{< imgproc single_board Fit "700x1024" "Single board attached to the first case layer">}} 

I knew that [RPi stackable Dog Bone Case](http://amzn.to/2rbgZqb) would fit Tinker board as well, but I had no idea about the drives. In fact, they are much bigger, and screw holes in drives do not match much those in the case (not to mention they need completely different screws). So... double-sided tape to the rescue! I've taped the disks to the case layers, and the result was very satisfying. 

Now it's time to connect disks and Tinker boards. The idea is simple - the micro USB cable goes from the charger to disk via another cable included with the disk. This cable is then connected to the board twice - once via micro USB (to supply power), and once via USB-A (for data). The USB cable is quite short, so make sure it fits well before securing the two cases together!

{{< imgproc charged_not_mounted_powered_on_3 Fit "700x1024" "Two cases combined together">}}

Once both cases (the case with boards and case with disks) were fully stacked, I've "secured" them together using velcro ties that came with micro USB cables. 

I mentioned that these boards tend to heat quite a lot. In fact, after I deployed Kubernetes and kept them running for some time (without any additional workload), the temperature went up to 60-65C.  I figured that's too much (especially that I wanted to store the cluster in a closet without much air ventilation. I ordered a [$4 Arctic USB Fan](http://amzn.to/2Dw7mon) and used the last remaining USB port in the charger for it. The result? **the average temperature dropped to 35-40 degrees!**.

{{< imgproc fan_mounted_1 Fit "700x1024" "Final result, placed in the closet, with USB fan mounted">}}


**Woop, woop, our Cluster is ready for containers!**

## Imaging & First boot ##
... But first, we need to prepare our SD cards with an OS image. I've used [Armbian](https://www.armbian.com/) with great success. Head over to [https://dl.armbian.com/tinkerboard/](https://dl.armbian.com/tinkerboard/) to download the image. I've used nightly Ubuntu Xenial mainline build. The thing is... this image is no longer available. It looks like there is [Ubuntu Xenial Next Desktop](https://dl.armbian.com/tinkerboard/Ubuntu_xenial_next_desktop.7z) available. However, this image is much bigger as it includes the desktop bits. If that's ok for you, feel free to use it. The currently available [server image](https://dl.armbian.com/tinkerboard/Ubuntu_xenial_next.7z) is from 2017-10-18, which is too old and does not include [this fix](https://github.com/armbian/build/commit/72b42b5b10495d16562cd87bfbb889a054ced2a1). If you don't mind downloading images from random internet locations, [here is the image](http://kars7e-public.e24files.com/Armbian_5.35.171201_Tinkerboard_Ubuntu_xenial_next_4.14.2.img) I've used, uploaded by me. I see updated images based on Debian - my guessing is that Armbian maintainers expect users to use Debian for server purposes. I haven't tried it yet, but I'll update the post if I try it and it works. 

Now it's time to "burn" image to SD card. Armbian recommends [Etcher](https://etcher.io/), I can wholeheartedly recommend it as well. It's straight-forward, multi-platform, and it simplifies the multi-card burning process.

Once all the cards are ready, it's time to **boot your cluster for the first time!**.
Note that, however, the first boot will require monitor and keyboard...  you will need to log in with default credentials, change the root password, and create a new non-root user. I haven't figured out how to avoid that step, but to be honest, I haven't even tried - I'm guessing it might be well-documented on Armbian website. I will update the article if I find a way.

While you are logged in to the board via monitor & keyboard, there are two steps that you can do here (everything else we will automate, I promise!) on each of the boards:

### Make sure networking will not get borked after you reboot
It seems to me that there is an issue between `/etc/network/interfaces` and `NetworkManager`, both trying to claim ownership of networking card. I've resolved that issue by
ensuring NetworkManager is the winner - just run `cp /etc/network/interfaces.network-manager /etc/network/interfaces`, which will basically remove all interfaces from this file.

### Format and mount the PiDrive 
 Run `cfdisk /dev/sda` to create new Linux partition, run `mkfs.ext4 /dev/sda1` to format it as `ext4` file system, and finally add `UUID=e4ea132a-2f0f-418c-928a-c3d0503c6b44 /var ext4 defaults,noatime,nodiratime,errors=remount-ro 0 2` to `/etc/fstab` (You can find the uuid of the disk by running `ls -l /dev/disk/by-uuid/ | grep sda1` or `blkid /dev/sda1 | awk '{print $2;}'`). Finally, run the following sequence of commands:
```
cp -a /var /var.bak
mkdir /var
mount /var
cp -a /var.bak/* /var/
reboot
rm -rf /var.bak
``` 
The above will ensure that your `/var` directory is using PiDrive, and that its current content is copied over.

That was a lot of manual work, yuck! But from now on, we will **automate all the things!**

## Preparing OS & installing Kubernetes##
There are still few things we want to configure on our boards before we can install Kubernetes. Updating packages, installing dependencies, and so on... but we will do that all using [Ansible](https://www.ansible.com/). Make sure you have ansible installed before proceeding (you can find install docs on Ansible website). One more thing to do before running ansible: **configure your SSH**. First, copy your public key to all nodes: `ssh-copy-id -i ~/.ssh/id_rsa YOUR_USERNAME@tinker-0` for each of the nodes. Ensure your `~/.ssh/config` has following lines:
```
Host tinker-0 tinker-1 tinker-2 tinker-3
    User YOUR_USERNAME
```

and finally, log in to each of the nodes, run `visudo`, and make sure the following line is there:
```
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```
The above will make sudo passwordless, and your ansible will run smoothly. 

[Here](https://github.com/kars7e/tinker-cluster/blob/master/ansible/configure.yml) is the playbook that will get you the fully working Kubernetes cluster. Just make sure you have an inventory file, like this:
```
tinker-0
tinker-1
tinker-2
tinker-3

[k8s-master]
tinker-0

[k8s-node]
tinker-1
tinker-2
tinker-3
``` 
(Note: the above file assumes that your cluster nodes are accessible via their hostnames. if your DNS does not provide that, add `ansible_host=192.0.2.50` to each entry in first four lines of the inventory file, replacing the IP with the correct one).

Then run it like this:
```
ansible-playbook -i inventory.ini configure.yml
```
**That's it; now you have fully functional Kubernetes 1.9 cluster!**

## Installing OpenFaaS ##
Before installing OpenFaaS, make sure you have `kubectl` installed (`brew install kubernetes-cli` on macos), and you have it configured (Copy `/etc/kubernetes/admin.conf` from your master node to `~/.kube/config`). Once you have the above, just run:
```
git clone https://github.com/openfaas/faas-netes
cd faas-netes/yaml_armhf
kubectl apply -f nats.armhf.yml,faas.async.armhf.yml,rbac.yml,monitoring.armhf.yml
```

***And you should be good to go!***

