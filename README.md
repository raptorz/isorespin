# isorespin

This project is based on the original **Linuxium `isorespin.sh` v8.7.1**.
It respins Ubuntu-family desktop ISO images and adds the 32-bit UEFI support
needed by some older tablets and small-form-factor systems with a 64-bit CPU
but 32-bit firmware.

`isorespin.sh` is the project entry point. Its legacy path retains the
original customisation features, while its content-aware path handles newer
images without flattening their squashfs layout.

The original customisation features can:

- add or remove packages and repositories;
- add files, kernels, boot parameters, and firmware;
- run user-supplied pre- and post-install commands;
- add a 32-bit UEFI bootloader for compatible hardware; and
- create a new ISO suitable for writing to optical media or a USB device.

The legacy path is intended for Ubuntu, Ubuntu flavours, and Linux Mint desktop
images supported by the original Linuxium implementation. Compatibility still
depends on the particular ISO, firmware, storage layout, and hardware.

The modern path currently recognises Linux Mint 22.3 Cinnamon/Xfce and Ubuntu
or Xubuntu 26.04 desktop images. Linux Mint 20.3 remains on the legacy path.
It preserves the original kernel, initramfs, package database, layer structure,
and mixed BIOS/UEFI ISO layout while adapting the installer and installed-
system IA32 bootloader stage.

## Repository layout

```text
isorespin.sh          Primary entry point, based on Linuxium v8.7.1
lib/atom/             Historical Atom driver scripts and firmware
lib/isorespin/        Modern IA32 adapters and installer helpers
docs/                 Chinese notes and validation details
tests/                Automated regression and ISO-structure tests
```

## Atom-specific support files

The `lib/atom/` directory contains the historical files used by the original
Atom mode. They are kept as source material and firmware files, but the current
`isorespin.sh` no longer downloads or automatically injects the obsolete Atom
driver bundle. These files include:

- Broadcom wireless/Bluetooth driver installation scripts;
- UCM audio configuration installation scripts and wrappers; and
- firmware files for devices commonly found in Intel Atom tablets, including
  touchscreen and wireless/audio-related hardware.

The driver scripts may download or install additional packages. Review them
before use and make sure the target ISO and hardware match the intended Atom
configuration. The modern IA32 path does not promise Atom device-driver
support; graphics, Wi-Fi, audio, and touchscreen compatibility still require
separate hardware testing.

## Requirements

Run the script on a supported Linux host with root privileges. The legacy path
uses the tools required by the original script, including `bash`, `sudo`,
`xorriso`, `squashfs-tools`, `rsync`, `wget`, `unzip`, and the package-
management tools required by the selected options. Network access is normally
required when adding packages, repositories, kernels, or the Atom drivers.

The modern path additionally requires Python 3.10+, PyYAML, `mtools`,
`dpkg-dev`, and `ubuntu-keyring`. It uses an isolated APT state and does not
modify the host package database.

Because ISO extraction and squashfs repacking need substantial temporary disk
space, keep several times the size of the input ISO available in the working
directory.

## Basic usage

Display the built-in help:

```sh
./isorespin.sh --help
```

Build a respun image from an ISO using the legacy path:

```sh
sudo ./isorespin.sh -i /path/to/input.iso
```

Inspect a supported modern ISO without root or network access:

```sh
./isorespin.sh --inspect -i /path/to/linuxmint-22.3-xfce-64bit.iso
```

Build a modern Mint 22.3 image:

```sh
sudo ./isorespin.sh -i /path/to/linuxmint-22.3-xfce-64bit.iso
```

The modern path accepts `-i/--iso`, `-w/--work-directory`, `--inspect`, and
`--keep-work`. It writes `linuxium-<input-name>.iso` and a JSON build report.
The legacy path retains the broader option set documented by
`./isorespin.sh --help`; unsupported options on the modern path fail explicitly
instead of being silently ignored.

All generated images should be tested on the target firmware. A successful
build is not a guarantee that a particular tablet can run the desktop, enter
the installer, complete installation, or boot after the first restart.

## Modern IA32 implementation

The modern path is deliberately content-based. It identifies the distribution,
release, desktop edition, Casper layers, kernel configuration, and installer
from the ISO contents rather than trusting the filename or volume label. It
currently supports:

- Linux Mint 20.3 through the preserved legacy path;
- Linux Mint 22.3 Cinnamon and Xfce through the Ubiquity adapter; and
- Ubuntu and Xubuntu 26.04 desktop images through the Subiquity/curtin adapter.

The modern builder keeps the original kernel, initramfs, Casper UUID, package
database, squashfs layer graph, compression settings, and source selections.
It changes only the installer components needed for IA32 firmware. The build
does not run programs from the target filesystem and does not install or
remove ordinary desktop packages.

For Mint 22.3, the Ubiquity bootloader stage delegates to the shared
`lib/isorespin/install-ia32` helper only when the machine is booted with 32-bit
UEFI. Other firmware follows the original installer path. For Ubuntu/Xubuntu
26.04, the seeded installer snap is adapted with the same firmware check;
modified local snaps are marked unasserted and their self-refresh is disabled
so that the adapter is not silently replaced during installation.

The ISO includes an isolated, checksummed offline repository. It contains the
co-installable IA32 GRUB modules and the local `isorespin-ia32` maintenance
package. The builder checks dependencies against both an empty package state
and the package state recorded in the input image. It does not replace the
distribution's signed GRUB/shim packages or remove the Mint installer. After
installation, the maintenance package can regenerate IA32 GRUB files after
kernel or GRUB updates; the installed ESP must be mounted at `/boot/efi`.

The input ISO's complete boot layout is replayed, including BIOS El Torito,
UEFI, hybrid MBR/GPT/APM metadata, and the original x64 EFI file. The IA32 EFI
loader is checked in the actual EFI image, not only in the ISO directory tree.
Secure Boot must be disabled for the unsigned IA32 GRUB resources used by this
project.

## Validation status and limitations

The automated tests cover shell syntax, profile detection, kernel capability
checks, installer source-shape checks, preservation of the original embedded
boot archive, synthetic ISO replay, appended and embedded EFI layouts, and
layered installer snap adaptation. They are structural tests, not firmware
tests.

A local Mint 22.3 Xfce build has also been checked end-to-end at the file and
ISO-structure level: the output MD5 list passed, the original kernel/initramfs
and package database were preserved, the `sudo` setuid metadata was retained,
and the IA32/x64 EFI files were read back from the generated FAT image. The
image has not been validated on an IA32-UEFI tablet through Live boot,
installation, first reboot, or subsequent kernel/GRUB updates.

Newer desktops may exceed the memory or graphics capabilities of legacy Atom
tablets. Wi-Fi, audio, graphics, touchscreen, suspend, storage, and power
management support must be tested on the target hardware separately. The old
Atom driver bundle is not a long-term replacement for maintained kernel
drivers.

For the detailed Chinese notes, see [`docs/README_zh.md`](docs/README_zh.md).

## License and attribution

The project retains the original Linuxium licensing and attribution in
`isorespin.sh` and `LICENSE`. See those files for the full terms.
