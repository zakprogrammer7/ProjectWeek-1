# Linux System Administration Lab Summary

This document summarizes the user management, shared directory permissions, and Logical Volume Manager (LVM) storage configuration set up on `debian`.

---

![alt text](<Screenshot 2026-11-21 180813.png>)

![alt text](<Screenshot 2026-11-21 174327.png>)

## 1. Users & Groups

### Users (`/etc/passwd`)
| Username | UID | GID | Home Directory | Shell |
| :--- | :--- | :--- | :--- | :--- |
| `zaki` | `1000` | `1000` | `/home/zaki` (`drwx------`) | `/bin/bash` |
| `user1` | `1001` | `1001` | `/home/user1` (`drwx------`) | `/bin/bash` |
| `user2` | `1002` | `1002` | `/home/user2` (`drwx------`) | `/bin/bash` |
| `admin` | `1003` | `1003` | `/home/admin` (`drwx------`) | `/bin/bash` |

### Groups (`/etc/group`)
| Group Name | GID | Members |
| :--- | :--- | :--- |
| `admins` | `1004` | `admin`, `zaki` |
| `Members` | `1005` | `user1`, `user2`, `admin`, `zaki` |

---

## 2. Shared Directory & Special Permissions (`/SharedFiles`)

| Path | Owner:Group | Permissions | Special Bits & Behavior |
| :--- | :--- | :--- | :--- |
| `/SharedFiles` | `admin:Members` | `drwxr-xr-t` (`1755`) | **Sticky Bit (`+t`)**: Prevents non-owners from deleting/renaming top-level items. |
| `/SharedFiles/AdminsOnly` | `admin:admins` | `drwxrws---` (`2770`) | **SGID (`g+s`)**: Restricted to `admins` group; new files automatically inherit the `admins` group. |
| `/SharedFiles/All` | `admin:Members` | `drwxrws--T` (`3770`) | **SGID + Sticky Bit**: Accessible by `Members`; new files inherit `Members` group, and users can only delete their own files (except folder owner `admin` and `root`). |

### Sample Files in `/SharedFiles/All`
* `testfile` (`50 MiB`, owned by `admin:Members`, `-rw-rw-r--`)
* `testfile2` (`0 B`, owned by `user1:Members`, `-rw-rw-r--`)

---

## 3. LVM Storage & Mount Configuration

### Physical Volumes & Volume Group (`vg1`)
* **Volume Group:** `vg1` (`19.99 GiB` total, `5118` PEs @ `4.00 MiB` PE size, `0` free)
* **Physical Volumes:**
  * `/dev/sdb` (`10 GiB` / `2559` PEs)
  * `/dev/sdc` (`10 GiB` / `2559` PEs)

### Logical Volumes & Persistent Mounts (`/etc/fstab`)
All logical volumes are formatted with `ext4` and mounted persistently at boot via UUID:

| Logical Volume | Device Path | Size | Mount Point | Filesystem |
| :--- | :--- | :--- | :--- | :--- |
| `lv1` | `/dev/vg1/lv1` | `2.00 GiB` | `/mnt/data1` | `ext4` |
| `lv2` | `/dev/vg1/lv2` | `2.00 GiB` | `/mnt/data2` | `ext4` |
| `lv3` | `/dev/vg1/lv3` | `2.00 GiB` | `/mnt/data3` | `ext4` |
| `lv4` | `/dev/vg1/lv4` | `2.00 GiB` | `/mnt/data4` | `ext4` |
| `lv5` | `/dev/vg1/lv5` | `2.00 GiB` | `/mnt/data5` | `ext4` |
| `lv6` | `/dev/vg1/lv6` | `9.99 GiB` | `/mnt/data6` | `ext4` *(spanned across `sdb` & `sdc`)* |