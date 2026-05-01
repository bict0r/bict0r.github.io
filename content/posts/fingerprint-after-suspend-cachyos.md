+++
title = 'Fixing ThinkPad Fingerprint Auth After Suspend on CachyOS'
date = 2026-05-01T00:00:00-05:00
draft = false
description = "fprintd breaks after the ThinkPad wakes from suspend — here's how to auto-restart it with a system-sleep hook so fingerprint login works every time."
tags = ["linux", "cachyos", "thinkpad", "sysadmin", "fprintd"]
+++

After getting fingerprint login working on CachyOS, I ran into a second problem: every time I closed the lid and woke the laptop, fingerprint auth would fail instantly — no prompt, no chance to scan, just an immediate failure that also blocked the password field from appearing.

This is a follow-up to [ThinkPad X1 Fingerprint Setup on CachyOS with KDE Plasma](/posts/fixing-fingerprint-on-cachyos/).

---

## The problem

When the system suspends and resumes, `fprintd` ends up in a broken state. It doesn't crash — it's still running — but the fingerprint reader is unresponsive and PAM gets an instant failure from it. Because `pam_fprintd.so` is listed as `sufficient`, that failure can race ahead and lock out the password prompt too.

The fix is to restart `fprintd` every time the system wakes from suspend.

---

## The fix — system-sleep hook

systemd runs scripts in `/lib/systemd/system-sleep/` automatically on suspend and resume. Create a hook there:

```bash
sudo nano /lib/systemd/system-sleep/fprintd-reset
```

Paste:

```sh
#!/bin/sh
case "$1" in
  post)
    systemctl restart fprintd.service
    ;;
esac
```

Make it executable:

```bash
sudo chmod +x /lib/systemd/system-sleep/fprintd-reset
```

No reboot required. The hook fires on the next wake.

---

## How it works

systemd calls every script in `/lib/systemd/system-sleep/` with two arguments:

- `$1` — either `pre` (about to suspend) or `post` (just woke up)
- `$2` — the suspend type (`suspend`, `hibernate`, `hybrid-sleep`)

The script only acts on `post`, which runs after the system is fully back up. `systemctl restart fprintd.service` forces a clean restart of the fingerprint daemon, clearing any stuck sensor state from before suspend.

---

## Verify it's working

After waking from suspend:

```bash
journalctl -u fprintd --since "1 min ago"
```

You should see `fprintd` restarting. Fingerprint login on the lock screen should work normally again.

---

## Fallback

If the fingerprint reader is still unresponsive after a wake, press **Enter** or **Esc** to dismiss the fingerprint prompt and get the password field. The PAM config from the setup guide includes password as a fallback — you're never locked out.

---

## Quick reference

| File | Purpose |
|------|---------|
| `/lib/systemd/system-sleep/fprintd-reset` | Restarts `fprintd` on every wake from suspend |

```bash
# Check hook ran after waking
journalctl -u fprintd --since "1 min ago"

# Manual restart if needed
sudo systemctl restart fprintd.service
```
