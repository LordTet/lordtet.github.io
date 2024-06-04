---
layout: post
title:  "Experiences with Virtualization Solutions"
date:   2024-06-04
categories: [technical, IT, software]
---

![Me with my degree](/assets/Virtualization-Understandings/diploma.jpg){:height="50%" width="50%"}
# Introduction
So, I graduated college recently (🥳🎉) and I came into possession of two valuable items: New hardware for my server, and plenty of time to kill. It was time for a refactor of my homelab. Historically, I've had one rack server virtualize the rest of the lab (except for the networking equipment), which gives me a lot of options for what software stack I want to use to run all of my services. Before I go into the solution I did end up choosing (spoiler: proxmox virtual environment), I think it would be a nice use of my time to document what I _have_ used. Because this server has received many, many, software reinstalls. 

### Preliminary Paradigm: Valkyrie Host
I figured this was worth mentioning before I actually start talking about virtualization or containerization. The very first thing I did when I started my Homelab in 2018 was load Ubuntu Sever onto the thing and throw anything I wanted to host onto it. Game servers, SSH, SMB, you name a non-networking critical protocol, I was probably running it on that device. I think this is great for a beginner if you don't expose the thing to the internet too much, just in case. But we can do better.

#### Pros
- Dead simple
- Great way to start out learning what you might want a server to do for you

#### Cons
- No separation layer, all resources on one machine, one point of failure
    - Said point of failure has to expose something to the internet if you plan on accessing it, such as SSH
- No organization, rapidly becomes a cluttered mess
- A total mess of ports if you network a lot of the same application (I once had to host a few Minecraft servers - awful idea with one box honestly)

## Virtualization Attempts
For the rest of the post here I'll run through the ways I've organized my homelab via virtualization technologies, and what I think of each of them as of now.

### VMWare ESXi
At the time, this was the first option to come to mind. Prior to setting this up, the only hypervisors I had used were VMware and Virtualbox. Which makes sense, as they are easily installed on a personal machine and operated at a desk. Initially, I had no problems with this solution. VMWare's interface is incredibly intuitive, and configuring the network bridging is extremely straightforward. No complaints about performance either. However, after VMWare was bought out by Broadcom, and they announced their intent to axe the free versions of ESXi, it was definitely time to jump ship.

#### Pros
- Still very simple to use
- Easy learning curve for virtualization
- Fairly performant

#### Cons
- Broadcom gave us the axe
- Free version has a few limitations imposed upon it (maximum vcpus, vcenter support, etc)

### FreeBSD w/ Bhyve
Heads up, between when I tried this and now, Bhyve has had a lot of work put into it ([check out their website](https://bhyve.org/)). At some point I might revisit it, but that's another story.

The BSD Hypervisor (Bhyve) is a hypervisor designed for BSD environments that is meant to be written from the ground up to be simple and performant. I was quite excited to try this when I decided to sit down and install it to my hardware, because I had previously never used a BSD system regularly. During that time, I learned a lot about what Unix operating systems can be outside of the Linux ecosystem (frankly worthy of another post), and had a few interesting things running!

For example, for all of my \*nix boxes run on top of bhyve, I was able to create a device file in /dev/ that functioned as a serial port for each of the VMs. In which case, I was able to jump from each host to the vm using the `cu` utility to dial up a terminal. At the time, management of Bhyve resources was up to me, so taking advantage of many interesting features was on my shoulders to implement for my setup.

Unfortunately, I ran into a few performance issues when it came to CPU bottlenecking. Some of the programs on the guest OS's were bottlenecking hard in my case, though I don't think my use case is what bhyve is built for.

#### Pros
- Learning opportunity for unique platforms (BSD)
- Simple (note: not necessarily easy) and clean paradigm
- Extremely active development, only growing

#### Cons
- Struggled to run demanding single-core applications within the guests
- Time invested setup (I was in school at the time!)

### Linux KVM
It's a bit comical how Linux's KVM solution came after bhyve in my iterations, I suppose I just enjoy new and novel solutions. Either way, for this solution I intended to mean business. Nothing crazy. Load up something rock solid (Debian) and use whatever is provided to make provisioning virtual machines easier. Thankfully, there are plenty of solutions out there that make this fairly straightforward - such as cockpit! Cockpit ended up being the icing on the cake of this solution, as it makes provisioning VMs extremely easy. [You can check out RedHat's documentation for the process, but it's incredibly easy once it's all set up](https://www.redhat.com/sysadmin/manage-virtual-machines-cockpit).

Unfortunately not every feature I could want is implemented by cockpit yet - things such as multiple monitors for spice/VNC connections required me to dig into the VM configuration and edit some XML data on the host. Ultimately though, this was a favorite pick of mine, I have not much bad to say about the KVM tech itself.

#### Pros
- Can be built on top of something exceptionally stable
- Cockpit makes management easy and intuitive
    - RedHat support on cockpit
- KVM is now a somewhat mature technology, lots of docs
- Up until this point, my most performant choice. VMs ran beautifully.

#### Cons
- Any configuration outside of Cockpit can be difficult and hard to find
- Whenever Cockpit would run into a hitch, it was exceptionally bad at error reporting

### Proxmox VE
And now we're at the present. Proxmox is essentially a debian derivative that presents itself as a management engine for VMs and Containers. It does this by managing LXC containers and KVM VMs behind the scenes, and presenting them with an easy to use frontend. Really, this just ended up being Cockpit with less problems to me. Additionally, using Containers instead of VMs for many of my applications provided an immediate performance boost (duh.), and being able to manage the containers alongside and virtualization-required applications is convenient. Even if something goes wrong on the backend, it's just a Linux box, which lends itself to all the tools needed to fix the issue.

#### Pros
- Good support, both community and enterprise
- Rock solid UI, haven't been able to break it yet
- Container management, worth using more often than not
- Working management for software RAID, LVM, ZFS

#### Cons
- Some of the defaults don't make sense for the personal user
- Paradigm is somewhat geared towards datacenter use


## Conclusion
Clearly, I have not many bad things to say about Proxmox. Thus far, it's my favorite solution for isolation with my one server box. Containers are performant, clean, and small. For something that cannot be containerized, the VMs are easy to set up.

Though after writing this, I am considering revisiting Bhyve sometime. Definitely not on my main machine in my lab, but just for fun on a piece of hardware I don't care for.

Here's hoping this configuration is to last! And use containers, ya dingus, they're good.