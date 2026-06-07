# Commands Run — F9212B Tablet Investigation Session
**Date:** 2026-06-07  
**Device:** alps F9212B / ac8227l / Android 10 / Kernel 3.18.79 armv7l

---

## 1. Kernel & Build Info

```bash
adb shell uname -r
adb shell uname -a
adb shell cat /proc/version
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk
```

---

## 2. Root & Module Status

```bash
# Module loading
adb shell cat /proc/sys/kernel/modules_disabled
adb shell lsmod
adb shell find / -name "*.ko" 2>/dev/null

# Tools present
adb shell which insmod
adb shell which modprobe
adb shell which rmmod
```

---

## 3. USB & Network

```bash
# USB
adb shell ls /dev/bus/usb/
adb shell cat /proc/bus/usb/devices 2>/dev/null
adb shell lsusb 2>/dev/null
adb shell cat /sys/class/power_supply/usb/type 2>/dev/null

# Network interfaces
adb shell ip link show
adb shell cat /proc/net/dev
adb shell ifconfig -a 2>/dev/null
```

---

## 4. Kernel Config

```bash
adb shell ls /proc/config.gz
adb shell gzip -d -c /proc/config.gz | grep -i "USB_NET\|WIRELESS\|CFG80211\|MAC80211\|MODULES"
adb shell ls /system/kernel/config.gz 2>/dev/null
```

---

## 5. SELinux & Lockdown

```bash
adb shell getenforce
adb shell cat /sys/kernel/security/lockdown 2>/dev/null
adb shell dmesg | grep -i "lockdown\|module\|denied" | head -30
```

---

## 6. USB Driver & Device Tree

```bash
adb shell ls /sys/class/net/
adb shell ls /sys/bus/usb/drivers/
adb shell ls /sys/bus/usb/devices/
adb shell cat /sys/bus/usb/drivers/*/uevent 2>/dev/null
```

---

## 7. Root Status Investigation

```bash
# Current privilege level
adb shell id
adb shell whoami
adb shell getprop service.adb.root
adb shell ps | grep adbd

# Try ADB root escalation
adb root
adb shell id

# SU binary locations
adb shell which su
adb shell ls -la /system/bin/su 2>/dev/null
adb shell ls -la /system/xbin/su 2>/dev/null
adb shell ls -la /sbin/su 2>/dev/null
adb shell find / -name "su" -type f 2>/dev/null

# SU type/version
adb shell su --version 2>/dev/null
adb shell su -v 2>/dev/null

# Magisk checks
adb shell ls /data/adb/magisk 2>/dev/null
adb shell ls /sbin/.magisk 2>/dev/null
adb shell getprop ro.boot.magisk_version 2>/dev/null
adb shell which magisk 2>/dev/null

# SuperSU checks
adb shell ls /system/app/SuperSU 2>/dev/null
adb shell ls /system/xbin/daemonsu 2>/dev/null

# KingRoot checks
adb shell ls /system/bin/kc 2>/dev/null
adb shell ls /system/app/KingRoot 2>/dev/null

# SELinux context of su
adb shell ls -laZ /system/bin/su 2>/dev/null
adb shell ls -laZ /system/xbin/su 2>/dev/null

# Partition mount status
adb shell mount | grep system
adb shell mount | grep vendor

# Build properties
adb shell getprop ro.debuggable
adb shell getprop ro.secure
adb shell getprop ro.build.tags
adb shell getprop ro.boot.verifiedbootstate 2>/dev/null
adb shell getprop ro.boot.flash.locked 2>/dev/null
adb shell getprop ro.boot.veritymode 2>/dev/null
adb shell getprop ro.boot.warranty_bit 2>/dev/null

# Process capabilities
adb shell cat /proc/self/status | grep -i "cap\|uid\|gid"

# Init scripts
adb shell ls /system/etc/init.d/ 2>/dev/null
adb shell ls /etc/init.d/ 2>/dev/null

# Functional root test
adb shell su -c id
adb shell "su 0 id"
```

---

## 8. Aptoide Root Detection — Full Signal Check

```bash
# SU binary paths (full scan)
adb shell ls -la /system/bin/su 2>/dev/null
adb shell ls -la /system/xbin/su 2>/dev/null
adb shell ls -la /system/sbin/su 2>/dev/null
adb shell ls -la /sbin/su 2>/dev/null
adb shell ls -la /su/bin/su 2>/dev/null
adb shell ls -la /data/local/su 2>/dev/null
adb shell ls -la /data/local/bin/su 2>/dev/null
adb shell ls -la /data/local/xbin/su 2>/dev/null
adb shell ls -la /system/sd/xbin/su 2>/dev/null
adb shell ls -la /system/bin/failsafe/su 2>/dev/null

# Magisk full check
adb shell ls /data/adb/magisk 2>/dev/null
adb shell ls /data/adb/modules 2>/dev/null
adb shell ls /sbin/.magisk 2>/dev/null
adb shell ls /debug_ramdisk/.magisk 2>/dev/null
adb shell getprop ro.boot.magisk_version 2>/dev/null
adb shell getprop persist.sys.safetynet.hide 2>/dev/null

# SuperSU full check
adb shell ls /system/xbin/daemonsu 2>/dev/null
adb shell ls /system/etc/.installed_su_daemon 2>/dev/null
adb shell ls /system/app/SuperSU 2>/dev/null
adb shell ls /system/app/SuperSU.apk 2>/dev/null

# Root package scan
adb shell pm list packages | grep -i "supersu\|magisk\|kingroot\|kingoroot\|towelroot\|framaroot\|superuser\|chainfire\|stericson"

# Build props
adb shell getprop ro.debuggable
adb shell getprop ro.secure
adb shell getprop ro.build.tags
adb shell getprop ro.build.type
adb shell getprop ro.build.selinux
adb shell cat /system/build.prop | grep "tags\|type\|secure\|debug"
adb shell getprop | grep -E "ro.build.tags|ro.build.type|ro.secure|ro.debuggable"

# Filesystem write check
adb shell ls -la /system 2>/dev/null | head -5
adb shell ls -la /data 2>/dev/null | head -5
adb shell ls -la /proc/net/tcp 2>/dev/null

# Dangerous props
adb shell getprop | grep -i "root\|debug\|unlock\|flash\|verify\|boot" | head -30

# Busybox
adb shell which busybox 2>/dev/null
adb shell ls /system/bin/busybox 2>/dev/null
adb shell ls /system/xbin/busybox 2>/dev/null
adb shell ls /data/local/busybox 2>/dev/null

# Running root processes
adb shell ps | grep -i "root\|daemon\|su\|magisk" | head -20
adb shell ps -A 2>/dev/null | grep -i "supersu\|magiskd\|daemonsu" | head -10

# Capabilities
adb shell cat /proc/self/status | grep -i "uid\|gid\|cap"

# SafetyNet / Play Integrity signals
adb shell getprop ro.boot.verifiedbootstate 2>/dev/null
adb shell getprop ro.boot.flash.locked 2>/dev/null
adb shell getprop ro.boot.veritymode 2>/dev/null
adb shell getprop ro.hardware

# SELinux
adb shell getenforce
adb shell getprop ro.boot.selinux

# Device identity
adb shell getprop ro.product.model
adb shell getprop ro.product.brand
```

---

## 9. Git / GitHub — Report File Management

```bash
# Push to wrong repo (ZScreen-Dash-USB-Tesla-Style) — MISTAKE
git add Downloads/F9212B_Root_Status_Report.docx
git commit -m "Add F9212B root status report"
git push

# Remove from wrong repo
git rm Downloads/F9212B_Root_Status_Report.docx
git commit -m "Remove F9212B root status report"
git push

# Verify file location
ls -la /home/jonathan/Downloads/F9212B_Root_Status_Report.docx

# Create new dedicated repo and push
cd ~
mkdir device-analysis
cd device-analysis
git init
git remote add origin https://github.com/electronics101clt/device-analysis.git
cp /home/jonathan/Downloads/F9212B_Root_Status_Report.docx .
git add F9212B_Root_Status_Report.docx
git commit -m "Add F9212B root status report"
git branch -M main
git push -u origin main
```

---

## Key Findings Summary

| Check | Result |
|---|---|
| `modules_disabled` | 0 — module loading allowed |
| `su` binary | `/system/xbin/su` — setuid root |
| `su 0 id` | `uid=0(root)` — functional |
| `ro.debuggable` | 1 |
| `ro.build.tags` | test-keys |
| `ro.boot.selinux` | permissive |
| `ro.boot.veritymode` | disabled |
| Magisk | NOT present |
| SuperSU | NOT present |
| Root manager | NONE |
| ADB root (`adb root`) | FAILED — adbd runs as shell |
| Effective root method | `adb shell su 0 <command>` |
