# Lab Sheet 4: User Management

Run these exercises on a test Ubuntu system. The commands create and modify local accounts and groups.

**Q1. Display the current username, UID, primary GID, and supplementary groups.**
```bash
whoami
id
groups
```
`id` displays the UID, GID, and group IDs. `groups` displays group names.

**Q2. Display a test user's `/etc/passwd` entry and explain its fields.**
```bash
grep '^root:' /etc/passwd
getent passwd labuser1   # Run after creating the test user
```
Format: `username:password-placeholder:UID:GID:GECOS:home-directory:login-shell`. The `x` placeholder means the password hash is stored in `/etc/shadow`.

**Q3. Display account/password status without displaying a password.**
```bash
sudo passwd -S root
sudo chage -l root
```
On Ubuntu, `passwd -S` commonly reports `P` for a usable password, `L` for a locked password, and `NP` when no password is set.

**Q4. Create `labuser1` with `useradd` and verify the account.**
```bash
sudo useradd labuser1
id labuser1
getent passwd labuser1
```
Ubuntu's `useradd` does not create a home directory unless `-m` is specified.

**Q5. Create `labuser2` with a home directory and verify its ownership.**
```bash
sudo useradd -m labuser2
ls -ld /home/labuser2
```
The owner and group should both be `labuser2`.

**Q6. Create `labuser3` with the `/bin/bash` shell.**
```bash
sudo useradd -s /bin/bash labuser3
getent passwd labuser3
```
The final field should be `/bin/bash`.

**Q7. Set a password for `labuser1` and verify its status.**
```bash
sudo passwd labuser1
sudo passwd -S labuser1
```
Enter the password interactively when prompted. The status should normally be `P`.

**Q8. Change `labuser1`'s password and check its aging information.**
```bash
sudo passwd labuser1
sudo passwd -S labuser1
sudo chage -l labuser1
```

**Q9. Change `labuser2`'s shell to `/bin/bash` with `usermod`.**
```bash
sudo usermod -s /bin/bash labuser2
getent passwd labuser2
```

**Q10. Change `labuser2`'s home directory first without moving its contents, then with the contents.**
```bash
echo "test data" | sudo tee /home/labuser2/file.txt

# Without -m: updates the account entry but does not move/create the directory.
sudo usermod -d /home/labuser2_new labuser2
getent passwd labuser2
ls -l /home/labuser2

# Restore the recorded path, then move the directory and its contents.
sudo usermod -d /home/labuser2 labuser2
sudo usermod -m -d /home/labuser2_moved labuser2
ls -l /home/labuser2_moved
```

**Q11. Create `userlabgroup`, add `labuser1`, and verify membership.**
```bash
sudo groupadd userlabgroup
sudo usermod -aG userlabgroup labuser1
id labuser1
getent group userlabgroup
```
Use `-aG` to append a supplementary group without removing existing supplementary memberships.

**Q12. Lock `labuser3`, verify the status, then unlock and verify again.**
```bash
sudo passwd labuser3
sudo usermod -L labuser3
sudo passwd -S labuser3
sudo usermod -U labuser3
sudo passwd -S labuser3
```
Set a password first so the status transition is clear. `passwd -l` and `passwd -u` are alternatives.

**Q13. Display detailed password-aging information for `labuser3`.**
```bash
sudo chage -l labuser3
```
The output includes the last password change, expiration dates, minimum/maximum days, and warning period.

**Q14. Set and verify an account expiration date for `labuser3`.**
```bash
sudo chage -E 2026-12-31 labuser3
sudo chage -l labuser3 | grep "Account expires"
```
To remove the account-expiration date, run `sudo chage -E -1 labuser3`.

**Q15. Change `labuser1`'s primary group to an existing group.**
```bash
sudo usermod -g userlabgroup labuser1
id labuser1
```
Lowercase `-g` sets the primary group; uppercase `-G` sets supplementary groups.

**Q16. Display user and group information with `id`, `groups`, and `getent`.**
```bash
id labuser1
groups labuser1
getent passwd labuser1
getent group userlabgroup
```

**Q17. Delete `labuser1` without deleting its home directory.**
```bash
sudo mkdir -p /home/labuser1
sudo userdel labuser1
getent passwd labuser1
ls -ld /home/labuser1
```
`getent` should return no account entry, while the home directory remains. The explicit `mkdir` makes the retained-directory result predictable even if the account was created without `-m`.

**Q18. Delete `labuser2` and its home directory.**
```bash
sudo userdel -r labuser2
getent passwd labuser2
ls -ld /home/labuser2_moved
```
The last command should report that the directory does not exist, assuming Q10 was completed.

**Q19. Complete workflow: create a user, set a password, configure a shell and group, then lock/unlock and verify.**
```bash
sudo useradd -m -s /bin/bash flowuser
sudo passwd flowuser
sudo groupadd flowgroup
sudo usermod -aG flowgroup flowuser
sudo usermod -L flowuser
sudo passwd -S flowuser
sudo usermod -U flowuser
sudo passwd -S flowuser
getent passwd flowuser
id flowuser
ls -ld /home/flowuser
```

**Q20. Clean up the test accounts and groups, then verify the result.**
```bash
sudo userdel -r flowuser
sudo userdel -r labuser3
sudo rm -rf /home/labuser1 /home/labuser2_new /home/labuser2_moved
sudo groupdel flowgroup
sudo groupdel userlabgroup
getent passwd | grep -E 'labuser|flowuser'
getent group | grep -E 'labuser|flowgroup|userlabgroup'
sudo pwck -r
sudo grpck -r
```
