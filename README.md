# isorespin

This project is based on the original **Linuxium `isorespin.sh` v8.7.1**.
It respins Ubuntu-family desktop ISO images and adds the 32-bit UEFI support
needed by some older tablets and small-form-factor systems with a 64-bit CPU
but 32-bit firmware.

The original script is kept as `isorespin.sh`. It can customise a supported
ISO by unpacking the image, applying the requested changes, and rebuilding a
bootable ISO. Depending on the selected options, it can:

- add or remove packages and repositories;
- add files, kernels, boot parameters, and firmware;
- run user-supplied pre- and post-install commands;
- add a 32-bit UEFI bootloader for compatible hardware; and
- create a new ISO suitable for writing to optical media or a USB device.

The script is intended for Ubuntu, Ubuntu flavours, and Linux Mint desktop
images supported by the original Linuxium implementation. Compatibility still
depends on the particular ISO, firmware, storage layout, and hardware.

## Atom-specific support files

The `lib/isorespin/` directory contains files used by the original `--atom`
mode. They are not a replacement for the main script; they are copied or
called by the Atom-specific installation steps when that mode is selected.
These files include:

- Broadcom wireless/Bluetooth driver installation scripts;
- UCM audio configuration installation scripts and wrappers; and
- firmware files for devices commonly found in Intel Atom tablets, including
  touchscreen and wireless/audio-related hardware.

The driver scripts may download or install additional packages. Review them
before use and make sure the target ISO and hardware match the intended Atom
configuration.

## Requirements

Run the script on a supported Linux host with root privileges. The host should
have the tools used by the original script, including `bash`, `sudo`,
`xorriso`, `squashfs-tools`, `rsync`, `wget`, `unzip`, and the package-
management tools required by the selected options. Network access is normally
required when adding packages, repositories, kernels, or the Atom drivers.

Because ISO extraction and squashfs repacking need substantial temporary disk
space, keep several times the size of the input ISO available in the working
directory.

## Basic usage

Display the built-in help:

```sh
./isorespin.sh --help
```

Build a respun image from an ISO:

```sh
sudo ./isorespin.sh -i /path/to/input.iso
```

Enable the original Atom-specific additions when appropriate:

```sh
sudo ./isorespin.sh -i /path/to/input.iso --atom
```

The complete option set and supported distributions are documented by the
script itself. Always test the resulting ISO in the target device's firmware
mode before installing it.

## License and attribution

The project retains the original Linuxium licensing and attribution in
`isorespin.sh` and `LICENSE`. See those files for the full terms.
