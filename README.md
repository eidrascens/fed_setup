# Fedora Silverblue Setup and Optimizations

Post install/upgrade recommendations and suggestions for Fedora Silverblue or `ostree` based Fedora (such as Kinoite).

> [!IMPORTANT]
> Originally, this repository was in `iaacornus/silverblue_postinstall-upgrade`. It was moved here to unify all of my previous and current works under this name

For anyone interested in contributing, see [Contributing](#contributing).

# Note / Disclaimer

> [!IMPORTANT]
> All commands in this repository was tested and used in the personal
> workstation and setup for quite a while (more than a year) by the owner
> of this repository. My personal experience notwithstanding, there might
> still be cases where the experience or result may vary. Proceed with caution
> and **DO NOT DO ANYTHING YOU DO NOT UNDERSTAND,** instead open an issue
> and ask away.

> [!WARNING]
> NEVER RUN A COMMAND FROM INTERNET WITHOUT ANY ANALYSIS AND CONSIDERATION OF ANY UNFORESEEABLE OR FORESEEABLE CONSEQUENCES.

> [!NOTE]
> You can skip all of the steps, majority are not required, but can be beneficial or can be useful later.

You can get the silverblue cheatsheet of Fedora's Team Silverblue [here](https://docs.fedoraproject.org/en-US/fedora-silverblue/_attachments/silverblue-cheatsheet.pdf).

# Table of Contents

Skip to what you need:

- [Fedora Silverblue Setup and Optimizations](#fedora-silverblue-setup-and-optimizations)
- [Note / Disclaimer](#note--disclaimer)
- [Table of Contents](#table-of-contents)
- [Update the System](#update-the-system)
  - [Update Base Image](#update-base-image)
  - [Update Preinstalled Flatpaks](#update-preinstalled-flatpaks)
  - [Update Firmwares](#update-firmwares)
- [Manage Repositories](#manage-repositories)
  - [Flathub](#flathub)
  - [RPMFusion](#rpmfusion)
    - [Reinstall Unversioned RPMFusion](#reinstall-unversioned-rpmfusion)
  - [Remove Unused Repositories](#remove-unused-repositories)
- [Codecs](#codecs)
  - [Thumbnail Support](#thumbnail-support)
  - [GStreamer](#gstreamer)
- [NVidia Drivers](#nvidia-drivers)
- [Flatpak Modifications](#flatpak-modifications)
  - [Permissions](#permissions)
  - [Unnecessary Flatpaks](#unnecessary-flatpaks)
  - [Replace Fedora Flatpaks](#replace-fedora-flatpaks)
  - [Theming](#theming)
    - [Theming Extended](#theming-extended)
- [System-Wide Changes and Optimizations](#system-wide-changes-and-optimizations)
  - [Replace RPM Firefox with Flatpak](#replace-rpm-firefox-with-flatpak)
  - [Disable `NetworkManager-wait-online.service`](#disable-networkmanager-wait-onlineservice)
  - [Disable GNOME Software](#disable-gnome-software)
  - [SSD Related Optimizations](#ssd-related-optimizations)
    - [Disable Workqueues](#disable-workqueues)
    - [Enable Discard](#enable-discard)
    - [Change to `noatime`](#change-to-noatime)
  - [Removable Base Image Packages](#removable-base-image-packages)
- [Laptop Users](#laptop-users)
  - [Set `intel_pstate=passive` for Improved Battery Life](#set-intel_pstatepassive-for-improved-battery-life)
  - [Checking Battery Consumption](#checking-battery-consumption)
  - [Deep Sleep](#deep-sleep)
  - [Battery Threshold](#battery-threshold)
  - [Keyboard Backlight](#keyboard-backlight)
- [Tips and Tricks](#tips-and-tricks)
  - [Revert to Previous Version](#revert-to-previous-version)
  - [Compare and Contrast](#compare-and-contrast)
- [Miscellaneous](#miscellaneous)
  - [Use FISH](#use-fish)
    - [Install FISH](#install-fish)
    - [Default Shell](#default-shell)
    - [Customize FISH](#customize-fish)
  - [Protection Against BadUSBs](#protection-against-badusbs)
  - [Mount External Drives](#mount-external-drives)
    - [Automatically Mount](#automatically-mount)
- [Contributing](#contributing)

---

# Update the System

## Update Base Image

After booting the system, GNOME Software will automatically download updates of your system. Hence, running `rpm-ostree upgrade` after boot would only give `stderr`. You can wait and reboot later, though usually GNOME gives notifications after the update. Although you can its status with:

```bash
rpm-ostree upgrade --check
```

> [!TIP]
> `rpm-ostree` relies on `polkit`, thus it does not need to be run with `sudo.`

You can obtain the summary of the changes or update, including added, removed and upgraded with

```bash
rpm-ostree upgrade --check
# this will also work
rpm-ostree upgrade --preview
```

## Update Preinstalled Flatpaks

Update your preinstalled flatpaks, this may also not be necessary, since this is automatically updated by GNOME software center, but if you want to be sure, do:

```bash
flatpak update
```

## Update Firmwares

It is also !important to update your firmware using `fwupdmgr`:

```bash
sudo fwupdmgr refresh --force
# get-devices need to have your device connected to AC
sudo fwupdmgr get-devices
sudo fwupdmgr get-updates
sudo fwupdmgr update
```

Reboot. You can also reboot later on.

---

# Manage Repositories

Note that some of the drivers may come preinstalled in your system, confirm before proceeding.

## Flathub

Fedora has its own flatpak repository where it filters some of the applications, for access to flathub setup the flathub repository:

```bash
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

## RPMFusion

The main repository of Fedora does not contain every applications, some of the codecs are in the RPMFusion, the NVidia drivers are in the nonfree, while some of the codecs are in free.

> [!TIP]
> Avoid layering to be base image as much as possible. Substantial number
> of layered packages will take a massive toll on system's performance.
> Consider installing some packages inside of a container or as a flatpak.

```bash
# Nonfree
rpm-ostree install https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Free
rpm-ostree install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm

# Both
rpm-ostree install https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
```

### Reinstall Unversioned RPMFusion

> [!CAUTION]
> This should only be done after installing the versioned RPMFusion (see above) and rebooting. Otherwise, this will yield errors.

The current RPMFusion installed by the command was version-specific as notable by `$(rpm -E %fedora)` in the command. Thus, rebasing for the next release would be a [problem](https://discussion.fedoraproject.org/t/simplifying-updates-for-rpm-fusion-packages-and-other-packages-shipping-their-own-rpm-repos/30364/23). Fortunately, it can be fixed by installing a "_general_" repo:

```bash
rpm-ostree update --uninstall rpmfusion-free-release --uninstall rpmfusion-nonfree-release --install rpmfusion-free-release --install rpmfusion-nonfree-release
```

## Remove Unused Repositories

Some repositories are enabled by default but are not used. You can disable it by:

```bash
sudo sed -i 's/enabled=1/enabled=0/' \
/etc/yum.repos.d/_copr:copr.fedorainfracloud.org:phracek:PyCharm.repo \
/etc/yum.repos.d/fedora-cisco-openh264.repo \
/etc/yum.repos.d/google-chrome.repo \
/etc/yum.repos.d/rpmfusion-nonfree-nvidia-driver.repo \
/etc/yum.repos.d/rpmfusion-nonfree-steam.repo
```

Although you may want to enable `fedora-cisco-openh264`.

# Codecs

## Thumbnail Support

Unfortunately, perhaps due to legal/patent reasons, `ffmpeg` is not included by default. You can install `ffmpegthumbnailer` to enable thumbnail support with:

```bash
rpm-ostree install ffmpegthumbnailer
```

## GStreamer

For intel (`intel-media-driver`) (_use `libva-intel-driver` for older versions of Intels_) and then the codecs:

```bash
rpm-ostree install ffmpeg gstreamer1-plugin-libav gstreamer1-plugins-bad-free-extras gstreamer1-plugins-bad-freeworld gstreamer1-plugins-ugly gstreamer1-vaapi intel-media-driver
```

For AMD users, refer [here](https://rpmfusion.org/Howto/OSTree)

Reboot again.

# NVidia Drivers

Check first if you have nvidia card with `/sbin/lspci | grep -e 3D`, it would show you something like this:

```bash
02:00.0 3D controller: NVIDIA Corporation GP108M [GeForce MX230] (rev a1)
```

Otherwise, you don't have nvidia card. If you have NVidia card you can install the proprietary driver from the nonfree repo:

```bash
rpm-ostree install akmod-nvidia
```

Finally, check your NVidia install with `modinfo -F version nvidia`, it should give the version number of your driver such as `510.60.02`, not `stderr`.

---

# Flatpak Modifications

Flatpak apps are sandboxed. Thus, may not work as expected. The following are some solutions to the errors that may arise from default Flatpak security permissions.

## Permissions

Other reddit users suggested, such as [u/IceOleg](https://www.reddit.com/user/IceOleg/), you can remove permissions for access to `$HOME` and `host` dir with:

```bash
flatpak override --user --nofilesystem=home
flatpak override --user --nofilesystem=host
```

The overriden directories can be returned with when required with `flatpak override --filesystem=home` or `host`.

[Flatseal](https://github.com/tchx84/flatseal) is also a good utility for managing permissions as [u/GunnarRoxen](https://www.reddit.com/user/GunnarRoxen/) suggested. It can be installed with:

```bash
flatpak install flathub com.github.tchx84.Flatseal
```

The flatpak modifications can be reset with

```bash
sudo flatpak override --system --reset
```

The `--system` flag can also be omitted and `--user` can be used for user-wide changes.

## Unnecessary Flatpaks

> [!CAUTION]
> Only do this if your storage space is sufficiently low. Otherwise, this yields little to no benefits.

There are also some preinstalled flatpak that you can safely remove. You can completely remove the flatpak with:

```bash
flatpak uninstall --system --delete-data <app>
```

Here are some you can remove:

1. Calculator `org.gnome.Calculator`
2. Calendar `org.gnome.Calendar`
3. Connections `org.gnome.Connections`
4. Characters `org.gnome.Characters`
5. Contacts `org.gnome.Contacts`
6. Extensions `org.gnome.Extensions`
7. Logs `org.gnome.Logs`
8. Loupe `org.gnome.Loupe`
9. Maps `org.gnome.Maps`
10. Nautilus Previewer `org.gnome.NautilusPreviewer`
11. Papers `org.gnome.Papers`
12. Snapshot `org.gnome.Snapshot`
13. Texteditor `org.gnome.TextEditor`
14. Weather `org.gnome.Weather`
15. Disk Usage Analyzer `org.gnome.baobab`
16. Clocks `org.gnome.clocks`
17. Font Viewer `org.gnome.font-viewer`

To remove all of this, refer to the section below.

## Replace Fedora Flatpaks

By default, Silverblue comes with flatpak applications installed from Fedora's flatpak repositories. These can be removed with:

```bash
flatpak uninstall --all --system --delete-data --assumeyes
```

These can be reinstalled, and some can be replaced with its new counterpart with the command:

```bash
flatpak install flathub org.gnome.Calculator org.gnome.Calendar org.gnome.Characters org.gnome.Connections org.gnome.Contacts org.gnome.Extensions org.gnome.Logs org.gnome.Loupe org.gnome.Maps org.gnome.NautilusPreviewer org.gnome.Papers org.gnome.Snapshot org.gnome.TextEditor org.gnome.Weather org.gnome.baobab org.gnome.clocks org.gnome.font-viewer
```

## Theming

One of the easiet approach to theme Flatpak applications is to install the flatpak version of GTK theme you are using, which you can find by using `search`:

```bash
flatpak search gtk3
```

Then you can install the said theme with `flatpak install flathub <theme_name>`. Another approach is by providing access to directories where the theme was installed, which depends on how it was installed:

```bash
# choose one, you can do all of them but I don't recommend doing it

# if installed in home dir
sudo flatpak override --system --filesystem=$HOME/.themes # if installed in home dir

# if layered in image
sudo flatpak override --system --filesystem=/usr/share/themes

# or whatever
sudo flatpak override --system --filesystem=xdg-data/themes
```

### Theming Extended

In some cases, where themes do not apply, especially in GTK4, it can be forced by including it in `$HOME/.profile`, as well as the settings (`settings.ini`):

> [!WARNING]
> Do not copy and execute the command, replace `<theme-name>` with the name of the theme

```bash
echo "export GTK_THEME=<theme-name>" >> $HOME/.profile;                         \
if [ ! -d $HOME/.config/environment.d/ ];                                       \
then mkdir -p $HOME/.config/environment.d/; fi;                                 \
echo "GTK_THEME=<theme-name>" >> $HOME/.config/environment.d/gtk_theme.conf;    \
echo "GTK_THEME=<theme-name>" >> $HOME/.config/gtk-4.0/settings.ini
```

This does the following: `echo "export GTK_THEME=<theme-name>" >> $HOME/.profile` append `export GTK_THEME=<theme-name>` to `$HOME/.profile`; create `$HOME/.config/environment.d/gtk_theme.conf` file with:

```bash
if [ ! -d $HOME/.config/environment.d/ ]; then
    mkdir -p $HOME/.config/environment.d/
fi

echo "GTK_THEME=<theme-name>" >> $HOME/.config/environment.d/gtk_theme.conf
```

And append `GTK_THEME=<theme-name>` at the end of the `gtk_theme.conf`; append `GTK_THEME=<theme-name>` to `settings.ini` config. If this didn't suffice, then, you can try:

```bash
sudo flatpak override --system --env=GTK_THEME='<theme-name>'
```

---

# System-Wide Changes and Optimizations

## Replace RPM Firefox with Flatpak

There are Fedora-specific changes in the source code of softwares deposited in Fedora repositories. One difference is that [Mozilla's Firefox (in Flathub / Flatpak) supports "profile-guided-optimization," whilst the Fedora's version does not](https://x.com/gnomealex/status/1207933145959555072). This apparently causes a loss of 15 - 20% in performance. Note, however, this report was a year ago by the time of this writing (16/10/26).

```bash
rpm-ostree override remove firefox firefox-langpacks
flatpak install flathub org.mozilla.firefox
```

Hereafter, enable Firefox to use Wayland if it does not already do (by Firefox 121, wayland should be the default):

```bash
sudo flatpak override --socket=wayland org.mozilla.firefox
sudo flatpak override --env MOZ_ENABLE_WAYLAND=1 org.mozilla.firefox
```

## Disable `NetworkManager-wait-online.service`

> In some multi-user environments part of the boot-up process can come from the network. For this case `systemd` defaults to waiting for the network to come on-line before certain steps are taken.

You can also disable `NetworkManager-wait-online.service`. It is simply a ["service simply waits, doing absolutely nothing, until the network is connected, and when this happens, it changes its state so that other services that depend on the network can be launched to start doing their thing."](https://askubuntu.com/questions/1018576/what-does-networkmanager-wait-online-service-do/1133545#1133545)

Disabling it can decrease the boot time of at least ~15s-20s, anecdotally:

```bash
sudo systemctl disable NetworkManager-wait-online.service
```

Masking it is not recommend, since as explained by [u/chrisawi](https://www.reddit.com/user/chrisawi/):

> Also, wait-online services are `WantedBy=network-online.target`, so they do nothing unless another service explicitly pulls that target in because it can't handle starting before the network is up. The nfs services are a typical example, see: `systemctl list-dependencies --reverse network-online.target`. It might be better to disable such services than to leave them potentially broken.

## Disable GNOME Software

GNOME Software launches automatically starts up after boot. This takes at least 100MB of RAM up to 900 MB (anecdotally). You can prevent GNOME Software from autostart by removing `/etc/xdg/autostart/org.gnome.Software.desktop`:

```bash
sudo rm /etc/xdg/autostart/org.gnome.Software.desktop
```

## SSD Related Optimizations

If you will proceed in this part, you need to obtain your device's unique identifier (`UID`). A `UID` is unique in each of your blocks/devices. You can find your partition/block/device's `UID` with `lsblk -p` or `fdisk -l`:

```bash
lsblk -p
NAME                MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
/dev/zram0          252:0    0   7.5G  0 disk  [SWAP]
/dev/nvme0n1        259:0    0 476.9G  0 disk
├─/dev/nvme0n1p1    259:1    0   600M  0 part  /boot/efi
├─/dev/nvme0n1p2    259:2    0     1G  0 part  /boot
└─/dev/nvme0n1p3    259:3    0 475.4G  0 part
  └─/dev/mapper/luks-<UUID>
                    253:0    0 475.3G  0 crypt /var/home
...
```

### Disable Workqueues

> [!WARNING]
> There are reported data loss on some and not on others, citing that the code of CloudFlare is buggy. I've tried it myself for about 2 years now (as of this commit), so far I didn't experienced any data loss. Moreover, I didn't encountered complains about it yet from zen kernel users, since zen kernel disabled it by default. But again, it may not be always the case.

Quoting [Arch Wiki](<https://wiki.archlinux.org/title/Dm-crypt/Specialties#Disable_workqueue_for_increased_solid_state_drive_(SSD)_performance>):

> Solid state drive users should be aware that, by default, discarding internal read and write workqueue commands are not enabled by the device-mapper, i.e. block-devices are mounted without the no_read_workqueue and no_write_workqueue option unless you override the default.

> The no_read_workqueue and no_write_workqueue flags were introduced by [internal Cloudflare research Speeding up Linux disk encryption](https://blog.cloudflare.com/speeding-up-linux-disk-encryption/) made while investigating overall encryption performance. One of the conclusions is that internal dm-crypt read and write queues decrease performance for SSD drives. While queuing disk operations makes sense for spinning drives, bypassing the queue and writing data synchronously doubled the throughput and cut the SSD drives' IO await operations latency in half. The patches were upstreamed and are available since linux 5.9 and up [[5](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/drivers/md/dm-crypt.c?id=39d42fa96ba1b7d2544db3f8ed5da8fb0d5cb877)].

In my setup, the encrypted partition is `/dev/nvme0n1p3`. This can be verified with `sudo cryptsetup isLuks /dev/<DEV> && echo SUCCESS` where device is the device name, e.g. `nvme0n1p3`, it should echo `SUCCESS` if the given partition is encrypted. Once confirmed, obtain the device name with:

> [!WARNING]
> `<UUID>`, `<UID>`, and `<LUKS-UUID>` are placeholders and must be replaced.

```bash
sudo dmsetup info luks-<UUID>
```

Which should output something like this:

```bash
sudo dmsetup info luks-e88105e1-690f-423e-a168-a9f9a2e613e9
Name:              luks-e88105e1-690f-423e-a168-a9f9a2e613e9
State:             ACTIVE
Read Ahead:        256
Tables present:    LIVE
Open count:        1
Event number:      0
Major, minor:      253, 0
Number of targets: 1
UUID: CRYPT-LUKS2-e88105e1690f423ea168a9f9a2e613e9-luks-e88105e1-690f-423e-a168-a9f9a2e613e9
```

Take the name, in this case `luks-e88105e1-690f-423e-a168-a9f9a2e613e9`, and execute the command:

```bash
sudo cryptsetup --perf-no_read_workqueue --perf-no_write_workqueue --persistent refresh <name>
```

Finally, reboot. This can be verified with `sudo cryptsetup luksDump /dev/<DEV> | grep Flags` whereas it should return something like:

```
Flags:        no-read-workqueue no-write-workqueue
```

### Enable Discard

> [!CAUTION]
> Due to [security implications](https://asalor.blogspot.com/2011/08/trim-dm-crypt-problems.html), `discard` option is not enabled by default. However, for majority it should not be significant. Thus, the performance improvement(s) outweight(s) the security concern(s).

In Fedora Silverblue, the `crypttab` (`/etc/crypttab`) is not passed into the `intramfs` images. But you can enable discard by passing it as kernel arguments using `rpm-ostree kargs --append-if-missing`:

```bash
# this enables discard for all
rpm-ostree kargs --append-if-missing=rd.luks.options=discard
# for specific
rpm-ostree kargs --append-if-missing=rd.luks.options=<LUKS-UUID>=discard
```

### Change to `noatime`

By default, the BTRFS subvolumes/volumes in Fedora Silverblue is mounted as `relatime` (along with `compress=zstd:1`[^1]). The main interest here is the option, `relatime`. Disabling `relatime` can possible reduce unnecessary writes on SSD. Thus, potentially reducing the amount of read-write cycle, insignificantly improving its lifespan. Although in my case, it has considerable benefits:

[^1]: BTRFS supports transparent compression, whereas the files are automatically compressed when written into the disk. In SSD, this reduces the amount of read-write cycle. Thus, prolonging its lifespan. Although, not really that significant.

```bash
sudo sed -i 's/compress=zstd:1/noatime,compress=zstd:1/' /etc/fstab
```

## Removable Base Image Packages

> [!CAUTION]
> This yields little to no benefit, and may become problematic. Only do if you know what you are achieving. Additionally, this needs to be reset before you can rebase to another version, _e.g._ 36 -> 37, refer [here](https://github.com/fedora-silverblue/issue-tracker/issues/288) for more information.

[u/VVine6](https://www.reddit.com/user/VVine6/) recommended some packages that can be removed from the base image, such as VM host support and GNOME classic shell:

```bash
rpm-ostree override remove open-vm-tools-desktop open-vm-tools qemu-guest-agent spice-vdagent spice-webdavd virtualbox-guest-additions gnome-shell-extension-apps-menu gnome-classic-session gnome-shell-extension-window-list gnome-shell-extension-background-logo gnome-shell-extension-launch-new-instance gnome-shell-extension-places-menu
```

Later on, before rebasing this needs to be included back, which can be done with `rpm-ostree override reset`.

---

# Laptop Users

## Set `intel_pstate=passive` for Improved Battery Life

For users with Sandy Bridge and later CPU (for Intel, [docs](https://www.kernel.org/doc/html/v4.19/admin-guide/pm/intel_pstate.html)) (for AMD, refer to Kernel document on [`amd_pstate`](https://docs.kernel.org/admin-guide/pm/amd-pstate.html)).

> intel_pstate is a part of the CPU performance scaling subsystem in the Linux kernel.

By default, `intel_pstate` is in "Active Mode" (`intel_pstate=active`):

> the driver bypasses the scaling governors layer of CPUFreq and provides its own scaling algorithms for P-state selection

Meanwhile, `intel_pstate=passive`:

> If the driver works in this mode, the scaling_driver policy attribute in sysfs for all CPUFreq policies contains the string `intel_cpufreq`. Then, the driver behaves like a regular CPUFreq scaling driver. That is, it is invoked by generic scaling governors when necessary to talk to the hardware in order to change the P-state of a CPU (in particular, the schedutil governor can invoke it directly from scheduler context).

This can be achieved by appending `intel_pstate=passive` to the Kernel arguments using `rpm-ostree kargs`:

```bash
rpm-ostree kargs --append-if-missing=intel_pstate=passive
```

## Checking Battery Consumption

There are several tools that can be installed to check the power consumption on your machine. However, for those who does not like to layer additional software like me:

```bash
awk \'{print $1*1e-6 " W"}\' /sys/class/power_supply/BAT0/power_now
```

This reads `/sys/class/power_supply/BAT0/power_now` and converts it to Watts (W). In case of Lunar Lake, one can achieve power consumption to as low as 3.4 W.

## Deep Sleep

> [!CAUTION]
> Only use deep sleep / suspend-on-ram if your laptop drains fast under `s2idle`

In some laptop, the battery drains rapidly when suspended under `s2idle`, particularly those with Alder Lake CPUs. One of the workaround is setting the kernel parameters with `mem_sleep_default=deep`. This can be done via `grubby`:

```bash
rpm-ostree kargs --append-if-missing="mem_sleep_default=deep
```

Do a reboot, then check it with `cat /sys/power/mem_sleep`, where the `deep` should be enclosed with brackets (`[deep]`).

## Battery Threshold

I recommend setting battery threshold of at least 80% to decrease wear on the battery. This can be done by echoing the threshold to `/sys/class/power_supply/BAT0/charge_control_end_threshold`. However, this resets every reboot, so it is good idea to make a systemd service for it:

```
[Unit]
Description=Set the battery charge threshold
After=multi-user.target
StartLimitBurst=0

[Service]
Type=oneshot
Restart=on-failure
ExecStart=/usr/bin/env bash -c 'echo 89 > /sys/class/power_supply/BAT0/charge_control_end_threshold'

[Install]
WantedBy=multi-user.target
```

## Keyboard Backlight

In some laptops whereas keyboard backlight do not work out of the box, it can be toggled with `brightnessctl`.

The keyboard backlight can be found in `/sys/class/leds`. It is usually named `::kbd_backlight/brightness` which can be contained in one more directory, in Asus laptops it is usually in `/sys/class/leds/asus\:\:kbd_backlight/brightness`. You can find out the current brightness by:

```bash
brightnessctl --device='<device>::kbd_backlight' info
```

If it is set to 0, it is disabled, in 1 it is in lowest, and as the number increment, the brightness increases. You can set the brightness by `brightnessctl --device='<device>::kbd_backlight' set 3`, for example in Asus laptops it is:

```bash
brightnessctl --device='asus::kbd_backlight' set 3
```

---

# Tips and Tricks

## Revert to Previous Version

There are cases where an update introduces some unwanted behavior—not
necessarily "brick"-ing changes—but something that can be irritating.

> [!TIP]
> Prevention is better than cure: before updating, always pin the current
> and working image with
>
> ```bash
> sudo ostree admin pin <index>
> ```
>
> Replace the `<index>` with an integer denoting the position of your
> current deployment (0-based).

Reverting to older deployment or image fixes such issues. However, if
you already updated and the previous system image was removed for some
reason, then that might be a problem. Fortunately, you can retrieve the
previous deployments / image of your OSTree system, and deploy it.

Get the logs with

```bash
sudo ostree pull --commit-metadata-only --depth=-1 fedora fedora/<version_num>/x86_64/<variant>
```

Here, replace the `<version_number>` with the actual version, _e.g._ `44`
for `44`, and `<variant>` with the actual _variant_, _e.g._ `silverblue`
for Silverblue, and so forth. This command usually takes a while, about
2 - 4 (_or more_) minutes. After a successful execution of that command, do

```bash
ostree log fedora:fedora/<version_num>/x86_64/<variant>
```

This will output something like this:

```
commit d1905a1486a1f2cdc97c06fd5c0df25d319a69fc75a59fc55235dba093a55049
Parent:  f81f0968681aa293d404d70bda2d62b44dba54bff799eb1cceb1d9ac110533bc
ContentChecksum:  5e9044b541173406bc00ec63caecbe8140693f364da62b8366e15150534b9b80
Date:  2026-10-27 00:37:35 +0000
Version: 44.20261027.0
(no subject)
```

Then look at the date, or the commit hash. If you can figure the date when
your previous deployment was released, you can now take the version number
in `Version`, _e.g._ `44.20261027.0` and deploy it with

```bash
rpm-ostree deploy 44.20261027.0
```

However, if you do not know the date, then compare the commit hash to the
current one with

```bash
rpm-ostree db diff <prev_commit> <current|new_commit>
```

You can obtain the `<current_commit>` with `rpm-ostree status`. This will
show what packages were updated, downgraded, removed, or added. Using that
information, revert accordingly.

## Compare and Contrast

This can be helpful in debugging as suggested by [u/VVine6](https://www.reddit.com/user/VVine6/)

```bash
sudo ostree admin config-diff | sort | grep -v system.control
```

> The output will list files as Removed, Added or Modified. The defaults are available in `/usr/etc` in the very same path, so to revert a modification or a removal simple copy the file over.

You can also utilize `diff` to obtain a more elaborate report:

```bash
sudo diff -yrW200 --suppress-common-lines --color=always /usr/etc /etc 2>/dev/null
```

---

# Miscellaneous

## Use FISH

> Fish (friendly interactive shell) is a smart and user-friendly command line shell that works on Linux, MacOS, and other operating systems. Use it for everyday work in your terminal and for scripting. Scripts written in fish are less cryptic than their equivalent bash versions.

FISH (Friendly Interactive SHell) is an alternative for BASH (Bourne Again SHell) and ZSH (Z SHell) which comes with out-of-the-box useful features such as:

- Syntax highlighting
- Web based configuration
- Inline searchable history
- Inline autosuggestion
- Tab completion using manpage data

DEMO (Credits to Sid Mohanty, [link to original article, suggested read for more info](https://betterprogramming.pub/fish-vs-zsh-vs-bash-reasons-why-you-need-to-switch-to-fish-4e63a66687eb?gi=fc345308724e))

![](https://miro.medium.com/max/640/1*AhoFOHQxoLzKiLMg-NVRKQ.gif)

FOR INTERESTED:

- [https://opensource.com/article/20/3/fish-shell](https://opensource.com/article/20/3/fish-shell)
- [https://fedoramagazine.org/fish-a-friendly-interactive-shell/](https://fedoramagazine.org/fish-a-friendly-interactive-shell/)

### Install FISH

Install `fish` with:

```bash
rpm-ostree install fish
```

You can also install it inside `toolbox`:

```bash
sudo dnf install install fish
# or if outside toolbox
toolbox run sudo dnf install fish
```

### Default Shell

Since Fedora does not have `chsh` in the base image of Silverblue due to its `setuid` root, you can use `usermod` to set the default shell:

```bash
# after reboot
sudo usermod --shell /usr/bin/fish $USER
```

### Customize FISH

FISH comes with web-based configuration which can be access with:

```bash
fish_config
```

This will give a GUI where you can set your prompt, color of syntax highlighting, aliases, functions (view of defined functions). Then to disable the welcome message you can run (once):

```bash
set -U fish_greeting
```

## Protection Against BadUSBs

## Mount External Drives

Some of the external drives will not be automatically mounted by the system if it was not mounted/specified during the installation.

External drives, along with its `UUID` (Universally Unique Identifier), can be found using `fdisk -l` and be mounted using:

```bash
sudo mount /dev/sdX <dir>
# replace <dir> where you want to mount the drive
# sdx can be nvmeNnJpI where N, J and I are integers
# e.g. /dev/nvme0n1p1, they can also be /dev/sdb
```

### Automatically Mount

External drives can be automatically mounted on boot by including it into `/etc/fstab`.

Firstly, you need to obtain the `UUID` of the drive and make sure its mount point exists. You can find the drives and their corresponding `UUID` with `lsblk -f` or `fdisk -l`. Then, add it to `/etc/fstab` with format of:

```bash
UUID    mount_point     type    options     dump    fsck
```

Below is an example, **DO NOT COPY AND PASTE**:

```
# Ignore the comments, this is and example to fstab entry, don't copy and paste this, your system won't boot
# UUID                                      # mount point (full), also  # filesystem format   # options # dump # fsck
#                                           # no env variables such as
#                                           # $HOME
# UUID=e423cfe8-5e8a-419c-87d0-8abb39aa498c /var/home/iaacornus/Storage ext4               defaults 0       0
# UUID=<your device uuid>                   <mount point>               <filesystem format> <options> <dump>  <fsck>
```

Instead of using `<device>`, it is highly recommended to use the `UUID` instead. Moreover, I strongly encourage to use `defaults` for `options`, `0` for `dump` and `fsck` to disable the checking (increasing the boot time, and avoiding potential errors, and since you only do checking if the drive is part of the OS filesystem), refer to [ArchWiki - fstab](https://wiki.archlinux.org/title/fstab).

Finally, check `/etc/fstab` with `cat /etc/fstab`. Be sure to input the correct UUID and options, otherwise your system won't boot.

---

# Contributing

All contributions are welcome, except those that were generated by AI.
If you were to submit contributions **assissted**—not generated—by AI,
use the trailer `Assisted by: <LLM>` to be considered.

Undeclared used of AI will be severely dealt with, and lead to permanent
ban in this repository.

Moreover, all commands and changes recommended herein must be **TESTED**
in the latest version. If not, at least there should be previous guide
from where it was based upon.
