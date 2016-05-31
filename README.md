# Hikey R-LCR + OP-TEE manifest

This repository contains patches, local manifest files and scripts
that can be used to build a Hikey R-LCR build that includes OP-TEE
feature with AOSP 4.4 kernel.

To use OPTEE with AOSP master for Hikey, please check instructions [here][3]

## 1. References

* [Hikey R-LCR build instructions][1]

## 2. Prerequisites

* Should already be able to build aosp.  Distro should have necessary
  packages installed, and the repo tool should be installed.
  Please note that the `mtools` package is installed, which is needed
  to make the hikey boot image.

## 3. Build steps

### 3.1. In an empty directory, clone the tree:
```bash
$ repo init -u https://android.googlesource.com/platform/manifest -b android-6.0.1_r43
```
### 3.2. Add the OP-TEE overlay:
```bash
$ cd .repo
$ git clone https://github.com/liuyq/optee_android_manifest -b hikey-marshmallow local_manifests
$ cd ..
```
### 3.3. Sync
```bash
$ repo sync
```
### 3.4. Download the HiKey vendor binary
```bash
$ wget https://dl.google.com/dl/android/aosp/linaro-hikey-20160226-67c37b1a.tgz
$ tar xzf linaro-hikey-20160226-67c37b1a.tgz
$ ./extract-linaro-hikey.sh
```
### 3.5. Apply patches for optee on hikey device config:
```bash
$ ./android-patchsets/hikey-optee
```
### 3.6. Apply patches for optee on hikey AOSP 4.4 kernel:
```bash
$ ./android-patchsets/hikey-optee
```
### 3.7. Configure the environment for Android
```bash
$ source ./build/envsetup.sh
$ lunch hikey-userdebug
$ export TARGET_BUILD_KERNEL=true
$ export TARGET_BOOTIMAGE_USE_FAT=true
```
### 3.9. Run the rest of the android build, For an 8GB board, use:
```bash
$ make -j32
```
For a 4GB board, use:
```bash
make -j32 TARGET_USERDATAIMAGE_4GB=true
```

## 4. Flashing the image
The instructions for flashing the image can be found in detail under
`device/linaro/hikey/install/README` in the tree.
1. Jumper links 1-2 and 3-4, leaving 5-6 open, and reset the board.
2. Invoke
```bash
./device/linaro/hikey/installer/flash-all.sh /dev/ttyUSBn
```
where the ttyUSBn device is the one that appears after rebooting with
the 3-4 jumper installed.  Note that the device only remains in this
recovery mode for about 90 seconds.  If you take too long to run the
flash commands, it will need to be reset again.

## 5. Partial flashing
The last handful of lines in the `flash-all.sh` script flash various
images.  After modifying and rebuilding Android, it is only necessary
to flash *boot*, *system*, *cache*, and *userdata*.  If you aren't
modifying the kernel, *boot* is not necessary, either.

This directory contains a prebuilt trusted firmware image `fip.bin`.
If you wish to build the trusted os from source, follow the HiKey
instructions in the [OP-TEE OS README][2].  After running the build,
the `fip.bin` file will be under
```
arm-trusted-firmware/build/hikey/release/fip.bin
```
## 6. Known issues
There are some known issues:
1. fip.bin is not the latest version, and would cause xtest 1013.x failed
2. xtest 1008.x failure is investigation.
3. Some projects are still in personal repository, the changes will be upstreamed soon
4. master version optee projects are used, will changed to stable version when released
Also bugs and comments are welcome.

[1]: https://source.android.com/source/devices.html
[2]: https://github.com/OP-TEE/optee_os/blob/master/README.md
[3]: https://github.com/linaro-swg/optee_android_manifest
