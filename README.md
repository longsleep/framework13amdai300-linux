
# Running Pop!_OS 22.04 on Framework Laptop 13 AMD Ryzen AI 300 Series (2025)

```
System Information
        Manufacturer: Framework
        Product Name: Laptop 13 (AMD Ryzen AI 300 Series)
        Version: A9
        SKU Number: FRANVCCP09
        Family: Laptop
```

This is for now just my update/migration to this Laptop from a Lenovo T14-G3-AMD (by cloning the disk).

## Encountered issues

- The laptop keyboard does not work in ZFSBootMenu
- MESA fails to initialize when wayland is started and falls back to CPU
- The power-profiles-daemon cannot be installed as it conflicts with system76-power

I was already running Kernel 6.14 at the time of migration (including lastest linux-firmware),
so the system otherwise booted fine on the first try.

I managed to fix all these issues pretty quickly (see below for steps). Now the new Laptop is
ready for use.

Some other things I did:

- I installed a equalizer profile for the sound output (using easy effects).
- I tried tlp (latest version via ppa), but ultimatively settled on power-profiles-daemon for now.
- I built https://github.com/FrameworkComputer/framework-system? and played around with the EC.
- I am using an external display (LG UltraFine) via USB-C as sole connection and for power delivery,
  seems to work fine when the laptop is already on and booted - I have some trouble booting it with
  the lid closed and all. Further investigation follows.

Some might wonder, why not update to Ubuntu 24.04 (or even 25.04 for that matter). While that might
be fine in general, I am not ready to migrate to a past Gnome 3 desktop and my highly customized
desktop simply cannot be replicated completely with new Gnome 3 versions. So I am stuck at 22.04
for the time being.


### Fixing the laptop keyboard in ZFSBootMenu

The Framework 13 uses a i8042 based keyboard and trackpad. That Kernel module is not there in
binary builds of ZFSBootMenu. Adding it to the "dracut" configuration and rebuilding ZFSBootMenu
did the trick.

I already were building by own ZFSBootMenu anyways as i am using LUKS for encryption for
historic reasons.

```
cat dracut.conf.d/framework13-keyboard.conf                                                                                                      ✔
force_drivers+=" i8042 "
```

### Fixing 3D accellaration (MESA fails to initialize)

The MESA version in Pop!_OS 22.04 does know nothing about the GPU in the Framework 13
and hence does not initialize.

After adding https://launchpad.net/~kisak/+archive/ubuntu/turtle PPA and manually
installing the explicit versions of the "jammy" packages in that PPA, it works again.

Without that the system is not really usable as the CPU is constantly busy hard time
and gets very hot.

### Replacing system76-power with power-profiles-daemon

Get latest Power Profiles Daemon for AMD systems from https://launchpad.net/~superm1/+archive/ubuntu/ppd
PPA is easy, but it cannot be installed as the system76-power package (which is
pretty useless on the Framework 13) cannot be installed without reming pretty much
the whole desktop.

The solution is to create a "dummy" package to satisfy the dependency. This can be
easily done with https://github.com/Yanik39/Equivs.

```
sudo apt install equivs
````

Follow the equivs instructions for how to use it. Essentially I made this config
file file for it:

```
cat dummy-system76-power|grep -v '#'
Section: misc
Priority: optional
Standards-Version: 3.9.2

Package: dummy-system76-power
Version: 1.0
Provides: system76-power (= 1.1.0)
Description: Dummy package replacing system76-power
 To allow install of power-profile-daemon
```

After builting that, one can force remove system76-power and install the
newly built dummy instead, and everything is happy.

Now power-profile-daemon can be installed - oh happy day.