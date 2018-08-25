---
layout: post
title:  "How to make Ubuntu 18.04 work on Dell G5"
date:   2018-08-14 15:14:33 +0200
background: ''
categories: dell g5 linux ubuntu installation nvidia graphics
comments: true
---

Linux is a great! It's configurable, it's perfect for developers and it gives you an overlall nice experience. Unless... you are installing it
on a new machine and something doesn't go right.

You have to give it to Mac and Windows - they don't offer the same flexibility as Linux systems, but at least they
WORK OUT OF THE BOX.

This is a the story about how I made Ubuntu 18.04 working on my machine, which is a Dell G5 5587. This is by no means a general guide of how to
approach the problem I encountered, but a list of step-by-step instructions which worked for me.

## In the beginning there was the USB stick...

So recently I bought a new laptop - a Dell G5 5587 with NVidia GTX 1060 <nobr>Max-Q.</nobr> It had a Windows preinstalled and two drives:
SSD and HDD. SSD is 256GB and has Windows on it. I decided to divide this partition into 2 parts as I want both Windows and Linux
to perform as best as possible. I shrinked the Windows partition using Windows disk management tool and left free space for Linux
partition. After that I downloaded Ubuntu ISO from their official website.

Believe it or not, problems started at this stage already. After you download the Ubuntu ISO you have to install it on a flash drive.
To do that I used Ubuntu's built-in 'Startup disk creator' (I used another machine here). After installation finished and I excitingly
booted from the USB and it froze! I googled this issue I found that it's best to run Rufus disk creator from Windows and look for bad sectors on
the flash drive of the USB stick. Indeed, turned out my pendrive was at the end of it's life due to natural causes (it's quite old) as Rufus detected
some bad sectors. Fortunately I had another, newer flash drive that I could use.

Before you run the Ubuntu installation itself make sure to disable secure boot in BIOS settings. This will allow you to install third-party software during
Ubuntu install and also it's a recommended step to do, because secure boot may (or may not) cause some additional issues. Just remember turn it on
later.

Now boot from your USB and go through the installation process and make sure you have internet connection at all times.

It may happen that your installer crashed at some point during install - like mine did. You can try putting `nomodeset` in boot options when booting
from USB. I'm not sure if this has any effect, because in my case installer crashed at random times so it could be that I was just lucky.

After it's done click "Restart now". This can crash, but no worries.

## Graphics issues - nomodeset to the rescue

When booting ubuntu, insert `nomodeset` before `splash` in boot options. If you don't do this your laptop will crash somewhere around splash screen
and you'll have to to force shutdown. When in grub menu go over `Ubuntu` and press `e` key. Find line with `splash` at the end and put `nomodeset`
before `splash`. Continue with the boot process. If you don't want to put `nomodeset` every time before booting, update your /etc/default/grub
file and execute `update-grub` as root. Take note though that putting `nomodeset` in boot options is temporary.

Now you should be able to boot Ubuntu and login using Xorg. Because we disabled kernel graphics configuration (`nomodeset`), some
features may not work. In my case, manipulation of the display was limited. I couldn't change brightness using buttons on
my keyboard and Night Light wasn't working, but this is fine for now.

Here is a good time to make some research about the state of your system. Install `lshw`: `sudo apt install lshw` (remember about
running `sudo apt update` before). This utility will provide you with details about your hardware. Execute `lshw -C video`.
In my case, I got two entries, one from NVidia, other from Intel (Dell G5 has two graphic cards). Both of them had `UNCLAIMED`
word next to them. This is due to the fact that we set `nomodeset` in the kernel. `UNCLAIMED` means that there is no driver attached to
the hardware. Now the plan is to prevent the system from crashing and get rid of `nomodeset`. In my case what worked was the
installation of proper NVidia drivers, but I'll come back to it in a bit.

## Further examination

It's best to find out as much as you can about the state of the system before proceeding to be sure about what you're doing next.
You can try `lspci | grep -i vga` to get a list of PCI video devices. Here I got 2 entries for both of my graphic cards
(NVidia and Intel). Also run `lsmod | grep i915`. i915 is the name of the Intel graphics card driver. If it shows in the result of the command then Intel
drivers should work properly without `nomodeset`. If it's not there then maybe the name of the driver is different in your case, or maybe
you need to install it first before proceeding further.

Now, I want to let you know that I spent figuring things out here for a looong time. I read a blog posts and forum discussions,
but either they didn't exactly fit my case or people who had same issues as me stopped posting on their threads. The reason it was so difficult
was that I tried to make Ubuntu work without removing `nomodeset` from boot options. Here are some of the things which **didn't** work:

* purging nouveau - there is no need as nvidia drivers blacklist nouveau automatically
* installing various NVidia drivers from version `nvidia-390` to `nvidia-375` and reinstalling / dpkg-reconfigured xserver-xorg related packages
  each time and rebooting. Here when reloading gdm `sudo systemctl restart gdm`, screen flickered a couple of times and didn't start X server. When checked
  X logs for errors `grep EE /var/log/Xorg.0.log` I could see a message `failed to initialize glx extension (compatible nvidia x driver not found)`.
  So the next step I took is I created the Xorg configuration file manually `X -configure` (when root) and added a couple of things [0] to that configuration:

  ```
      Section "Device"
          Identifier     "Device0"
          Driver         "nvidia"
          VendorName     "NVIDIA Corporation"
          BoardName      "GeForce GTX 1060 Mobile"
          BusID          "PCI:1:0:0"
      EndSection

      Section "Screen"
          Identifier     "Screen0"
          Device         "nvidia0"
          Monitor        "Monitor0"
          DefaultDepth    24
          SubSection     "Display"
              Depth       24
              Modes       "1920x1080" "1600x1200" "1024x768"
          EndSubSection
      EndSection
  ```
  After setting this configuration and restarting gdm error message changed as in to "no enabled display devices found". NVidia driver seemed
  to be loading though. Unfortunately I couldn't make it work after this, so it looked like a dead end.

## Final solution
The solution turned out to be much simpler. All you need to do is install the latest nvidia driver available (I installed the not-stable nvidia-driver-396)
and then reboot Ubuntu **without** `nomodeset` option. Your Ubuntu should work fine after that.

I hope that helped! After spending so much time on the installation I had to put this in a blog post in case you also experience this problem.
Ubuntu can be quite painful to setup sometimes, but once you overcome the initial issues you quickly forgive Ubuntu all the problems it gave you :)

Thanks for reading!

Przemek
