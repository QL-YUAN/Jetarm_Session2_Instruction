# File Transfer Between Laptop and Jetson Orin Using SCP

This guide explains how to transfer files between your laptop and a **Jetson Orin** device using the `scp` (Secure Copy) command.

---

## Prerequisites

* Both devices must be connected to the same network.
* SSH must be enabled on the Jetson Orin.
* You must know:

  * Jetson IP address (example: `192.168.149.1`)
  * Username on Jetson (e.g., `hiwonder` or `ubuntu`)

---

## 1. Copy File **From Laptop to Jetson Orin**

Use the following command on your laptop:

```bash
scp github.pdf hiwonder@192.168.149.1:~/Desktop/
```

### Explanation:

* `github.pdf` → File on your laptop
* `hiwonder` → Jetson username
* `192.168.149.1` → Jetson IP address
* `~/Desktop/` → Destination folder on Jetson

If your Jetson username is `ubuntu`, use:

```bash
scp github.pdf ubuntu@192.168.149.1:~/Desktop/
```

---

## 2. Copy File **From Jetson Orin to Laptop**

Run this command on your laptop:

```bash
scp hiwonder@192.168.149.1:~/Desktop/*.py .
```

### Explanation:

* `hiwonder@192.168.149.1` → Jetson login
* `~/Desktop/*.py` → All Python files on Jetson Desktop
* `.` → Current folder on your laptop

If your username is `ubuntu`:

```bash
scp ubuntu@192.168.149.1:~/Desktop/*.py .
```

---

## 3. Copy Entire Folder (Optional)

To copy a folder recursively:

### Laptop → Jetson

```bash
scp -r my_folder hiwonder@192.168.149.1:~/Desktop/
```

### Jetson → Laptop

```bash
scp -r hiwonder@192.168.149.1:~/Desktop/my_folder .
```

---

## 4. Common Issues

### Permission Denied

* Make sure SSH is enabled:

  ```bash
  sudo systemctl status ssh
  ```
* Verify username and password.

### Connection Refused

* Check IP address using:

  ```bash
  ifconfig
  ```

  or

  ```bash
  ip addr
  ```

---

## 5. Quick Summary

| Direction       | Command Example                   |
| --------------- | --------------------------------- |
| Laptop → Jetson | `scp file.pdf user@IP:~/Desktop/` |
| Jetson → Laptop | `scp user@IP:~/Desktop/file.py .` |

---
