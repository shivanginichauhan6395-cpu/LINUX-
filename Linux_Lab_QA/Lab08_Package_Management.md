# Lab 8: Package Management on Ubuntu

This lab uses Ubuntu's APT and dpkg tools. Use a disposable Ubuntu system or VM. Installing, upgrading, and removing packages changes system state. `tree` and `wget` are example packages; check whether a package was already installed before removing it.

**Q1. Display the dpkg version and verify that dpkg is available.**
```bash
dpkg --version
command -v dpkg
```

**Q2. Display the APT version and verify that apt is available.**
```bash
apt --version
command -v apt
```

**Q3. Search the local package database and check whether `bash` is installed.**
```bash
dpkg -l bash
dpkg-query -W -f='${binary:Package} ${Version} ${Status}\n' bash
```
An installed package reports the `install ok installed` status.

**Q4. Display detailed information for an installed package.**
```bash
apt show bash
dpkg-query -s bash
```
The output includes package name, version, architecture, dependencies, and description.

**Q5. List files installed by a package.**
```bash
dpkg -L bash
dpkg-query -L bash
```
To locate configuration files, inspect the package file list for paths under `/etc`.

**Q6. Find the package that owns a file.**
```bash
dpkg -S /usr/bin/ls
dpkg -S /etc/passwd
```

**Q7. Display an installed package's version and installation-related information.**
```bash
dpkg-query -W -f='${binary:Package} ${Version}\n' bash
apt show bash
```
APT's package metadata may not provide the local installation date. For local package transaction history, inspect `/var/log/dpkg.log` and rotated logs.

**Q8. Display configured APT repositories.**
```bash
grep -R --no-filename -E '^(deb |URIs:|Suites:|Components:)' \
  /etc/apt/sources.list /etc/apt/sources.list.d 2>/dev/null
```
Ubuntu versions may store repository configuration in traditional `.list` files or deb822 `.sources` files.

**Q9. Display repository and candidate-version information for a package.**
```bash
apt-cache policy bash
```
The output shows installed and candidate versions and the package sources that provide them.

**Q10. Search APT package metadata.**
```bash
apt search tree
apt-cache search "web server"
apt-cache show apache2
```

**Q11. Install `tree` and verify the installation.**
```bash
sudo apt update
sudo apt install tree
dpkg-query -W tree
tree --version
```

**Q12. Install `wget` and verify its installed version.**
```bash
sudo apt install wget
dpkg-query -W -f='${binary:Package} ${Version}\n' wget
apt-cache policy wget
```
Record whether `wget` was already installed before this exercise so you do not remove a pre-existing package during cleanup.

**Q13. List available package upgrades without applying them.**
```bash
sudo apt update
apt list --upgradable
```
`apt update` refreshes repository metadata; it does not upgrade installed packages.

**Q14. Upgrade one selected package and verify its version.**
```bash
dpkg-query -W -f='${Version}\n' tree
sudo apt install --only-upgrade tree
dpkg-query -W -f='${Version}\n' tree
```
If no newer version is available, the installed package remains unchanged.

**Q15. Upgrade all installed packages and inspect the result.**
```bash
sudo apt upgrade
apt list --upgradable
```
Review the proposed changes before confirming. A kernel or other core-system upgrade may require a reboot.

**Q16. Remove the test package and verify its status.**
```bash
sudo apt remove tree
dpkg-query -W -f='${db:Status-Status}\n' tree
command -v tree
```
The package status should no longer be `installed` (it may be `config-files`). `apt remove` leaves system-wide configuration files when applicable; `apt purge` removes them too. Do not purge packages unless the lab explicitly requires it.

**Q17. Identify package source and file ownership.**
```bash
apt-cache policy wget
apt show wget
dpkg -S /usr/bin/wget
```
If `wget` is not installed, `dpkg -S` will not find the file; install it before running that command.

**Q18. Inspect package transaction history.**
```bash
less /var/log/apt/history.log
less /var/log/dpkg.log
```
Press `q` to exit `less`. Rotated logs may contain older transactions.

**Q19. Complete workflow: verify, install, upgrade, and remove a test package.**
```bash
dpkg-query -W tree 2>/dev/null || echo "tree is not installed"
sudo apt install tree
dpkg-query -W -f='${db:Status-Status}\n' tree
sudo apt install --only-upgrade tree
sudo apt remove tree
dpkg-query -W -f='${db:Status-Status}\n' tree
```
If `tree` was installed before the lab, avoid removing it; choose a different test package that was not previously installed.

**Q20. Clean up only packages installed for this lab and verify repository configuration.**
```bash
sudo apt remove tree
```
Remove `wget` only if this lab installed it and it was not present beforehand:
```bash
sudo apt remove wget
```
Review repository configuration with the command from Q8. Do not delete files from `/etc/apt` or run `apt autoremove` without reviewing the proposed changes.
