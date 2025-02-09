#dev #linux 

after performing a system for Ubuntu 24.04 update, system fails to boot properly and displays the error message:

```
Oh no! Something is wrong. A problem has occurred and the system can't recover. Please contact a system administrator.
```

attempts to access a terminal via `ctrl + alt + f3` do not work. the `esc` or `shift` key during boot briefly shows something but no longer works. if you have important files on the system the following guide is for you

---

lets walk you through this :3

`step 1: boot into a live environment`

since the system cannot boot properly, use a **live usb** to access the files and troubleshoot the issue

1. create a bootable usb with a linux distro using `rufus` (windows) or `dd` (linux/macos). or use the one you used initially to install ubuntu on to your system if you still have it
2. insert the usb into the pc and boot from it
3. select the option to "try ubuntu" (or your respective distro's live mode)

`step 2: mount the root filesystem`

once inside the live session:

4. open a terminal
5. identify the root partition using:
    
    ```bash
    lsblk
    ```
    
6. mount the partition (assuming it's `/dev/sda2`, adjust as needed):
    
    ```bash
    sudo mount /dev/sda2 /mnt
    ```
    
7. bind system directories:
    
    ```bash
    for i in /dev /proc /sys /run; do sudo mount --bind "$i" /mnt$i; done
    ```
    

`step 3: chroot into the system`

8. enter the installed system using:
    
    ```bash
    sudo chroot /mnt
    ```
    
9. update package sources:
    
    ```bash
    apt update
    ```
    

`step 4: fix broken packages`

10. try auto-fixing broken dependencies:

```bash
apt --fix-broken install
```

11. if the display manager is failing, reinstall it:

```bash
apt reinstall gdm3
```

(for kde, use `sddm`, and for xfce, use `lightdm`)


`step 5: reinstall display drivers`

if a driver issue caused the failure:

12. check for installed drivers:

```bash
ubuntu-drivers devices
```

13. install recommended drivers:

```bash
ubuntu-drivers install
```

`step 6: update initramfs and grub`

14. regenerate the initramfs:

```bash
update-initramfs -u -k all
```

15. update grub:

```bash
update-grub
```


` step 7: reboot the system`

16. exit the chroot environment and unmount everything:

```bash
exit
for i in /dev /proc /sys /run; do sudo umount /mnt$i; done
sudo umount /mnt
```

17. then remove the live usb and reboot:

```bash
sudo reboot
```

`extras`
if the issue persists, check logs using:

```bash
journalctl -xb
```

or try an alternative session by running:

```bash
startx
```

`my situation`:
my pc was not able to translate dns, couldn't retrieve packages from official servers. pc was not able to update properly so got the error while doing a system update. terminal would not pop up even when pressing `ctrl + alt + f3`. 

after chrooting into my system with the live boot, i was able to configure the dns settings. packages were then retrieved to fix missing mesa and nvidia packages

if your case is similar, check [this](https://www.reddit.com/r/Ubuntu/comments/1cd86l4/no_connection_after_upgrading_to_2404_lts/) out

if none of the above work, consider:
- contacting their official support
- joining their official discord and asking for help
- asking for help on [r/Ubuntu](https://www.reddit.com/r/Ubuntu/)
- backing up files and reinstalling the system
- `crying cus i wasted your time :( </3`

