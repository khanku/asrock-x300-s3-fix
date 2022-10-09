# ASRock DeskMini X300 S3 suspend fix

[Blog post](https://lorenz.brun.one/enabling-s3-sleep-on-x300/) with much more in-depth information
about this.

## Requirements

- A Cezanne-based APU (Ryzen 5X00G) (unless you want to be a guinea pig)
- A Linux-based operating system
- Latest firmware at the time of writing (1.70)

## :warning: WARNINGS

1. This is unsupported by both the vendor (ASRock) and me. I am not responsible for dead hardware or
   eaten cats. And depending on your jurisdiction it might void your warranty.
2. Any change to firmware settings can change the underlying ACPI tables. Since the patched one is
   not taken from the firmware you then have an inconsistent set of ACPI tables loaded which can
   lead to unpredictable behavior and in extreme cases even hardware damage. The only "safe" way to
   do this is to never change firmware settings or update the firmware after you've injected the
   patch. Otherwise you need to remove the hack first, change the settings and then redo the whole
   procedure.
3. This is only confirmed to work on Cezanne-based APUs (so the Ryzen 5x00G series) with TSME
   disabled. It might very well not work with other APUs.
4. It only works if your operating system supports overriding ACPI tables. I don't know how to do
   that on Windows and have never tested it.

## Instructions

```sh
cd /path/to/this/repo
sudo cp /sys/firmware/acpi/tables/SSDT1 ssdt1.aml
sudo cp /sys/firmware/acpi/tables/SSDT6 ssdt6.aml
sudo cp /sys/firmware/acpi/tables/DSDT dsdt.aml
./mkoverride.sh
```

After that you'll have a file named `acpi_dsdt_override.cpio` in the repository root. It needs
to be loaded before your initramfs, which can be done with grub2.

## Gentoo notes

- copy `iasl/iasl-accept-duplicates.patch` to e.g. `/etc/portage/patches/sys-power/iasl-20200717/` before merging `sys-power/iasl`
- run `bash -xe ./mkoverride.sh`

and then
- either use your kernel's `gen_init_cpio` (e.g. `/usr/src/linux-6.0.0-gentoo/usr/gen_init_cpio override_cpio_spec >acpi_dsdt_override.cpio`)
- or `mkdir -p overlay/kernel/firmware/acpi/; cp dsdt.aml !$` and use `genkernel --initramfs-overlay=path/to/overlay [...]`
  this requires CONFIG_ACPI_TABLE_UPGRADE=y, CONFIG_ACPI_TABLE_OVERRIDE_VIA_BUILTIN_INITRD=y and CONFIG_INITRAMFS_COMPRESSION_NONE=y
