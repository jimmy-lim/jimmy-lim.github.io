---
layout: post
title: From power button to i3
tags: IT
description: Reading my laptop's dmesg line by line to learn how Linux boots, all the way up to the desktop that replaced Windows.
---

I wanted to actually understand what happens between pressing the power button and seeing my desktop. So I dumped `dmesg` on my old Acer Aspire S5 (i5-7200U, 8 GB, NVMe, Debian 13) and read all 1,031 lines in order.

Not hunting for errors. Just reading it like a story. Turns out it is one.

## How to read a line

```
[    0.059489] hpet: HPET dysfunctional in PC10. Force disabled.
 └─ seconds    └─ subsystem   └─ message
    since the kernel started
```

`dmesg` only shows the **kernel's** messages. So the log starts at the moment the kernel is already running. Three things happened before line 1:

1. **UEFI firmware** powers on, checks the hardware, finds the EFI partition.
2. **GRUB** loads two files into RAM: the kernel (`vmlinuz`) and the **initramfs** (`initrd.img`).
3. GRUB jumps into the kernel. Line 1.

## The kernel finds out where it is (0.000 s)

The very first lines are the kernel introducing itself, then the command line GRUB handed it:

```
Command line: BOOT_IMAGE=/boot/vmlinuz-6.12.107+deb13-amd64 root=UUID=... ro quiet
```

- `root=UUID=...` is which partition holds the real root filesystem.
- `ro` is "mount it read-only first", so it can be checked before anything writes to it.
- `quiet` is "don't spam all of this on screen".

Then the firmware hands over the **memory map** (`BIOS-e820` lines): which physical address ranges are RAM and which belong to firmware or hardware. Then the **ACPI tables**: the firmware's description of everything the kernel can't discover by itself. Power buttons, battery, fans, sleep states, interrupt wiring. Some of those tables contain actual bytecode the kernel will run later.

At this point there's one CPU core, no drivers, and no memory allocator.

## Memory, then CPUs (0.01 to 0.08 s)

- `Zone ranges: DMA / DMA32 / Normal`: memory gets split into zones because some ancient devices can only reach the first 16 MB or 4 GB.
- `SLUB:`: the kernel's own memory allocator comes up. From here it can `malloc`, kernel-style.
- `Spectre V1 / V2 / RETBleed / MDS / ...`: a long list of CPU security workarounds. A 2016 CPU gets most of them.
- `smpboot: CPU0: Intel(R) Core(TM) i5-7200U`: everything so far ran on **one** core.
- `smp: Bringing up secondary CPUs ... #1 #2 #3`: the other three get woken up.
- `Memory: 7874272K/8253572K available`: the final RAM count.
- `devtmpfs: initialized`: `/dev` exists. Device files will appear as drivers find hardware.

## Finding the hardware (0.08 to 0.22 s)

`ACPI: Interpreter enabled` means the kernel is now running the firmware's bytecode. Then it scans the **PCI bus**, and every device answers with a vendor:device ID. On my laptop:

| PCI address | What it is |
|---|---|
| `00:02.0 [8086:5916]` | Intel HD 620 graphics |
| `00:14.0` | USB 3 controller |
| `00:15.x` | I²C controllers (the touchpad lives here) |
| `01:00.0 [8086:f1a5]` | NVMe SSD |
| `02:00.0 [168c:003e]` | Atheros Wi-Fi |
| `03:00.0 [10ec:5229]` | SD card reader |

`8086` is Intel's vendor ID. Someone at Intel had a sense of humour.

## The initramfs (0.24 s)

```
Trying to unpack rootfs image as initramfs...
```

This step is the clever bit. The kernel can't mount my SSD yet: the NVMe driver and ext4 support are modules that live **on** that SSD. Chicken, meet egg. So GRUB brought along a tiny filesystem in RAM with just enough drivers to reach the real disk.

Around the same time the console gets a framebuffer (`efifb: mode is 1920x1080x32`), the hardware clock sets the date, and the kernel loads its signing certificates.

## The most important line in the log (0.56 s)

```
Run /init as init process
```

The kernel is done setting itself up. It starts the **first user-space program**, PID 1, from the initramfs. From here on, user space drives the boot and the kernel just serves requests.

On Debian, `/init` is a shell script. It loads the storage drivers, finds `nvme0n1: p1 p2 p3`, checks swap for a hibernation image (`PM: Image not found`, so cold boot), and mounts root:

```
EXT4-fs (nvme0n1p2): mounted filesystem ... ro
```

Then `switch_root`: the real disk becomes `/`, and the script **replaces itself** with `/sbin/init`, which is systemd. Still PID 1, different program.

## systemd takes over (1.8 s)

```
systemd[1]: Queued start job for default target graphical.target.
```

That's the goal. systemd works backwards through dependencies to figure out everything required to get there, and starts as much as it can in parallel.

- `re-mounted ... r/w`: root becomes writable.
- `Started systemd-journald`: logging moves to `journalctl`.
- `systemd-udev-trigger`: udev replays an "add" event for every device the kernel found, which loads the rest of the drivers. That's why there's an avalanche of hardware in the next few seconds: lid switch, battery, webcam, Bluetooth, Wi-Fi, the real graphics driver (`i915`), sound.
- `Adding 8252412k swap`: swap on.

The last kernel line is Wi-Fi associating with my access point at **9.7 seconds**. Kernel's done.

## Where dmesg ends and my desktop begins

Here's the part `dmesg` doesn't show. I don't run a display manager. No GNOME, no KDE, no login screen. Just:

1. systemd starts `getty` on tty1: a plain text login prompt.
2. I log in. `login` starts `bash`.
3. I type `startx`.
4. `xinit` starts **Xorg**, the display server that owns the screen, keyboard and mouse. Its log lives in `~/.local/share/xorg/Xorg.0.log`, not in dmesg.
5. `xinit` runs my `~/.xinitrc`, which is one line: `exec i3`.
6. **i3**, a tiling window manager, takes over and runs its own autostart:

```
exec --no-startup-id dex --autostart --environment i3
exec --no-startup-id xss-lock --transfer-sleep-lock -- i3lock --nofork
exec --no-startup-id nm-applet
exec --no-startup-id nitrogen --restore
bar { status_command i3status }
```

That's the screen locker, the network tray icon, the wallpaper and a status bar. That's my whole desktop.

## The whole flow in one picture

```
UEFI firmware → GRUB → kernel                                  (not in dmesg)
  ├─ 0.000  memory map, ACPI tables, memory setup
  ├─ 0.06   interrupts, CPU security workarounds, start 4 CPUs
  ├─ 0.08   /dev, ACPI interpreter, PCI scan
  ├─ 0.24   unpack initramfs, console, keys
  └─ 0.56   ★ Run /init  (initramfs, PID 1)
       ├─ load NVMe / USB / touchpad drivers
       ├─ 1.60  mount root read-only
       └─ switch_root → systemd (still PID 1)
            ├─ 1.81  systemd starts, aims for graphical.target
            ├─ 2.38  root read-write, journald, udev
            ├─ 2.77  swap, buttons, battery, webcam
            ├─ 3–5.5 Bluetooth, AppArmor, Wi-Fi, GPU, audio
            ├─ 9.7   Wi-Fi connected                   ← dmesg ends here
            └─ getty on tty1 → login → bash
                 └─ startx → xinit
                      ├─ Xorg   (display server: screen, keyboard, mouse)
                      └─ ~/.xinitrc → exec i3   (window manager)
                           ├─ i3bar + i3status   (status bar)
                           ├─ nm-applet          (network tray)
                           ├─ xss-lock → i3lock  (lock on sleep)
                           ├─ nitrogen           (wallpaper)
                           └─ dex                (XDG autostart apps)
```

## So what replaced Windows, exactly?

Windows does all of the same jobs. It just doesn't show you the seams. Lined up side by side:

| Job | Windows | My laptop |
|---|---|---|
| Boot loader | Windows Boot Manager | GRUB |
| Kernel | `ntoskrnl.exe` | Linux |
| Early drivers before the disk is mounted | boot-start drivers loaded by `winload.efi` | initramfs |
| First process, starts services | `smss.exe` → `wininit.exe` → `services.exe` | systemd |
| Login | Winlogon + LogonUI | getty + `login` |
| Drawing the screen | DWM / `win32k` | Xorg |
| The desktop shell | `explorer.exe` | i3 |
| Taskbar and clock | taskbar | i3bar + i3status |
| Startup apps | Startup folder / Run keys | i3 `exec` lines + dex |
| Lock screen | Win+L | i3lock |
| Network tray icon | network flyout | nm-applet |
| Event Viewer | Event Viewer | `dmesg` + `journalctl` |

The difference is that on Windows every row is one company's binary, glued together and hidden. Here every row is a separate program I picked, and I can read a log line telling me exactly when it started. The whole thing fits in one page of ASCII.

## Try it on your own machine

- `dmesg -T`: the kernel log with real clock times.
- `journalctl -b`: everything from this boot, including systemd and services.
- `systemd-analyze`: how long firmware, loader, kernel and user space each took.
- `systemd-analyze critical-chain`: which services held the boot up.
- `lsinitramfs /boot/initrd.img-$(uname -r)`: what's inside your initramfs.
- `lspci -nn`: your own PCI table.

Read it top to bottom once. It's a much better story than I expected.
