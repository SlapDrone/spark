
# Arch Setup Guide

A simple guide for getting Arch running on ASUS laptops

## Installing

To install Arch on all ASUS laptops just follow the regular [installation guide](https://wiki.archlinux.org/title/installation_guide). Only issue which might appear is nouveau crashing the installation, this can be solved by adding a boot parameter `modprobe.blacklist=nouveau` to the kernel cmdline before booting the installation media.

To edit the installation media boot entry just press `e` on it and then put the blacklist parameter at the end of all parameters. Example:

![Blacklist nouveau](/images/guides/arch-guide/blacklist-nouveau.png)

The same parameter can be used to boot the system after installing as long you don't install nvidia drivers.

## Repo

g14 repo contains all the tools you need on a ROG laptop precompiled for you. g14 is only a name and all tools from it apply to most ROG laptops. Before adding the repo you need to add the repo sign key to your pacman-key. Run the following commands to add it:

```bash
pacman-key --recv-keys 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
pacman-key --finger 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
pacman-key --lsign-key 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
pacman-key --finger 8F654886F17D497FEFE3DB448B15A6B0E9A3FA35
```

This should show output similar to this:
![Pacman key](/images/guides/arch-guide/g14-sign-key.png)

If you have any problems check if `/etc/pacman.d/gnupg/gpg.conf` doesn't have specified the keyserver or make sure it is `hkp://keyserver.ubuntu.com`.

If you still have problems check if you are not running some active VPN connection, this does sometimes cause problems when fetching the server.

If you still have problems you can do it the less proper way by running those commands:

```bash
wget "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x8b15a6b0e9a3fa35" -O g14.sec
sudo pacman-key -a g14.sec
```

After that to get the repo add to your `/etc/pacman.conf` at the end:

```bash
[g14]
Server = https://arch.asus-linux.org
```

You could use a mirror instead if the origin has slow connection. Following is a list of available servers:

```bash
https://arch.asus-linux.org # Germany, origin
https://naru.jhyub.dev/$repo # Republic of Korea
```

After adding the repo run a full system update before you go to install tools from the repo:

```bash
pacman -Suy
```

## Asusctl - custom fan profiles, anime, led control etc.

The recommended way to install asusctl is using g14 pacman repo. Packages like asusctl-git from AUR aren't supported. Also installing manually from cloned git isn't supported. For installing it run:

```bash
pacman -S asusctl power-profiles-daemon
```

asusd service is triggered by a udev rule after the keyboard driver is ready, the service doesn't need to be enabled and is not supposed to be.

power-profiles-daemon is optional but highly recommend, other power management tools can create conflicts with asusctl or supergfxctl.

```bash
systemctl enable --now power-profiles-daemon.service
```

Be aware that some functions or asusctl need kernel level drivers support, take a look at the "Custom kernel section"

## Supergfxctl - graphics switching

The same rules as for asusctl, installing:

```bash
pacman -S supergfxctl switcheroo-control
```

Enable and start the systemd service:

```bash
systemctl enable --now supergfxd
systemctl enable --now switcheroo-control
```

supergfxctl can be used without asusctl. You can use `switcherooctl launch your_command` command in hybrid command to run selected programs on the dGPU. Any program/command run by the main command does inherit the select GPU (so launching Steam with switcherooctl will make run any game in it on the dGPU).

## ROG Control Center

ROG Control Center is a GUI tool for configuring few aspects of asusctl and supergfxctl. It was previously part of the asusctl package, but has now been separated as its own package in G14 repo. After adding the repo to `/etc/pacman.conf` as stated above, then you can install the tool:

```bash
pacman -S rog-control-center
```

![GUI Main Window](/images/guides/gui-main.png)

## Custom kernel - drivers fixes, hardware support

Newer devices often require custom kernel with patches, that kernel is also available in the g14 pacman repo, to install it just run:

```bash
pacman -Sy linux-g14 linux-g14-headers
```

Again, don't get confused by the name, it exists only for historical reasons. If you are using a custom kernel use the `nvidia-dkms` package for nvidia drivers, the regular `nvidia` package works only with stock Arch kernel.

At the date of updating this article devices from 2023 and older shouldn't need a custom kernel, fixes for 2024 are currently be worked on. Be aware that this state might change more often then this guide gets updated.

After installing the new kernel you need to regenerate your boot menu or add a new boot entry depending on what boot manager you are using. For GRUB that will be:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

For others refer to their documentation/Arch Wiki page. You can check currently booted kernel with command `uname -r`. It should give you for example:

```bash
6.8.1-arch1-g14-1
```

The `-g14` part is the important one.

## Other distros based on Arch

To ease the installation you can use other distro based on Arch, only Manjaro is highly not recommend (it is not really based on Arch and might not be compatible with points above and things might break with upgrades).

Recommended ones are: EndeavourOS, RebornOS, Garuda. Blacklisting nouveau might be still needed.

## EndeavourOS

When installing EndeavourOS don't use the option with Nvidia drivers preinstalled, this installs configs with might conflict with supergfxctl and installs the driver with works only with stock kernel. Use the default install option then install `nvidia-dkms` post install.
