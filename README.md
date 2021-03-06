# MicroRenovator

![MicroRenovator](https://github.com/syncsrc/syncsrc.github.io/blob/master/public/microrenovator.png?raw=true)

# WARNING

_This application is designed to modify the EFI partition and bootloader of
your system. Users acknowledge that running this program can result
in corruption of the operating system and loss of data._


## Background

The mitigations for [Spectre](https://spectreattack.com/) require updates to
both operating system kernels and
[processor microcode](https://www.intel.com/content/dam/www/public/us/en/documents/sa00115-microcode-update-guidance.pdf).
While microcode updates can be deployed at runtime, not all operating system
vendors are redistributing processor microcode updates, instead relying on
hardware manufactures to supply it through updated platform firmware. However,
millions of system are no longer receiving manufacturer support, and so have
no means to apply the processor microcode patches, leaving them unable to
mitigate the Spectre vulnerabilities.

MicroRenovator provides a mechanism for deploying processor microcode that is
independent of both manufacturer and operating-system supplied updates, by
adding a custom EFI boot script which loads microcode prior to the operating
system being run. This enables the operating system kernel to detect the updated
microcode and enable Spectre mitigations that depend on it.

## Usage

Boot the target system using a linux LiveCD or USB, such as
[Fedora](https://getfedora.org/) or [Ubuntu](https://www.ubuntu.com/download)

Clone this repository
```
git clone https://github.com/syncsrc/MicroRenovator.git
```
Then run uRenovate.sh to install the microcode updater
```
./uRenovate.sh
```
The installer will perform the following actions:
1. find appropriate microcode for the current system
2. attempt to locate an EFI partition
3. find the bootloader on that partition
4. copy the included microcode updater to the EFI partition
5. add a startup script to run the microcode updater prior to the OS bootloader

To uninstall, run
```
./uRenovate.sh -u
```


## Offline Usage

The kickstart file can be used to build a custom LiveCD image based on Fedora
27 that includes all the necessary files and packages to build and install the
microcode loader application.

Install the LiveCD Creator utility and the sample kickstart files, and make a
local copy of the kickstart files to work with.
```
dnf -y install livecd-tools spin-kickstarts
cp /usr/share/spin-kickstarts/\*.ks .
```
The LiveCD utility will need to be modified to launch the correct OS on boot.
```
sed -i 's/set default="1"/set default="0"/' /usr/lib/python3.6/site-packages/imgcreate/live.py
```
Finally, run LiveCD-Creator to build the ISO
```
livecd-creator --verbose --config=reno-live.ks --fslabel=URENO
```
The resulting URENO.iso file is a bootable image that can be burned to a DVD or
USB drive like any other live image. Once booted into this live image, simply
run the uRenovate.sh installer script.


## Building EFI Utilities

Building the EFI applications requires an
[EDK2 environment](https://github.com/tianocore/tianocore.github.io/wiki/Common-instructions).

Copy the Uload directory into the edk2/ folder of a configured EDK2 environment,
and run the following:
```
build -a X64 -p ShellPkg/ShellPkg.dsc -b RELEASE
build -a X64 -p Uload/Uload.dsc -b RELEASE
```

To use the resulting files instead of the provided .efi binaries, change the
"edk2_dir" in uRenovate.sh to point at the desired edk2/ directory.

If using a LiveCD created by the MicroRenovator kickstart file, running the
included build_efi.sh script will generate the necessary files.


## Known Issues
* Not compatible with Sleep (S3). Hibernate is not impacted
* Not currently compatible with UEFI secure boot
* Windows sometimes fails to boot after running Uload.efi, rebooting usually resolves the problem


## ToDo
* error handling in EFI application and script
* re-implement as DXE driver (to support S3)
