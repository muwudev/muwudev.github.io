#dev #linux 
After performing a system for Ubuntu 24.04 update, the system fails to boot properly and displays the following error message:

```
Oh no! Something is wrong. A problem has occurred and the system can't recover. Please contact a system administrator.
```

Attempts to access a terminal via `Ctrl + Alt + F3` do not work. The `Esc` or `Shift` key during boot briefly shows something but no longer works.

Important files are present on the system, so a fresh installation is not preferred.

---

## Solution Walkthrough

### Step 1: Boot into a Live Environment
Since the system cannot boot properly, use a **Live USB** to access the files and troubleshoot the issue.

1. Create a bootable USB with a Linux distro using `Rufus` (Windows) or `dd` (Linux/macOS). Or use the one you used initially to install Ubuntu on to your system if you still have it.
2. Insert the USB into the pc and boot from it.
3. Select the option to "Try Ubuntu" (or your respective distro's live mode).

### Step 2: Mount the Root Filesystem
Once inside the live session:

4. Open a terminal.
5. Identify the root partition using:
   ```bash
   lsblk
   ```
6. Mount the partition (assuming it's `/dev/sda2`, adjust as needed):
   ```bash
   sudo mount /dev/sda2 /mnt
   ```
7. Bind system directories:
   ```bash
   for i in /dev /proc /sys /run; do sudo mount --bind "$i" /mnt$i; done
   ```

### Step 3: Chroot into the System

8. Enter the installed system using:
   ```bash
   sudo chroot /mnt
   ```
9. Update package sources:
   ```bash
   apt update
   ```

### Step 4: Fix Broken Packages

10. Try auto-fixing broken dependencies:
   ```bash
   apt --fix-broken install
   ```
11. If the display manager is failing, reinstall it:
   ```bash
   apt reinstall gdm3
   ```
   (For KDE, use `sddm`, and for XFCE, use `lightdm`.)

### Step 5: Reinstall Display Drivers
If a driver issue caused the failure:

12. Check for installed drivers:
   ```bash
   ubuntu-drivers devices
   ```
13. Install recommended drivers:
   ```bash
   ubuntu-drivers install
   ```

### Step 6: Update Initramfs and Grub

14. Regenerate the initramfs:
   ```bash
   update-initramfs -u -k all
   ```
15. Update GRUB:
   ```bash
   update-grub
   ```

### Step 7: Reboot the System
Exit the chroot environment and unmount everything:
```bash
exit
for i in /dev /proc /sys /run; do sudo umount /mnt$i; done
sudo umount /mnt
```
Then remove the live USB and reboot:
```bash
sudo reboot
```

If the issue persists, check logs using:
```bash
journalctl -xb
```
or try an alternative session by running:
```bash
startx
```

If none of the above work, consider backing up files and reinstalling the system.