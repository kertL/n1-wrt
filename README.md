# Phicomm N1 OpenWrt Actions Build

This repository uses GitHub Actions to build ImmortalWrt 24.10 for the generic `armsr/armv8` target, then optionally packages a Phicomm N1 `.img` with Flippy packaging scripts.

## What you get

- `openwrt-n1-rootfs`: a compressed root filesystem, not a directly bootable image.
- `openwrt-n1-image`: a complete Phicomm N1 image, only when you enable the packaging step.

The build uses `armsr/armv8`, not the older `armvirt/64` target name.

## Create the GitHub repository

1. Create a new **public** repository on GitHub. Public repositories do not consume your private Actions minutes.
2. Push this directory:

   ```sh
   git init
   git add .
   git commit -m "Build N1 OpenWrt"
   git branch -M main
   git remote add origin git@github.com:<your-user>/<your-repo>.git
   git push -u origin main
   ```

3. Open **Actions** and enable workflows if GitHub asks.
4. Select **Build N1 OpenWrt**.
5. Choose **Run workflow**:
   - Keep `v24.10.6` for the tested stable tag, or use `openwrt-24.10` to follow the branch.
   - Leave the N1 packaging checkbox off for the first run.
   - After the rootfs build succeeds, run again with the checkbox enabled.

The first build usually takes 1 to 3 hours. If it fails, open the failed job and read the verbose rebuild log.

## Download the image

When the job finishes:

1. Open the successful workflow run.
2. Scroll to **Artifacts**.
3. Download `openwrt-n1-image` if you enabled packaging, or `openwrt-n1-rootfs` if you only want the root filesystem.

Write the `.img` to a USB drive with balenaEtcher, then boot the N1 from USB. Back up the original eMMC before installing to internal storage. A USB-only test boot is safer than writing eMMC immediately.

## Third-party packaging

The optional N1 packaging step uses `ophub/flippy-openwrt-actions` and the Flippy kernel packaging scripts. It is convenient, but it is still third-party code.

After one successful run, pin the action to a specific commit SHA instead of `@main`. Review the action and the packaging scripts before you build a long-term image.

If the packaged kernel does not include UVC support, a rootfs package alone will not make the camera work. Verify the loaded module after boot.

## Camera checks

After OpenWrt boots with the Logitech USB camera connected:

```sh
lsusb
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --list-formats-ext
dmesg | grep -Ei 'uvc|video|usb'
```

The image includes `mjpg-streamer` for a low-overhead live stream and `motion-noffmpeg` for JPEG-based motion detection. The workflow installs motion from the selected OpenWrt packages branch. A USB camera can normally be opened by only one process at a time.

Store recordings on a USB drive, external disk, or NAS rather than eMMC.

The included hotplug script disables USB autosuspend for Logitech devices to reduce dropped camera streams.
