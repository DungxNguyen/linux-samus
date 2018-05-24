# Linux for Chromebook Pixel 2015

This repository contains packages for Debian and Arch Linux that installs the Linux kernel v4.10 with
a config that is somewhat optimized for the Chromebook Pixel 2015.

*As of v4.9 there is no need to patch the kernel sources to get sound support*.

*Current kernel version: v4.16.11*

## Installation

The easiest way to get going is to install the packages if you are running
Ubuntu, Debian or Arch Linux.

### Ubuntu / Debian

``` bash
$ git clone --depth=1 https://github.com/raphael/linux-samus
$ cd linux-samus/build/debian
$ sudo dpkg -i *.deb
```

### Arch Linux

Install the [`linux-samus4`](https://aur.archlinux.org/packages/linux-samus4/) package from the AUR:
```sh
$ yaourt -S linux-samus4
```

### Other distributions

The entire kernel patched tree is located under `build/linux`, compile and install using the usual
instructions for installing kernels. For example:
``` bash
$ git clone --depth=1 https://github.com/raphael/linux-samus
$ cd linux-samus/build/linux
$ make nconfig
$ make -j4
$ sudo make modules_install
$ sudo make install
```
> *NOTE* the steps above are just the standard kernel build steps and may
> differ depending on your distro/setup.

## Post-install steps

Once installed reboot and load the kernel.

### Sound

#### Setup

The scripts needed to enable audio and switch between the speakers and 
headphones are provided in the `scripts/setup/audio` folder.

- Enable audio: `scripts/setup/audio/enable-audio.sh`
- Switch to speakers: `scripts/setup/audio/enable-speakers.sh`
- Switch to headphones: `scripts/setup/audio/enable-headphones.sh`
- Toggle mute: `scripts/setup/audio/mute-toggle.sh`
- Decrease volume: `scripts/setup/audio/volume-down.sh`
- Increase volume: `scripts/setup/audio/volume-up.sh`

#### Users upgrading from pre-4.7 Samus kernel

To get sound working, you must unwind some of the previous samus sound settings:

1) edit the file `/etc/pulse/default.pa` and ensure these lines are commentted out:
```
#load-module module-alsa-sink device=hw:0,0
#load-module module-alsa-source device=hw:0,1
#load-module module-alsa-source device=hw:0,2
```
2) remove the following folders:  `/opt/samus`  and  `/usr/share/alsa/ucm/bdw-rt5677`  
3) edit the file `/etc/acpi/handler.sh` and remove any samus entries  

NOTE: settings to toggle headphone/speaker during `plug)` and `unplug)` events still need to be implemented.

### Touchpad

Since Linux 4.3 the Atmel chip needs to be reconfigured to guarantee that the touchpad works.
See [issue #73](../../issues/73) for details. The `linux-samus/scripts/setup/touchpad` directory contains a script
that does the reconfig:
```sh
$ cd linux-samus/scripts/setup/touchpad
$ ./enable-atmel.sh
```

This is only needed to be run once.

### Keyboard

Chrome OS sets up a number of useful key mappings, such as:

 * Home = Search + left arrow
 * End = Search + right arrow
 * PgUp = Search + up arrow
 * PgDn = Search + down arrow
 * Delete = Search + backspace

The `keyboard.sh` script will enable these mappings under standard Linux,
and it will also set up mappings for special functions that do not exist
on a standard 101-key PC keyboard:

 * Back = Search + F1
 * Forward = Search + F2
 * Reload = Search + F3
 * BrightnessDown = Search + F6
 * BrightnessUp = Search + F7
 * VolumeMute = Search + F8
 * VolumeDown = Search + F9
 * VolumeUp = Search + F10

i.e. the Chromebook's F-keys will produce F1-F10 _without_ the Search modifier,
and perform the marked functions _with_ the Search modifier.  F1-F3 can be
handled by apps (such as a web browser) while F6-F10 tend to be handled by
the desktop environment.  The latter keys were tested with Xfce.

This will add a command to `~/.xsessionrc` that enables the mappings on login:
```sh
$ cd linux-samus/scripts/setup/keyboard
$ ./keyboard.sh
```

### Xorg

To enable X11 acceleration run the `xaccel.sh` script:
```sh
$ cd linux-samus/scripts/setup/xorg
$ ./xaccel.sh
```

### Brightness

The script `scripts/setup/brightness/brightness` can be used to control the
brightness level.
```sh
$ cd scripts/setup/brightness
$ ./brightness --help
Increase or decrease screen brightness
Usage: brightness --increase | --decrease
```
Put `scripts/setup/brightness` in your path and bind the F6 key to
`brightness --decrease` and the F7 key to `brightness --increase` for an
almost native experience.

Similarly the script `script/setup/brightness/keyboard_led` can be used to
control the keyboard backlight, bind the ALT-F6 key to
`keyboard_led --decrease` and ALT-F7 to `keyboard_led --increase`.

Both these scripts require write access to files living under `/sys` which
get mounted read-only for non-root users on boot by default. If your system
uses `systemd` (e.g. ArchLinux) then the file
`script/setup/brightness/enable-brightness.service` contains the definition
for a systemd service that makes the files above writable to non-root user.
Run `systemctl enable enable-brightness.service` for the service to run on boot.

##### systemd
```sh
./setup.systemd.sh
```
The same directory also contains `setup.systemd.sh`. When executed, it copies
scripts to `/usr/local/bin` and configures systemd to run the script
`enable-brightness.sh` on boot.

##### OpenRC
```sh
./setup.openrc.sh
```
The same directory also contains `setup.openrc.sh`. When executed, it copies
scripts to `/usr/local/bin` and configures OpenRC to run the script
`enable-brightness.sh` on boot using the `local` service.

## Contributions

This repo exists so that we can all benefit from one another's work.
[Thomas Sowell's linux-samus](https://github.com/tsowell/linux-samus) repo
was both an inspiration and help in building it. The hope is that others
(you) will also feel inspired and contribute back. PRs are encouraged!
