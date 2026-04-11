+++
title = 'ThinkPad X1 Fingerprint Setup on CachyOS with KDE Plasma'
date = 2026-03-31T20:00:00-05:00
draft = false
description = "A complete guide to getting the Synaptics Prometheus fingerprint reader working on CachyOS with KDE Plasma — login, lock screen, sudo, and toggle scripts included."
tags = ["linux", "cachyos", "kde", "sysadmin", "thinkpad"]
+++

## System Info

| | |
|---|---|
| **Laptop** | ThinkPad X1 |
| **Sensor** | Synaptics Prometheus (`06cb:00fc`) |
| **OS** | CachyOS (Arch-based) |
| **Desktop** | KDE Plasma |

---

## Part 1 — Initial Setup

### 1. Verify the fingerprint reader

```bash
lsusb
```

Look for:

```
06cb:00fc Synaptics, Inc. Prometheus Fingerprint Reader
```

### 2. Install required packages

```bash
sudo pacman -S fprintd libfprint
```

### 3. Service behavior

```bash
systemctl status fprintd
```

> **Note:** `inactive (dead)` is normal — `fprintd` is D-Bus activated. Do **not** enable it with `systemctl`.

### 4. Enroll your fingerprint

```bash
fprintd-enroll
```

### 5. Verify enrollment

```bash
fprintd-list $USER
```

Expected output:

```
right-index-finger
```

### 6. Enable fingerprint for system login

```bash
sudo nano /etc/pam.d/system-local-login
```

Add at the top:

```
auth sufficient pam_fprintd.so
```

### 7. Enable fingerprint for KDE lock screen

```bash
sudo nano /etc/pam.d/kde
```

Add:

```
auth sufficient pam_fprintd.so
```

### 8. Enable fingerprint for SDDM login screen

```bash
sudo nano /etc/pam.d/sddm
```

Add:

```
auth sufficient pam_fprintd.so
```

### 9. Test the lock screen

```bash
loginctl lock-session
```

Then touch the fingerprint reader.

> **Note:** KDE may not show a fingerprint prompt — just touch the reader anyway. Password fallback always works.

---

## Part 2 — Optional Enhancements

### Enable fingerprint for sudo

```bash
sudo nano /etc/pam.d/sudo
```

Add:

```
auth sufficient pam_fprintd.so
```

For faster password fallback:

```
auth sufficient pam_fprintd.so max-tries=1 timeout=3
```

> **Limitation:** Fingerprint and password are not simultaneous. Fingerprint runs first; password is only available after timeout.

---

### Toggle Fingerprint for sudo

Rather than permanently editing `/etc/pam.d/sudo`, these scripts let you flip fingerprint auth on and off as needed.

**Backup your sudo PAM config first:**

```bash
sudo cp /etc/pam.d/sudo /etc/pam.d/sudo.bak
```

#### Enable script (`sudo-fp-on`)

```bash
sudo nano /usr/local/bin/sudo-fp-on
```

Paste:

```bash
#!/bin/bash
if [ "$EUID" -ne 0 ]; then
  exec sudo "$0" "$@"
fi
set -e
FILE=/etc/pam.d/sudo
if grep -q '^auth[[:space:]]+sufficient[[:space:]]+pam_fprintd.so' "$FILE"; then
  echo "Fingerprint for sudo is already ON."
  exit 0
fi
cp "$FILE" "${FILE}.togglebak"
awk '
BEGIN { inserted=0 }
{
  if (!inserted && $1 == "auth" && $2 == "include" && $3 == "system-auth") {
    print "auth sufficient pam_fprintd.so max-tries=1 timeout=3"
    inserted=1
  }
  print
}
END {
  if (!inserted) {
    print "auth sufficient pam_fprintd.so max-tries=1 timeout=3"
  }
}
' "$FILE" > /tmp/sudo.pam.new
install -m 644 /tmp/sudo.pam.new "$FILE"
rm -f /tmp/sudo.pam.new
echo "Fingerprint for sudo is now ON."
```

```bash
sudo chmod +x /usr/local/bin/sudo-fp-on
```

#### Disable script (`sudo-fp-off`)

```bash
sudo nano /usr/local/bin/sudo-fp-off
```

Paste:

```bash
#!/bin/bash
if [ "$EUID" -ne 0 ]; then
  exec sudo "$0" "$@"
fi
set -e
FILE=/etc/pam.d/sudo
cp "$FILE" "${FILE}.togglebak"
grep -v '^auth[[:space:]]+sufficient[[:space:]]+pam_fprintd.so' "$FILE" > /tmp/sudo.pam.new
install -m 644 /tmp/sudo.pam.new "$FILE"
rm -f /tmp/sudo.pam.new
echo "Fingerprint for sudo is now OFF."
```

```bash
sudo chmod +x /usr/local/bin/sudo-fp-off
```

#### Usage

```bash
sudo-fp-on   # enable fingerprint for sudo
sudo-fp-off  # disable fingerprint for sudo
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Enroll fingerprint | `fprintd-enroll` |
| List enrolled fingers | `fprintd-list $USER` |
| Test lock screen | `loginctl lock-session` |
| Enable sudo fingerprint | `sudo-fp-on` |
| Disable sudo fingerprint | `sudo-fp-off` |

---

## Recommended Setup

| Feature | Recommendation |
|---------|---------------|
| Login | Fingerprint enabled |
| Lock screen | Fingerprint enabled |
| sudo | Password by default, toggle when needed |

The "Place your right index finger on the fingerprint reader" prompt comes from `pam_fprintd` / `fprintd` and is not easily customizable.
