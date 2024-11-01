# Fork of `intel-vaapi-driver` with h264 G45 support

## The issue

Some Gen-4 intel GPUs (GMA X4500HD, GMA 4500MHD, X4700MHD) support hardware acceleration of h264 video decoding, but the user-space driver from Intel doesn't offer it.

Related issue https://github.com/intel/intel-vaapi-driver/issues/544

## Upstream

The video acceleration user-space driver was maintained at:
- https://cgit.freedesktop.org/vaapi/intel-driver (until 2017.02.18)
- https://github.com/intel/intel-vaapi-driver previously https://github.com/01org/intel-vaapi-driver (2024.10.29)

The driver is no longer maintained by Intel with `intel-vaapi-driver` successor https://github.com/intel/media-driver supporting Gen-8 and up.

## Patches

Patches that enable h264 in G45 are available at https://bitbucket.org/alium/g45-h264/downloads.  
I am not aware who is the author of the code, it might be the user called `alium` or someone else.  
Please open the issue if you know the details where this code came from.

## This project

This project is a fork of https://github.com/intel/intel-vaapi-driver `v2.4-branch` and `master` branches with applied patches from https://bitbucket.org/alium/g45-h264/downloads.

I created this repository to preserve the patches in a transparent way. Tar-balls that are available from Bitbucket feel like a black-box without a real option for further contribution.

## Packaging

Packages are available in:
- [Debian](https://packages.debian.org/search?keywords=i965-va-driver) called `i965-va-driver`.
- [Ubuntu](https://launchpad.net/ubuntu/+source/intel-vaapi-driver) called `i965-va-driver`.
- [Arch Linux](https://archlinux.org/packages/extra/x86_64/libva-intel-driver) called `libva-intel-driver`.

Patched driver (h264 G45) is available in:
- [Arch Linux AUR](https://aur.archlinux.org/packages/libva-intel-driver-g45-h264) called `libva-intel-driver-g45-h264`.

## Development

To avoid unnecessary changes to line endings

```sh
git config core.whitespace cr-at-eol
```
