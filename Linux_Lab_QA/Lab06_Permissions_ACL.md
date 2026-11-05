# Lab Sheet 6: File Permissions and ACLs

On a minimal Ubuntu installation, install ACL tools with `sudo apt update && sudo apt install acl`.

**Q1. Create a `lab6` directory and a `permissions.txt` file inside it.**
```bash
mkdir -p lab6
touch lab6/permissions.txt
cd lab6
```

**Q2. Display a long listing and identify the owner, group, and permissions.**
```bash
ls -l permissions.txt
```
In a permission string such as `-rw-rw-r--`, the first character is the file type (`-` means a regular file). The remaining nine characters describe owner (`rw-`), group (`rw-`), and others (`r--`) permissions. The listing also shows the owner and group names.

**Q3. Set symbolic permissions: owner read/write/execute, group read/write, and no permissions for others.**
```bash
chmod u=rwx,g=rw,o= permissions.txt
ls -l permissions.txt
```
The permission string should be `-rwxrw----`.

**Q4. Set numeric mode 640.**
```bash
chmod 640 permissions.txt
ls -l permissions.txt
```
The permission string should be `-rw-r-----`. Values are read `4`, write `2`, and execute `1`.

**Q5. Set numeric mode 755.**
```bash
chmod 755 permissions.txt
ls -l permissions.txt
```
The permission string should be `-rwxr-xr-x`.

**Q6. Remove execute permission for the owner.**
```bash
chmod u-x permissions.txt
ls -l permissions.txt
```

**Q7. Grant the group read and execute permissions without changing the owner permissions.**
```bash
chmod g+rx permissions.txt
ls -l permissions.txt
```

**Q8. Remove all permissions for others.**
```bash
chmod o-rwx permissions.txt
ls -l permissions.txt
```
The final three permission bits should be `---`.

**Q9. Create a `securedata` directory and set its mode to 750.**
```bash
mkdir securedata
chmod 750 securedata
ls -ld securedata
```
The permission string should be `drwxr-x---`.

**Q10. Explain and demonstrate the difference between file and directory permissions.**

| Permission | On a file | On a directory |
|-----------|-----------|----------------|
| `r` | Read file contents | List names in the directory |
| `w` | Modify file contents | Create, delete, or rename entries (execute permission is also needed) |
| `x` | Execute the file as a program | Traverse the directory and access entries by name |

Run this demonstration as a normal user in a disposable working directory. The root account can bypass many permission checks.
```bash
mkdir practice
echo hi > practice/a.txt
chmod 644 practice/a.txt

chmod u-x practice
cd practice              # Permission denied because the directory lacks execute permission
ls practice              # Names can be listed if read permission is present
cd ..
chmod u+x practice

chmod u-r practice
ls practice              # Permission denied because read permission is absent
cat practice/a.txt       # Works if execute permission is present and the filename is known
chmod u+r practice

chmod u-w practice
touch practice/b.txt     # Permission denied because the directory lacks write permission
chmod u+w practice
```

**Q11. Display and record the current umask.**
```bash
umask
umask -S
```

**Q12. Temporarily set umask to 027, create a file and directory, and inspect their permissions.**
```bash
old_umask=$(umask)
umask 027
touch file27
mkdir dir27
ls -ld file27 dir27
```
Typical results are mode 640 for the file and 750 for the directory. A file's requested base mode is normally 666; a directory's is 777.

**Q13. Restore the previous umask.**
```bash
umask "$old_umask"
umask
```

**Q14. Create a test user, group, and file for ACL practice.**
```bash
sudo useradd -m acluser
sudo groupadd aclgroup
touch aclfile.txt
```

**Q15. Display the file's ACL and identify its entries.**
```bash
getfacl aclfile.txt
```
The output includes the owner, owning group, and entries such as `user::rw-`, `group::r--`, and `other::r--`. Named `user:<name>:` and `group:<name>:` entries appear when extended ACLs are set. A `mask::` entry limits the effective permissions of named users, named groups, and the owning group.

**Q16. Grant a user read and write access to `aclfile.txt`.**
```bash
setfacl -m u:acluser:rw aclfile.txt
getfacl aclfile.txt
ls -l aclfile.txt
```
The ACL should include `user:acluser:rw-`, and `ls -l` should show a trailing `+`. The ACL mask can affect effective group-class permissions; inspect `getfacl` rather than inferring them only from the mode string.

**Q17. Grant a group read access.**
```bash
setfacl -m g:aclgroup:r aclfile.txt
getfacl aclfile.txt
```

**Q18. Modify one ACL entry and remove entries.**
```bash
setfacl -m u:acluser:r aclfile.txt
getfacl aclfile.txt
setfacl -x u:acluser aclfile.txt
setfacl -x g:aclgroup aclfile.txt
getfacl aclfile.txt
```

**Q19. Set a default ACL on a directory and verify inheritance by a new file.**
```bash
mkdir acldir
setfacl -d -m u:acluser:rw acldir
getfacl acldir
touch acldir/newfile.txt
getfacl acldir/newfile.txt
```
The directory should show `default:user:acluser:rw-`; the new file should inherit a named user entry, subject to the file's creation mode and ACL mask. If `#effective:r--` appears, inspect and adjust the ACL mask only when appropriate for the exercise.

**Q20. Complete the workflow and remove the test files and accounts.**
```bash
mkdir wf
cd wf
touch f.txt
chmod 640 f.txt
ls -l f.txt
umask 027
touch g.txt
ls -l g.txt
umask 022
setfacl -m u:acluser:rw f.txt
getfacl f.txt
mkdir d
setfacl -d -m u:acluser:rw d
touch d/h.txt
getfacl d/h.txt
setfacl -b f.txt
setfacl -k d
getfacl f.txt d
cd ..
rm -rf wf lab6 practice acldir securedata aclfile.txt file27 dir27
sudo userdel -r acluser
sudo groupdel aclgroup
```
Remove only the test files and directories created by this lab.
