# `intel-vaapi-driver` with h264 G45 support

## The issue

Some Gen-4 intel GPUs (GMA X4500HD, GMA 4500MHD, X4700MHD) support hardware acceleration of h264 video decoding, but the user-space driver from Intel doesn't offer it.

## Upstream

The video acceleration user-space driver was maintained at:
- https://cgit.freedesktop.org/vaapi/intel-driver (until 2017.02.18)
- https://github.com/intel/intel-vaapi-driver (until 2024.10.29)

The driver is no longer maintained by Intel with `intel-vaapi-driver` successor https://github.com/intel/media-driver supporting Gen-8 and up.

## Patches

Patches that enable h264 in G45 are available at https://bitbucket.org/alium/g45-h264/downloads.  
I am not aware who is the author of the code. It might be the Bitbucket's user called `alium` or someone else.  
Please open the issue if you know the details where this code comes from.

## This project

This project is a fork of https://github.com/intel/intel-vaapi-driver `v2.4-branch` and `master` branches with applied patches from https://bitbucket.org/alium/g45-h264/downloads.

I created this repository to preserve the patches in a transparent way. Archives that are available in Bitbucket feel like a black-box without a real option for further contribution.

## Get

```sh
git clone https://github.com/w8jcik/intel-vaapi-driver.git intel-vaapi-driver-src
```

You can specify branch `v2.4-branch-g45-h264`, `master-g45-h264`, `master-experimental`.

- `v2.4-branch-g45-h264` is based on upstream `v2.4-branch` and [intel-driver-g45-h264-2.4.1.tar.gz](https://bitbucket.org/alium/g45-h264/downloads/intel-driver-g45-h264-2.4.1.tar.gz)
- `master-g45-h264` is `v2.4-branch-g45-h264` rebased onto upstream `master`. It has few extra fixes.
- `master-experimental` is based on [intel-driver-g45-h264-2.4.1-experimental.tar.gz](https://bitbucket.org/alium/g45-h264/downloads/intel-driver-g45-h264-2.4.1-experimental.tar.gz). It has just one line of changed code that blindly enables h264.

Look into draft _pull requests_ in this project to see what are the modifications.

You can also download the original code [from Bitbucket](https://bitbucket.org/alium/g45-h264/downloads)

```sh
wget https://bitbucket.org/alium/g45-h264/downloads/intel-driver-g45-h264-2.4.1.tar.gz
tar -xvzf intel-driver-g45-h264-2.4.1.tar.gz
mv intel-vaapi-driver intel-vaapi-driver-src
```

## Get dependencies

For example in Ubuntu 24.04 or 22.04

```sh
apt install build-essential pkgconf autoconf libtool -y
apt install libdrm-dev libva-dev libx11-dev -y
```

## Build

```sh
cd intel-vaapi-driver-src
./autogen.sh
./configure LIBVA_DRIVERS_PATH="$(pwd)/../intel-vaapi-driver"
make install -j $(nproc)
```

The outcome can be found at `intel-vaapi-driver/i965_drv_video.so`.

## Use

For example on Ubuntu the `i965-va-driver` package installs a single file `/usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so`. Replacing the file with a modified version offers h264 acceleration.

Make copies first

```sh
sudo cp /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so-ubuntu
sudo cp intel-vaapi-driver/i965_drv_video.so /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so-g45-h264-alium
```

You can switch between the upstream and modified driver with

```sh
sudo cp /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so-ubuntu /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
```

and

```sh
sudo cp /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so-g45-h264-alium /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
```

## Check if it works

```sh
sudo vainfo
```

Before modification

```
libva info: VA-API version 1.22.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_1_20
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.22 (libva 2.22.0)
vainfo: Driver version: Intel i965 driver for Intel(R) GM45 Express Chipset - 2.4.1
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            :	VAEntrypointVLD
      VAProfileMPEG2Main              :	VAEntrypointVLD
```

After modification

```
libva info: VA-API version 1.22.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_1_22
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.22 (libva 2.22.0)
vainfo: Driver version: Intel i965 driver for Intel(R) GM45 Express Chipset - 2.4.1
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            :	VAEntrypointVLD
      VAProfileMPEG2Main              :	VAEntrypointVLD
      VAProfileH264ConstrainedBaseline:	VAEntrypointVLD
      VAProfileH264Main               :	VAEntrypointVLD
      VAProfileH264High               :	VAEntrypointVLD
```

```sh
mpv --hwdec=vaapi big_buck_bunny_720p_h264.mov
```

Unfortunatelly the modification doesn't work for me.
- `v2.4-branch-g45-h264` - refuses to use GPU in Ubuntu 24.04, most likely due to lack of fix that was added to `master` later.
- `master-g45-h264` - GPU acts strangely for few seconds, then GPU hangs, session crashes back to login screen.
- `master-experimental` - refuses to use GPU.

```sh
vlc big_buck_bunny_720p_h264.mov
```

## Test system

```sh
inxi -G
```

```
Graphics:
  Device-1: Intel Mobile 4 Series Integrated Graphics driver: i915 v: kernel
  Display: wayland server: X.Org v: 24.1.2 with: Xwayland v: 24.1.2
    compositor: gnome-shell v: 47.0 driver: dri: crocus gpu: i915
    resolution: 1680x1050~60Hz
  API: EGL v: 1.5 drivers: crocus,swrast
    platforms: gbm,wayland,x11,surfaceless,device
  API: OpenGL v: 4.5 compat-v: 2.1 vendor: intel mesa v: 24.2.3-1ubuntu1
    renderer: Mesa Mobile Intel GM45 Express (CTG)
```

```sh
sudo intel_gpu_top
```

```
Failed to initialize PMU! (No such device)
```

## Acceleration in browsers

Chromium needs `--enable-features=VaapiVideoDecodeLinuxGL` switch, for example

```sh
flatpak run com.google.Chrome --enable-features=VaapiVideoDecodeLinuxGL
```

Firefox might need configuration in `about:config`.

For YouTube both browsers need _h264ify_ or _enhanced h264ify_ extension to prevent use of AV1 or VP9 codecs. Blocking 60fps in favor of 30fps using these extensions might also help.

## Packages

Packages without modification are available in:
- [Debian](https://packages.debian.org/search?keywords=i965-va-driver) called `i965-va-driver`.
- [Ubuntu](https://launchpad.net/ubuntu/+source/intel-vaapi-driver) called `i965-va-driver`.
- [Arch Linux](https://archlinux.org/packages/extra/x86_64/libva-intel-driver) called `libva-intel-driver`.

Patched driver (h264 G45) is available in:
- [Arch Linux AUR](https://aur.archlinux.org/packages/libva-intel-driver-g45-h264) called `libva-intel-driver-g45-h264`.

## External documentation

- https://github.com/intel/intel-vaapi-driver/issues/544
- https://fedoraproject.org/wiki/Firefox_Hardware_acceleration
- https://wiki.archlinux.org/title/Hardware_video_acceleration
- https://wiki.archlinux.org/title/Intel_graphics#Hardware_accelerated_H.264_decoding_on_GMA_4500

## Development

To avoid unnecessary changes to line endings

```sh
git config core.whitespace cr-at-eol
```
