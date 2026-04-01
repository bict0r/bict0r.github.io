+++
title = 'Fixing fingerprint login on CachyOS with KDE Plasma'
date = 2026-03-31T20:00:00-05:00
draft = false
+++

This is my step-by-step write-up for getting the Synaptics Prometheus reader working on my ThinkPad X1 Carbon Gen 9 with CachyOS and KDE Plasma.

## What I was trying to do

Get fingerprint login and lock-screen unlock working.

## What I checked first

- Verified the sensor with `lsusb`
- Confirmed `fprintd` and `libfprint` were installed
- Enrolled my right index finger

## What fixed it

- Added `auth sufficient pam_fprintd.so` to:
  - `/etc/pam.d/system-local-login`
  - `/etc/pam.d/kde`
  - `/etc/pam.d/sddm`

## Notes

I also made sudo fingerprint optional with toggle scripts.
