# Lab 9: Disk and Storage Management

## Safe test setup

Use disposable image files attached as loop devices instead of real disks. Confirm that the image filenames below do not already contain files you need; `truncate` will create or resize them. Run the setup and later commands in the same Bash session so that the `LOOP` variable remains available.

```bash
# Image for mounting a file system without a partition table.
truncate -s 200M "$HOME/labfs.img"
sudo mkfs.ext4 -F "$HOME/labfs.img"

# Image for partition-table exercises.
truncate -s 300M "$HOME/labdisk.img"
LOOP=$(sudo losetup --find --show --partscan "$HOME/labdisk.img")
echo "$LOOP"
```
The loop device should look like `/dev/loopN`. Do not substitute a system disk. Detach the loop device only after unmounting it.

**Q1. Display mounted file systems and disk usage.**
```bash
df
```

**Q2. Display disk usage in human-readable units and explain the columns.**
```bash
df -h
```
Columns include filesystem/device, total size, used space, available space, percentage used, and mount point.

**Q3. Display the size of the current directory and its contents.**
```bash
du
du -sh .
```

**Q4. Display human-readable sizes for home-directory subdirectories, sorted largest first.**
```bash
du -h --max-depth=1 "$HOME" | sort -hr | head
```

**Q5. Display the size of a selected directory and its subdirectories.**
```bash
sudo du -h --max-depth=1 /var/log
sudo du -sh /var/log
sudo du -h --max-depth=2 /var
```

**Q6. Display block devices, disks, and partitions.**
```bash
lsblk
lsblk -a
```
`lsblk -a` also displays empty devices.

**Q7. Display device names, sizes, types, and mount points.**
```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS
```
Types may include `disk`, `part`, `loop`, `lvm`, and `rom`. An empty mount-point field means the device is not mounted.

**Q8. Display file-system information for block devices.**
```bash
lsblk -f
lsblk -o NAME,FSTYPE,LABEL,UUID,FSAVAIL,FSUSE%,MOUNTPOINTS
```

**Q9. Display mounted file systems and mount points.**
```bash
findmnt
findmnt -t ext4
mount | column -t | head
```

**Q10. Create `/mnt/labdisk` and verify its existence and permissions.**
```bash
sudo mkdir -p /mnt/labdisk
ls -ld /mnt/labdisk
```

**Q11. Mount the test file system at `/mnt/labdisk` and verify it.**
```bash
sudo mount -o loop "$HOME/labfs.img" /mnt/labdisk
findmnt /mnt/labdisk
df -hT /mnt/labdisk
```
Use only the image created in the safe test setup. Do not mount an unidentified device.

**Q12. Display usage for the mounted test file system.**
```bash
df -h /mnt/labdisk
```

**Q13. Create a test file and verify which file system contains it.**
```bash
echo "lab data" | sudo tee /mnt/labdisk/test.txt
ls -l /mnt/labdisk
df -h /mnt/labdisk/test.txt
findmnt -T /mnt/labdisk/test.txt
```

**Q14. Unmount the test file system and verify the result.**
```bash
sudo umount /mnt/labdisk
findmnt /mnt/labdisk
df -h | grep labdisk
ls -la /mnt/labdisk
```
The `findmnt` and `grep` commands should show no mounted lab file system. Files written to the mounted image will no longer appear while it is unmounted.

**Q15. Explain why an in-use file system cannot be unmounted, then release it and unmount.**
```bash
sudo mount -o loop "$HOME/labfs.img" /mnt/labdisk
cd /mnt/labdisk
```
In another terminal, inspect the process using the mount:
```bash
sudo lsof +f -- /mnt/labdisk
sudo fuser -vm /mnt/labdisk
```
The unmount can fail because a shell's current directory or an open file is using the mount. Return to the first terminal, leave the directory, and unmount:
```bash
cd "$HOME"
sudo umount /mnt/labdisk
```
Avoid force or lazy unmounts in this exercise; they can cause data loss or leave processes using the detached file system.

**Q16. Display the partition table on the disposable test image.**
```bash
sudo fdisk -l "$LOOP"
sudo parted "$LOOP" print
lsblk "$LOOP"
```

**Q17. Create a practice partition on the dedicated loop device and verify it.**
```bash
sudo parted --script "$LOOP" mklabel msdos mkpart primary ext4 1MiB 150MiB
sudo partprobe "$LOOP"
lsblk "$LOOP"
```
The output should show the loop device and a partition such as `/dev/loopNp1`. Never run these partitioning commands against a system disk.

**Q18. Create a file system on the test partition, mount it, and verify it.**
```bash
sudo mkfs.ext4 "${LOOP}p1"
sudo mount "${LOOP}p1" /mnt/labdisk
df -hT /mnt/labdisk
lsblk -f "$LOOP"
findmnt /mnt/labdisk
```
`mkfs.ext4` erases data on its target. Confirm that `${LOOP}p1` is the disposable test partition before running it.

**Q19. Unmount and verify the device and partition information.**
```bash
sudo umount /mnt/labdisk
findmnt /mnt/labdisk
lsblk -f "$LOOP"
ls -la /mnt/labdisk
```
Unmounting detaches the file system; it does not delete the partition or its data.

**Q20. Complete the mount, verification, unmount, and cleanup workflow.**
```bash
lsblk
df -h
sudo mkdir -p /mnt/labdisk
sudo mount "${LOOP}p1" /mnt/labdisk
df -h /mnt/labdisk
echo "final test" | sudo tee /mnt/labdisk/final.txt
cat /mnt/labdisk/final.txt
sudo umount /mnt/labdisk
findmnt /mnt/labdisk
lsblk -f "$LOOP"

# Cleanup after confirming that the test file systems are unmounted.
sudo losetup --detach "$LOOP"
rm -f "$HOME/labfs.img" "$HOME/labdisk.img"
sudo rmdir /mnt/labdisk
```
