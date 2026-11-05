# Lab 10: File Systems

## Safe test setup

Use a disposable image file and a loop device. Confirm that `$HOME/lab10.img` does not already contain data you need; `truncate` will create or resize it. Run setup and lab commands in the same Bash session so the `LOOP` and `DEV` variables remain available.

```bash
truncate -s 300M "$HOME/lab10.img"
LOOP=$(sudo losetup --find --show --partscan "$HOME/lab10.img")
sudo parted --script "$LOOP" mklabel msdos mkpart primary ext4 1MiB 250MiB
sudo partprobe "$LOOP"
DEV="${LOOP}p1"
echo "$DEV"
```
The device should look like `/dev/loopNp1`. **Creating a file system erases data on the target device. Never run `mkfs` on a system or unidentified partition.**

**Q1. Display block devices and identify the test partition.**
```bash
lsblk
lsblk -f
```
Identify the partition created from the disposable image; do not select a system partition such as `/`, `/boot`, or an unidentified device.

**Q2. Display the test partition's file-system type, UUID, and mount information.**
```bash
lsblk -f "$DEV"
sudo blkid "$DEV"
findmnt -S "$DEV"
```
Before formatting, the file-system type and UUID may be empty. `findmnt` shows a result only when the device is mounted.

**Q3. Create an ext4 file system on the test partition and verify it.**
```bash
sudo mkfs.ext4 "$DEV"
lsblk -f "$DEV"
sudo blkid "$DEV"
sudo file -s "$DEV"
```

**Q4. Display detailed ext4 information and the UUID.**
```bash
sudo dumpe2fs -h "$DEV" | head -20
sudo tune2fs -l "$DEV" | grep -i -E 'UUID|name|block count'
sudo blkid -s UUID -o value "$DEV"
```

**Q5. Create the `/mnt/labfs` mount point.**
```bash
sudo mkdir -p /mnt/labfs
ls -ld /mnt/labfs
```

**Q6. Mount the test file system manually and verify the mount.**
```bash
sudo mount "$DEV" /mnt/labfs
findmnt /mnt/labfs
```

**Q7. Display total, used, and available space with `df`.**
```bash
df -h /mnt/labfs
```
An empty ext4 file system can show some used space because it contains metadata and a `lost+found` directory.

**Q8. Create a test file and verify that it is on the mounted file system.**
```bash
echo "hello labfs" | sudo tee /mnt/labfs/test.txt
ls -l /mnt/labfs
df -h /mnt/labfs/test.txt
findmnt -T /mnt/labfs/test.txt
```

**Q9. Unmount the file system and verify that it is no longer mounted.**
```bash
sudo umount /mnt/labfs
findmnt /mnt/labfs
mountpoint /mnt/labfs
ls -la /mnt/labfs
```
The mount point should be empty after unmounting. The file remains on the test partition.

**Q10. Display `/etc/fstab` and explain the fields for a persistent mount.**
```bash
cat /etc/fstab
```
Each non-comment entry has six fields:
1. Device, commonly `UUID=...`, `LABEL=...`, or `/dev/...`.
2. Mount point, such as `/mnt/labfs`.
3. File-system type, such as `ext4` or `xfs`.
4. Mount options, such as `defaults` or `nofail`.
5. Dump setting; usually `0`.
6. File-system check order; commonly `0` to disable, `1` for root, and `2` for other file systems.

**Q11. Back up `/etc/fstab` and add a test entry using the partition UUID.**
```bash
FSTAB_BACKUP=$(sudo mktemp /etc/fstab.lab10.XXXXXX)
sudo cp /etc/fstab "$FSTAB_BACKUP"
UUID=$(sudo blkid -s UUID -o value "$DEV")
printf 'UUID=%s /mnt/labfs ext4 defaults,nofail 0 2\n' "$UUID" |
  sudo tee -a /etc/fstab
tail -n 2 /etc/fstab
```
`mktemp` creates a unique backup path. Keep this Bash session open so `FSTAB_BACKUP` remains available for restoration. Do not add the same mount entry more than once.

**Q12. Validate the fstab entry and test it without rebooting.**
```bash
sudo findmnt --verify
sudo systemctl daemon-reload
sudo mount -a
findmnt /mnt/labfs
```
Resolve any validation error before rebooting.

**Q13. Verify that the fstab entry can remount the file system.**
```bash
sudo umount /mnt/labfs
sudo mount -a
findmnt /mnt/labfs
```
A loop device does not automatically reattach after a reboot, so this test simulates the fstab mount without rebooting.

**Q14. Verify the mount with `findmnt` and `df`.**
```bash
findmnt /mnt/labfs
df -hT /mnt/labfs
```

**Q15. Compare the file-system UUID with the UUID recorded in fstab.**
```bash
sudo blkid "$DEV"
lsblk -no UUID "$DEV"
grep '/mnt/labfs' /etc/fstab
```
The UUIDs should match exactly.

**Q16. Unmount the file system and run an ext4 check.**
```bash
sudo umount /mnt/labfs
sudo e2fsck -f "$DEV"
echo "$?"
```
Run `e2fsck` only on the unmounted test partition. Exit status `0` means no errors were found; `1` means errors were corrected. A read-only check can be requested with `sudo e2fsck -fn "$DEV"`.

**Q17. Explain the difference between checking a mounted and unmounted file system.**

Do not run repair checks on a mounted file system. The kernel may be changing it while the checker runs, so results can be unreliable and repairs can cause corruption. For this lab, unmount first and run the check only on the disposable test partition:
```bash
sudo umount /mnt/labfs
sudo e2fsck -f "$DEV"
```
For XFS, use the appropriate XFS repair tool on an unmounted file system.

**Q18. Correct an invalid UUID in the test fstab entry.**
```bash
# Ensure the test partition is unmounted before testing the broken entry.
sudo umount /mnt/labfs

# Change only the lab mount entry to an intentionally invalid UUID.
sudo sed -i '\|/mnt/labfs|s|^UUID=[^[:space:]]*|UUID=1111-WRONG|' /etc/fstab
sudo findmnt --verify
sudo mount -a

# Restore the actual UUID on the same lab entry.
UUID=$(sudo blkid -s UUID -o value "$DEV")
sudo sed -i "\|/mnt/labfs|s|^UUID=[^[:space:]]*|UUID=$UUID|" /etc/fstab
sudo findmnt --verify
sudo systemctl daemon-reload
sudo mount -a
findmnt /mnt/labfs
```
The intentionally invalid UUID should fail validation or mounting. Restore the correct UUID immediately. Do not reboot while the fstab entry is invalid.

**Q19. Remove the lab entry and verify that it is no longer mounted.**
```bash
sudo umount /mnt/labfs
sudo cp "$FSTAB_BACKUP" /etc/fstab
sudo systemctl daemon-reload
sudo mount -a
grep '/mnt/labfs' /etc/fstab
findmnt /mnt/labfs
```
The last two commands should show no lab entry or mount. Restore the backup created in Q11; do not delete unrelated fstab entries.

**Q20. Complete the workflow and clean up.**
```bash
# Stop if any command fails; run this workflow in the same Bash session as setup.
(
set -e
FSTAB_BACKUP=$(sudo mktemp /etc/fstab.lab10.XXXXXX)
if findmnt -rn -S "$DEV" >/dev/null; then
  sudo umount "$DEV" || exit 1
fi
# Formatting erases the target: confirm DEV is the unmounted test partition first.
sudo mkfs.ext4 "$DEV"
sudo mkdir -p /mnt/labfs
sudo mount "$DEV" /mnt/labfs
df -hT /mnt/labfs
findmnt /mnt/labfs
UUID=$(sudo blkid -s UUID -o value "$DEV")
sudo cp /etc/fstab "$FSTAB_BACKUP"
printf 'UUID=%s /mnt/labfs ext4 defaults,nofail 0 2\n' "$UUID" |
  sudo tee -a /etc/fstab
sudo findmnt --verify
sudo systemctl daemon-reload
sudo umount /mnt/labfs
sudo mount -a
findmnt /mnt/labfs
sudo umount /mnt/labfs
sudo e2fsck -f "$DEV"

# Restore the original fstab and release the disposable loop device.
sudo cp "$FSTAB_BACKUP" /etc/fstab
sudo systemctl daemon-reload
sudo rm "$FSTAB_BACKUP"
sudo rmdir /mnt/labfs
sudo losetup --detach "$LOOP"
rm -f "$HOME/lab10.img"
! grep '/mnt/labfs' /etc/fstab
! findmnt /mnt/labfs
)
```
Run the cleanup only after confirming the test file system is unmounted. The grouped commands run with `set -e` in a subshell; restore the backup manually if the workflow stops before cleanup.
