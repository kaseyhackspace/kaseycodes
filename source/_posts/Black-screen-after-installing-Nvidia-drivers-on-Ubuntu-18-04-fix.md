---
title: Black screen after installing Nvidia drivers on Ubuntu 18.04 fix
date: 2018-10-13 22:26:34
tags: 
    - nvidia
    - drivers
    - ubuntu
    - '18.04'
    - grub
---

I recently needed to reinstall Nvidia drivers as I performed a clean format to Ubuntu 18.04 after horribly crippling my system due to a failed `apt-get dist-upgrade`. Since I didn't have the time to be checking what the best driver version was for my graphics card, I went ahead and installed the drivers using the `sudo ubuntu-drivers autoinstall` command. Everything seemed ok, until I rebooted the system and was greeted with a black screen, and an unresponsive system.

I did get to login to my system whenever I manually edited the grub configuration before booting up to change `quiet splash` to `nomodeset`. To make things more permanent, I did the following steps:

Edit the `/etc/default/grub` file as root using `nano`:

``` bash
$ sudo nano /etc/default/grub
```

Change the line that has the `GRUB_CMDLINE_LINUX_DEFAULT` string from 

``` bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

to 
``` bash
GRUB_CMDLINE_LINUX_DEFAULT="nomodeset"
```

My default grub configuration now looks like this:

``` bash
# If you change this file, run 'update-grub' afterwards to update
# /boot/grub/grub.cfg.
# For full documentation of the options in this file, see:
#   info -f grub -n 'Simple configuration'

GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="nomodeset"
GRUB_CMDLINE_LINUX=""

# Uncomment to enable BadRAM filtering, modify to suit your needs
# This works with Linux (no patch required) and with any kernel that obtains
# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)
#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"

# Uncomment to disable graphical terminal (grub-pc only)
#GRUB_TERMINAL=console

# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'
#GRUB_GFXMODE=640x480

# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux
#GRUB_DISABLE_LINUX_UUID=true

# Uncomment to disable generation of recovery mode menu entries
#GRUB_DISABLE_RECOVERY="true"

# Uncomment to get a beep at grub start
#GRUB_INIT_TUNE="480 440 1"
```

After saving that file (using `nano`, do a `Ctl+X` then press `Y`), go ahead and regenerate your grub file using the command

``` bash
sudo update grub
```

Once you reboot, you should have the Ubuntu recommended stable Nvidia drivers working! I know it works because my Google Earth Pro now renders the globe perfectly and the system is now smooth with its transitions :)

I hope this post helps a lot of people out, and see you guys in my next post!