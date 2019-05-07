---
layout: post
title: New Hardware, Switching to Arch Linux and PCI Passthrough for Gaming
tags: hardware sysadmin gaming
---

While playing The Witcher 3 on my old dual-boot system I realized that it was time for an upgrade. I would take this opportunity and try [Arch Linux](https://www.archlinux.org/) after all my life with Debian and also [PCI passthrough](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF) which sounded very exciting. Running Windows in a virtual machine is a viable alternative to dual-booting when you can pass through your graphics card with virtually native performance (pun intended).

This post is a summary of the process of building my new system. From choosing the hardware, to setting up the operating system and configuring my gaming virtual machine.

## Hardware

Besides being able to run current video games at max graphics settings I chose my hardware to be suitable for my PCI passthrough plans and to be relatively popular to avoid compatibility issues.

I went for an Intel processor with integrated graphics for my host system when the Windows guest is using my GPU. I settled on the Intel Core i5-9600K processor, the i5 series seems still cheaper in this performance range and it supports the required virtualization features.

My mainboard is a standard ATX board, the ASRock Z370 Pro4 has two PCIEx16 slots and four DDR4 slots, for now I got two 8 GB RAM sticks and left two slots free for future expansion. The board has HDMI and DVI-D ports for integrated graphics. I connect the HDMI to my main screen and the DVI-D to my small secondary screen which doesn't have HDMI. The board also has Bluetooth and WLAN which I might need in my new room when I move out from home. As a bonus I can overclock my processor if I ever want to do that.

Because I want to try the new hottest thing called ray tracing I got the Gigabyte GeForce RTX 2060 Windforce OC GPU. It has 6 GB of video RAM and the custom design runs at 1770 MHz in comparison to the founders edition with 1680 MHz.

For storage I went with three 2 TB hard drives two of which I cycle for backups. And a 500 GB SSD will store my Windows virtual machine image.

## Arch adventures

I had my eyes on the Arch Linux distribution for a while now. First I discovered Arch by it's [phenomenal wiki](https://wiki.archlinux.org/). Over time I became to use Debian like Arch. Making the Debian installation procedure work with my uncommon partition layout and window manager, hacking the initramfs or installing a newer version of Firefox with the Quantum engine. I decided instead of switching to Debian testing or unstable I would try Arch Linux, and here we are. I followed the [instructions on PCI passthrough](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF) from the Arch wiki without having to worry about differences on Debian. And having the newest software already paid off because they fixed some nasty audio crackling issues in the newest QEMU versions.

I was initially worried about the effort it would take to maintain a system with rolling releases i.e. how much would break and require manual fixing. A glance at the Arch news alleviated my concerns and today after nearly two months I can say that I only had one tiny issue which was dead easy to fix. My window manager wasn't starting after a shared library upgrade because it is manually compiled from the config file.

### The AUR is scary

I love Arch and I could write an entire post about what's great about it. But instead I want to talk about a few things which troubled me as someone coming from Debian.

The [Arch User Repository](https://aur.archlinux.org/) hosts unofficial Arch packages by the community and sooner or later you will need a package from the AUR. And the AUR is scary for newcomers. The wiki makes it clear that AUR packages are not supported and that it is your responsibility to update them and care for their library dependencies. The [instructions](https://wiki.archlinux.org/index.php/Arch_User_Repository#Installing_packages) on installing packages from the AUR look complex for just installing a package until you realize there are so called [AUR helpers](https://wiki.archlinux.org/index.php/AUR_helpers). But of course everyone is telling you not to blindly use AUR helpers but to understand what's going on behind the scenes.

Long story short, an additional repository which requires an additional package manager feels impractical when you're coming from Debian. I looked through the different AUR helpers with their numerous advantages and disadvantages and decided for [Yay](https://github.com/Jguer/yay). It seems like the easiest to use and it can even hide the separation between AUR and non-AUR packages which purists don't like.

Installing any package, official or not, is achieved with `yay -S PACKAGE` and upgrading works by `yay -Syu`. Just like [pacman](https://wiki.archlinux.org/index.php/Pacman).

### Fonts

I learned to appreciate how good the default Debian fonts are after switching to Arch. The Arch philosophy says to leave the choice to the user and apparently this also applies to fonts. On a fresh Arch installation you only have a hand full of absolutely necessary fonts installed. Especially the web looks bad but even GTK apps don't look right.

I went through the font packages installed on my old Debian system and tried to replicate this as good as I could on Arch. There are some fonts which are not packaged for Arch or only in the AUR and I didn't include those. This is my final list of font packages. Terminus is the exception here, it is not a Debian default font but my personal preference.

{% highlight bash %}
pacman -S cantarell-fonts ttf-caladea ttf-carlito ttf-dejavu \
          ttf-droid ttf-freefont ttf-inconsolata             \
          ttf-linux-libertine noto-fonts ttf-gentium         \
          ttf-bitstream-vera gsfonts terminus-font
{% endhighlight %}

I always like to turn on full font hinting. On Arch you change a symlink while you would use `dpkg-reconfigure fontconfig` on Debian.

{% highlight bash %}
rm /etc/fonts/conf.d/10-hinting-slight.conf
ln -s /etc/fonts/avail.d/10-hinting-full.conf /etc/fonts/conf.d/
{% endhighlight %}

It turns out getting proper subpixel hinting like I am used to from Debian also required editing `/etc/profile.d/freetype2.sh` and changing the mode to classic.

{% highlight bash %}
# Subpixel hinting mode can be chosen by setting the right TrueType
# interpreter version. The available settings are:
#
#   truetype:interpreter-version=35  # Classic mode (default in 2.6)
#   truetype:interpreter-version=38  # Infinality mode
#   truetype:interpreter-version=40  # Minimal mode (default in 2.7)
#
# There are more properties that can be set, separated by
# whitespace. Please refer to the FreeType documentation
# for details.

# Uncomment and configure below
export FREETYPE_PROPERTIES="truetype:interpreter-version=35"
{% endhighlight %}

With my collection of fonts the system was still choosing a font for GTK apps I didn't like so I told it to use the DejaVu Sans font by creating a GTK config file at `/etc/gtk-3.0/settings.ini`.

{% highlight none %}
[Settings]
gtk-font-name = DejaVu Sans
{% endhighlight %}

So unfortunately after some fiddeling around the fonts still don't look as good as on a Debian system, but I can get used to the new look. I was surprised about how small the collection of Debian default fonts is that looks good with pretty much any website. I would wish for a similar default on Arch---I believe few people like to step back and think about which fonts to pick after installing Firefox---but I know it's not happening because it conflicts with Arch philosophy.

### Version control for config files

I always like to know which configuration files I changed so I have two git repositories. One at `/` with mostly files from `/etc` and some `/usr/local` scripts. And one in my home mostly for dotfiles.

Categorizing files into multiple git repositories seems tempting especially for the home directory. But what I never liked about [vcsh](https://github.com/RichiH/vcsh) or [etckeeper](http://etckeeper.branchable.com/) is that they are specific to a particular path. The dream would be to have a set of config files to check out on remote machines but that might be just fantasy. It's a different system, the config manager won't be there and different versions might require different config files. I doubt this would ever just work.

### Disk encryption

On my old system the entire disk was encrpyted. This time I just encrypt the home partition and add a PAM configuration to hook into the login process and unlock the cryptsetup container. This means I have to type my password just once. The script at `/usr/local/lib/mount-home` will be called from the PAM config.

{% highlight bash %}
#!/bin/sh

tr -d '\0' | cryptsetup open UUID=123 home
mount /home
{% endhighlight %}

In `/etc/pam.d/system-login` after the `include system-auth` line a line using the pam_exec module that calls the script needs to be added.

{% highlight none %}
...
auth       include    system-auth
auth       optional   pam_exec.so expose_authtok quiet /usr/local/lib/mount-home
...
{% endhighlight %}

This also allows to unlock the home partition over SSH. I can start my computer from remote using Wake-on-LAN and when I connect via SSH it will ask me for a password because my home is not mounted yet and the `authorized_keys` file is not there.

### Backup

I backup my full system using rsync to two hard drives I cycle. The disks are bootable and have EFI, swap and data partitions. The data partition is encrypted using LUKS1 and also contains `/boot`. After copying the entire `/` to the backup disk I update `/etc/fstab` and set the kernel parameters for GRUB to handle an encrypted `/boot`. Then I use `arch-chroot` to update the initramfs and GRUB installation.

I can test the backup by booting it and it would be usable in case my hard drive is corrupted. Unfortunately I have to enter my password three times when booting from the backup. Once for GRUB, once for the initramfs and finally to log in.

## PCI Passthrough

I followed the excellent [guide](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF) on the Arch wiki to set up my Windows virtual machine. I want to go into a few choices I've made during the process and explain my reasoning.

### Configuration

For the disk driver I use SCSI (virtio-scsi) instead of the older VirtIO (virtio-blk) because it supports the discard feature (also known as TRIM). Adding the following line to the disk tag in the VM config makes the Qcow2 image shrink when the guest OS uses discard and thus prevents the image from slowly growing bigger.

{% highlight xml %}
<driver name='qemu' type='qcow2' discard='unmap'/>
{% endhighlight %}

This works as long as TRIM is configured on the guest but Windows 10 seems to have continous TRIM enabled by default. Removing a big file on my Windows VM makes the image file---which is a sparse file of course---shrink. Because the image is stored on my ext4 formatted SSD I also run a periodic TRIM service on the host system. The rsync command can copy sparse files with the `--sparse` flag so reducing the image size really saves backup time.

I also configured [CPU pinning](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#CPU_pinning) and simply assigned each of my 6 cores to a virtual core, my processor has no hyperthreading. For some reason I had to check *Manually set CPU topology* in the virt-manager. It kept messing up the topology telling the guest system I had 6 CPUs with 1 core each. First I thought I had a serious performance issue because Windows does only use two CPUs at maximum which meant the VM was only using two cores.

My keyboard and mouse can be switched between guest and host by pressing both CTRL keys at once as [explained on the Arch wiki](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#Passing_keyboard/mouse_via_Evdev). When I configured the [guest sound to be passed to PulseAudio](https://wiki.archlinux.org/index.php/PCI_passthrough_via_OVMF#Passing_VM_audio_to_host_via_PulseAudio) I needed to change the QEMU user and group to my own user. To keep the keyboard and mouse switching working you need to add your user to the `input` group. When passing the sound to PulseAudio the VM sound appears as a separate application in the pavucontrol mixer. Though especially in older QEMU versions there are bad audio cackling issues. In QEMU 3.0 they are not completely gone but they are extremly rare and subtle now.

For the network I use the MacVTap driver because it's supposed to have less CPU impact on the host.

I briefly explained my screen setup above. Both of my screens are connected to the integrated graphics used by the host and only my main screen is connected to the GPU used by the guest. I set up some xrandr bindings in my window manager to disable and enable my main screen for the host.

{% highlight bash %}
# turn on main screen
xrandr --output HDMI-1 --auto --right-of HDMI-2
# turn off main screen
xrandr --output HDMI-1 --off
{% endhighlight %}

My script to start the virtual machine also turns off the screen on the host so the screen automatically uses the other HDMI input.

### Conclusion

I am super pleased with my PCI passthrough setup. Apart from the initial effort it took to configure the VM no issues arised from the virtualization. I have finished The Witcher 3 on my new system in beautiful 60 frames per second and played a bunch of other games without issues. When I want to play an online game with a friend I run TeamSpeak on Linux and keep my tabs open on the host while starting the virtual machine. Goodbye dual-booting!

You can leave comments on [HackerNews](https://news.ycombinator.com/item?id=19849809). As an Arch newbie I'd be happy about your tips!
