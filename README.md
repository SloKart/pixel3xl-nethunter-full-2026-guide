# pixel3xl-nethunter-full-2026-guide
A TWRP-less Kali NetHunter Full install guide for the Pixel 3 XL (crosshatch) on stock Android 12, using Magisk, KernelFlasher, and the Alynx NetHunter kernel.
Pixel 3 XL (Crosshatch) — Kali NetHunter Full on Android 12

⚠️ Current Kali Rolling / Alynx 4.9 Warning

The install in this guide works, but there is currently a compatibility problem between the old Alynx 4.9.x kernel used by the Pixel 3 XL and newer Kali Rolling systemd / udev packages.

Do not immediately run a normal apt full-upgrade after installing the chroot.

Doing so can leave systemd, udev, and several NetHunter/Kali metapackages partially configured with errors such as:

Protocol driver not attached

Until the kernel compatibility issue is properly fixed, hold the systemd/udev family first:

apt-mark hold \
  systemd \
  systemd-sysv \
  libpam-systemd \
  libsystemd-shared \
  libsystemd0 \
  libudev1 \
  udev


apt update
apt full-upgrade

If APT reports those packages as Not Upgrading, that is intentional.

I am currently working on a proper long-term fix by looking at backporting the newer NetHunter statx support into the Alynx 4.9 Pixel 3 XL kernel. Until then, treat the package hold as a workaround, not a real fix.

Yes, it is stupid that a fresh NetHunter install can break itself by doing a normal Kali upgrade. Welcome to running a 2026 userspace on a 2018 phone.


I put this together because most of the Pixel 3 XL NetHunter guides still revolve around TWRP, and on Android 12 that path is a pain in the ass.

The basic problem is encryption. TWRP on the Pixel 3 XL can boot, but depending on the build you end up with broken decryption, a frozen touchscreen, recovery crashing, or some combination of the three. After screwing with it enough, I stopped trying to make TWRP part of the process at all.

You don't need it.

This setup uses:

Stock Android 12
Magisk patched boot.img
KernelFlasher
Alynx NetHunter kernel
NetHunter Full
ADB/Fastboot from Linux

The end result is a rooted Pixel 3 XL running the full NetHunter environment with the custom kernel support needed for things like Wi-Fi injection, external wireless adapters, HID/BadUSB, etc.

The important part is matching the Android build and kernel correctly. Some of the other versions can move around. Those cannot.

What you need
Stock Android 12 Factory Image

Use:

SP1A.210812.016.C2

This is the final Android 12 build for the Pixel 3 XL.

Factory image:

Stock Android 12 Factory Image

This version matters.

If the phone is currently running Android 13, 14, CalyxOS, or anything else, flash this first.

The Alynx kernel we're using is built for the Android 12 base. This is not one of those things where being "close enough" is good enough.

Magisk

Magisk Releases

The current version is fine.

Example:

Magisk-v30.7.apk

No need to hunt down whatever version happened to exist when an older guide was written.

Alynx NetHunter Kernel + Wireless Firmware

Alynx NetHunter Kernel for Pixel 3 / Pixel 3 XL

The downloads are attached near the bottom of the first post.

You need:

Alynx-12-nethunter-bluecross.zip
Wireless_firmware.zip

Again, the kernel version matters.

Use:

Alynx-12-nethunter-bluecross.zip

Do not grab Alynx-13 or Alynx-12L because they sound newer or better. They are for different Android bases.

KernelFlasher

KernelFlasher Releases

The newest release should be fine.

This replaces the part of the old process where TWRP would normally flash the custom kernel.

Kali NetHunter Full

Kali NetHunter Images

Grab the current Full package for crosshatch.

It should look something like:

kali-nethunter-xxxx.x-crosshatch-thirteen-full.zip

The thirteen in the filename does not mean the phone needs Android 13.

The phone is still running Android 12.

1. Flash stock Android 12

If you're already running:

SP1A.210812.016.C2

you can skip this.

Otherwise, extract the factory image on your Linux machine.

Reboot the phone into the bootloader:

adb reboot bootloader

Go into the extracted factory-image directory and run:

./flash-all.sh

This wipes the phone.

When Android comes back up, skip through setup far enough to enable:

Developer Options
USB Debugging

There's no reason to fully set the phone up yet. We're about to change half of it anyway.

If flash-all.sh gives you permission denied

Make it executable:

chmod +x flash-all.sh

Then run it again.

If flashing immediately fails

Make sure the bootloader is unlocked.

If necessary:

fastboot flashing unlock

You may need to enable OEM Unlocking under Developer Options first.

2. Root with Magisk by patching boot.img

This is the part where I stopped following the older TWRP instructions.

Instead of booting TWRP and hoping Android 12 encryption decides to cooperate, just patch the stock boot image directly.

Inside the extracted Android factory image is another ZIP named something like:

image-crosshatch-sp1a.210812.016.c2.zip

Extract it.

You want:

boot.img

Push it to the phone:

adb push boot.img /sdcard/Download/

Install Magisk:

adb install Magisk-v30.7.apk

Use whatever the actual APK filename is for the version you downloaded.

Open Magisk on the phone.

Go to:

Install
→ Select and Patch a File

Select:

boot.img

from Downloads.

Magisk will create a patched image with a name similar to:

magisk_patched-[random].img

Pull that back to the computer:

adb pull /sdcard/Download/magisk_patched-[random].img

Now reboot to the bootloader:

adb reboot bootloader

Flash the patched image:

fastboot flash boot magisk_patched-[random].img

Then:

fastboot reboot

Once Android boots, open Magisk and verify root is working.

That's it. No TWRP required.

cannot stat 'boot.img'

You're probably running adb push from a directory that doesn't actually contain boot.img.

Check:

ls

Make sure you extracted the inner image ZIP and are working from the right directory.

TWRP boots but touch is frozen

Yeah. That's the problem.

Force reboot the phone and use the Magisk boot-patching method above instead.

3. Flash the Alynx NetHunter kernel

Now that the phone is rooted, install the custom kernel.

Push the kernel ZIP to Downloads:

adb push Alynx-12-nethunter-bluecross.zip /sdcard/Download/

Install KernelFlasher:

adb install KernelFlasher-v1.0.0-alpha20.apk

Use the actual filename of the version you downloaded.

Open KernelFlasher.

Magisk will ask whether you want to grant it root.

Grant it.

KernelFlasher should show the active slot, either A or B.

Select the active slot and choose:

Flash AK3 zip

Then select:

Alynx-12-nethunter-bluecross.zip

Let it flash and reboot when it finishes.

KernelFlasher can't find the ZIP

Put it in:

/sdcard/Download/

Android's scoped storage can make files dumped directly into /sdcard/ disappear from app file pickers.

I just put the flashable stuff in Downloads and avoid the bullshit.

Bootloop after flashing the kernel

First thing to check is whether you flashed the correct Alynx build.

This guide uses:

Alynx-12-nethunter-bluecross.zip

on:

SP1A.210812.016.C2

If you mixed Android and kernel versions, I'd reflash the stock factory image and start from a known-good state rather than trying to repair an unknown mess.

4. Install the wireless firmware and NetHunter Full

At this point the phone should have:

Android 12
Magisk root
Alynx NetHunter kernel

Now install the remaining pieces.

Push the wireless firmware:

adb push Wireless_firmware.zip /sdcard/Download/

Push NetHunter Full:

adb push kali-nethunter-2026.2-crosshatch-thirteen-full.zip /sdcard/Download/

Obviously use the current filename if the release has changed.

Open Magisk and go to:

Modules
→ Install from storage

Install:

Wireless_firmware.zip

When it finishes, do not reboot yet.

Go back and install:

kali-nethunter-xxxx.x-crosshatch-thirteen-full.zip

The NetHunter package is big and has a shitload of files to extract, so this part can take a while.

If it sits on:

Extracting

for 10 or 20 minutes, that doesn't necessarily mean it's frozen.

Let it finish.

Once everything installs successfully, reboot.

Post-install

After Android comes back up, open NetHunter.

Grant it root through Magisk and give it the permissions it needs for storage, USB hardware, location, and the other NetHunter functions.

Then go into:

Kali Chroot Manager

and verify the Kali environment starts correctly.

At that point the stack should basically be:

Pixel 3 XL
│
├── Stock Android 12
│   └── SP1A.210812.016.C2
│
├── Magisk
│
├── Alynx NetHunter Kernel
│
├── NetHunter Wireless Firmware
│
└── Kali NetHunter Full

And that's really the whole trick.

Most of the older Pixel 3 XL instructions aren't fundamentally wrong; they're just built around TWRP being part of the process.

On Android 12, I found it easier to remove TWRP from the equation completely.

Patch the stock boot image with Magisk, flash the NetHunter kernel from the rooted OS, install the remaining modules, and move on with your life.

modern systemd/udev vs Alynx 4.9
/tmp inheriting Android’s /data/local/tmp
/dev/null/mount namespace weirdness when entering the chroot incorrectly from ADB
use su -mm when driving NetHunter through ADB
keep a known-good chroot backup before experimenting
