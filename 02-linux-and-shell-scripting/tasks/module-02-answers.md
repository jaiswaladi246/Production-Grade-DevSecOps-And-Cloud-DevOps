# Module 2 – Answers | Linux & Shell Scripting

> **Role context:** Infrastructure and Cloud Analyst (Intermediate) — Air Canada
> **Interview depth:** Production-grade, enterprise-level answers

---

## Task 1 – Linux Introduction & OS Details

### Commands

```bash
# Distribution and version
cat /etc/os-release

# Kernel version
uname -r

# Full system info
uname -a
```

### Sample Output

```
NAME="Ubuntu"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
KERNEL: 5.15.0-91-generic
```

### Interview Connect: Why Linux dominates DevOps environments?

> Linux is preferred for servers and DevOps tooling for several concrete reasons:
>
> - **Open source & free**: No licensing cost per VM — critical when you're running hundreds of EC2 or Azure VMs at scale.
> - **Stability**: Linux servers routinely run for years without reboots. Windows requires frequent patch-reboots that disrupt services.
> - **Performance**: Low overhead OS — more resources go to the workload, not the OS itself.
> - **CLI-first**: Everything in Linux is scriptable and automatable. You can manage 1,000 servers with a single command. Windows PowerShell is improving but Linux CLI is the standard in cloud.
> - **Container native**: Docker, Kubernetes, and virtually every DevOps tool runs natively on Linux. Docker on Windows/Mac runs a Linux VM under the hood.
> - **Security model**: Fine-grained permissions (user/group/other), SELinux/AppArmor, and a minimal attack surface with no GUI required.
> - **Cloud default**: AWS, Azure, and GCP all default to Linux AMIs/images for compute. Over 90% of cloud workloads run Linux.
>
> At Air Canada, every CI/CD agent, every Kubernetes node, and every cloud VM runs Linux. Not knowing Linux in a cloud/DevOps role is equivalent to a mechanic not knowing how to open a hood.

---

## Task 2 – Basic Linux Navigation

### Commands

```bash
pwd                        # Print working directory
ls -la                     # List all files with permissions
ls -lh /var/log            # Human-readable sizes
cd /etc                    # Change to absolute path
cd ../                     # Move up one level (relative)
mkdir -p /opt/myapp/config # Create nested directories
touch app.log              # Create empty file
rm app.log                 # Remove file
rm -rf /opt/myapp          # Remove directory recursively (use with caution)
```

### Interview Connect: Difference between relative and absolute paths?

> - **Absolute path**: Always starts with `/` (root). Works regardless of your current directory. Example: `/etc/nginx/nginx.conf`. Use absolute paths in scripts and cron jobs — they are predictable and unambiguous.
> - **Relative path**: Relative to your current working directory. Example: `../config/settings.yaml`. Use relative paths in application code where portability matters.
>
> In production scripts I always use absolute paths. If a cron job uses a relative path and runs from a different working directory than expected, it silently fails — a very common and hard-to-debug issue in production environments.

---

## Task 3 – File Operations

### Commands

```bash
cp app.conf app.conf.bak           # Copy (backup before editing)
cp -r /opt/app /opt/app-backup     # Recursive copy of directory
mv app.conf app-v2.conf            # Rename file
mv /tmp/config.yaml /etc/myapp/    # Move file to directory
ls *.log                           # Wildcard: all .log files
rm -f *.tmp                        # Delete all .tmp files
cp *.conf /backup/                 # Copy all .conf files to backup
```

### Interview Connect: How do you safely move files on servers?

> In production I follow this pattern:
>
> 1. **Always take a backup first**: `cp nginx.conf nginx.conf.bak.$(date +%Y%m%d)`
> 2. **Verify the destination has space**: `df -h /destination`
> 3. **Use `rsync` instead of `mv` for large files across filesystems**: `rsync -av --progress source/ dest/` — it's resumable and verifiable. `mv` across filesystems is actually a copy + delete, which risks data loss on failure.
> 4. **Verify the move**: After `rsync`, compare checksums with `md5sum` or `sha256sum` before deleting source.
> 5. **Never `rm -rf` in production without a second pair of eyes**. The famous `rm -rf /` incident at GitLab in 2017 deleted their primary database. Use `rm -i` for interactive confirmation or alias `rm` to `rm -i` in your shell profile.

---

## Task 4 – Viewing Files

### Commands

```bash
cat /etc/hosts                     # Print entire file
less /var/log/syslog               # Paginated view (q to quit, / to search)
more /etc/nginx/nginx.conf         # Older paginator
head -20 /var/log/auth.log         # First 20 lines
tail -50 /var/log/nginx/access.log # Last 50 lines
tail -f /var/log/app.log           # Follow live log output
tail -f /var/log/app.log | grep ERROR  # Filter live log stream
```

### Interview Connect: Why is `less` preferred over `cat` for logs?

> `cat` dumps the entire file to stdout — if the file is 2GB (common for production logs), it floods your terminal, can crash slow connections, and gives you no ability to scroll back or search.
>
> `less` is a pager — it reads the file lazily (only what fits on screen), lets you scroll forward/back, and supports `/pattern` search. It never loads the entire file into memory. For a 10GB nginx access log, `cat` is dangerous; `less` handles it fine.
>
> In production I use:
> - `less` for reading configuration files and logs
> - `tail -f` for live log monitoring during a deployment or incident
> - `tail -f logfile | grep -i error` to filter live errors during an incident
> - `zless` or `zcat` for compressed log archives (`.gz`)

---

## Task 5 – File Editing (Vim & Nano)

### Nano Basics

```bash
nano /etc/hosts
# Edit → Ctrl+O to save → Enter to confirm → Ctrl+X to exit
```

### Vim Basics

```bash
vim /etc/nginx/nginx.conf

# Modes:
# Normal mode  → default on open
# Insert mode  → press 'i' to enter, 'Esc' to return to normal
# Command mode → press ':' from normal mode

# Essential commands:
i          # Enter insert mode
Esc        # Return to normal mode
:w         # Save
:q         # Quit
:wq        # Save and quit
:q!        # Quit without saving (force)
:set nu    # Show line numbers
/pattern   # Search forward
n          # Next match
dd         # Delete current line
yy         # Yank (copy) current line
p          # Paste
u          # Undo
:%s/old/new/g  # Find and replace all
```

### Interview Connect: Why vim is commonly used in production?

> Vim is available on virtually every Linux system by default — even minimal Alpine-based containers and stripped-down cloud AMIs. Nano may not be installed. When you SSH into a production server during an incident at 2 AM, you need an editor that is **always there**.
>
> Beyond availability:
> - **No mouse required** — works over slow/unstable SSH connections
> - **Extremely fast** for experienced users — navigating, searching, replacing across large config files without leaving the keyboard
> - **Scripted edits** with `vim -c ":%s/8080/9090/g | wq" config.yaml` — useful in scripts
> - **`vimdiff`** for comparing two config files side-by-side during debugging
>
> I don't expect everyone to be a vim expert, but knowing `i`, `Esc`, `:wq`, `:q!`, and `/search` is the minimum survival kit for any cloud/DevOps engineer.

---

## Task 6 – Linux Directory Structure

### Key Directories

```bash
/bin        # Essential user binaries (ls, cp, mv, bash)
/sbin       # System binaries (reboot, fsck, ifconfig) — typically for root
/etc        # All configuration files (nginx, ssh, cron, hosts, fstab)
/var        # Variable data — logs, spool, runtime files
/var/log    # System and application logs
/home       # User home directories (/home/dhruwang)
/opt        # Third-party/optional software (Jenkins, custom apps)
/tmp        # Temporary files — cleared on reboot (do NOT store persistent data here)
/proc       # Virtual filesystem — live kernel and process data (not on disk)
/usr        # User programs, libraries, documentation
/root       # Home directory for the root user
/dev        # Device files (disks: /dev/sda, terminals: /dev/tty)
/mnt        # Temporary mount points for filesystems
/sys        # Virtual filesystem exposing kernel/hardware info
```

### Interview Connect: Where do logs and configs live?

> **Configs**: Always in `/etc`. Every service places its configuration here:
> - `/etc/nginx/nginx.conf` — Nginx web server
> - `/etc/ssh/sshd_config` — SSH daemon
> - `/etc/crontab` — system cron jobs
> - `/etc/fstab` — filesystem mount definitions
> - `/etc/hosts` — local DNS overrides
>
> **Logs**: Always in `/var/log`:
> - `/var/log/syslog` or `/var/log/messages` — system-wide log
> - `/var/log/auth.log` — authentication events (SSH logins, sudo)
> - `/var/log/nginx/access.log` and `error.log` — Nginx
> - `/var/log/docker.log` or `journalctl -u docker` — Docker daemon
>
> In cloud environments on AWS/Azure, system logs are also shipped to CloudWatch Logs or Azure Monitor using a log agent (CloudWatch Agent, Fluent Bit). At Air Canada, centralised log aggregation is critical for compliance and incident response.

---

## Task 7 – Log Monitoring

### Commands

```bash
# Follow system log live
tail -f /var/log/syslog

# Follow and filter simultaneously
tail -f /var/log/nginx/error.log | grep -i "502\|503\|error"

# Using journalctl (systemd-based systems)
journalctl -u nginx -f             # Follow nginx service logs
journalctl -u ssh --since "1 hour ago"
journalctl -p err -f               # Only error-level and above

# Search logs for a pattern with context
grep -C 5 "OutOfMemory" /var/log/syslog   # 5 lines before and after
```

### Interview Connect: How do you debug issues using logs?

> My approach during a production incident follows a structured flow:
>
> 1. **Identify the symptom**: Alert fires — e.g., "HTTP 502 spike on booking API"
> 2. **Check application logs first**: `tail -100 /var/log/app/error.log` — look for stack traces or connection refused errors
> 3. **Check system logs**: `journalctl -p err --since "10 minutes ago"` — kernel OOM killer? disk full?
> 4. **Check service status**: `systemctl status nginx` — is the process running? any recent restarts?
> 5. **Correlate timing**: When did the error start? Match with recent deployments or config changes
> 6. **Check resources**: `df -h` (disk), `free -m` (memory), `top` (CPU) — is this a resource exhaustion?
> 7. **Narrow the scope**: Filter logs by timestamp of the incident window, not the entire log file
>
> In production at scale, I'd use Datadog, Grafana Loki, or CloudWatch Logs Insights to query across all instances simultaneously — because `tail`-ing individual servers doesn't scale when you have 50 nodes behind a load balancer.

---

## Task 8 – Virtual Machine Basics

### Commands

```bash
# CPU info
lscpu
nproc                          # Number of processing units

# RAM
free -h                        # Human-readable memory usage
cat /proc/meminfo

# Disk
lsblk                          # Block devices
fdisk -l                       # Partition table
df -h                          # Filesystem usage

# Network interfaces
ip addr show
ip link show
cat /etc/network/interfaces    # (Debian/Ubuntu) static config

# Open ports
ss -tulnp                      # Socket statistics — preferred over netstat
netstat -tulnp                 # Older alternative
```

### Interview Connect: VM vs physical server?

> | Dimension | Physical Server (Bare Metal) | Virtual Machine |
> |---|---|---|
> | Resource isolation | Dedicated hardware | Shared hardware via hypervisor |
> | Provisioning time | Hours to days (rack, cable, image) | Minutes (API call) |
> | Cost | High CapEx | Pay-as-you-go OpEx |
> | Performance | Maximum — no hypervisor overhead | ~5-10% overhead for compute-heavy workloads |
> | Flexibility | Fixed hardware specs | Resize vCPU/RAM on demand |
> | Use case | High-performance databases, HPC | Most cloud workloads |
>
> In cloud environments like AWS and Azure, VMs (EC2, Azure VMs) are the primary compute primitive. For Air Canada's non-latency-sensitive workloads, VMs on EC2 or Azure are standard. For high-throughput database workloads, dedicated/bare-metal instances (e.g., `i3.metal` on AWS) may be used.

---

## Task 9 – SSH Access

### Commands

```bash
# Connect to a remote server
ssh -i ~/.ssh/air-canada-key.pem ubuntu@10.0.1.45

# Specify port (non-default)
ssh -i ~/.ssh/key.pem -p 2222 ubuntu@10.0.1.45

# Verify SSH port is listening
ss -tulnp | grep 22

# SSH config file for convenience
vim ~/.ssh/config
```

```
# ~/.ssh/config example
Host bastion-prod
    HostName 10.0.1.45
    User ubuntu
    IdentityFile ~/.ssh/air-canada-key.pem
    Port 22

Host app-server
    HostName 10.0.2.100
    User ec2-user
    ProxyJump bastion-prod        # Jump through bastion host
    IdentityFile ~/.ssh/key.pem
```

```bash
ssh bastion-prod     # Now works without remembering IPs or key paths
```

### Interview Connect: What is SSH?

> SSH (Secure Shell) is a cryptographic network protocol for operating network services securely over an unsecured network. It provides encrypted communication between client and server using asymmetric key cryptography.
>
> In production, SSH is how engineers access Linux VMs, Kubernetes nodes (via `kubectl exec`), and bastion hosts. Key concepts:
>
> - **Port 22** is the default — often changed to a non-standard port as a security hardening measure
> - **Bastion host / Jump host**: In a production VPC, application servers have no public IP. You SSH to a hardened bastion in the public subnet, then jump to private instances. This is the standard pattern at enterprises like Air Canada.
> - **SSH tunneling**: Port forwarding through SSH to access internal services (e.g., a database port) securely without exposing them publicly — `ssh -L 5432:db.internal:5432 bastion-prod`
> - **`~/.ssh/config`**: Stores connection profiles — saves time and reduces human error during incidents

---

## Task 10 – SSH Key-Based Authentication

### Commands

```bash
# Generate key pair (on local machine)
ssh-keygen -t ed25519 -C "dhruwang@aircanada-infra" -f ~/.ssh/ac_infra_key

# Copy public key to server
ssh-copy-id -i ~/.ssh/ac_infra_key.pub ubuntu@server-ip
# OR manually:
cat ~/.ssh/ac_infra_key.pub >> ~/.ssh/authorized_keys

# Disable password login (on server)
sudo vim /etc/ssh/sshd_config
# Set:  PasswordAuthentication no
#       PubkeyAuthentication yes
#       PermitRootLogin no

sudo systemctl restart sshd

# Verify
ssh -i ~/.ssh/ac_infra_key ubuntu@server-ip
```

### Interview Connect: Why key-based auth is more secure?

> Password authentication has several weaknesses:
> - Vulnerable to brute-force and credential stuffing attacks
> - Passwords can be phished, reused across systems, or found in data breaches
> - Shared passwords across teams violate least-privilege and make audit trails impossible
>
> SSH key-based authentication is superior because:
> - **Asymmetric cryptography**: The private key never leaves your machine. The server only holds the public key. Even if someone steals the public key it's useless without the private key.
> - **Passphrase-protected keys**: Add a second factor even on the key itself
> - **No brute force surface**: A 2048-bit RSA or ed25519 key cannot be brute-forced in any practical timeframe
> - **Auditability**: Each engineer has their own key pair — you know exactly who logged in and when from `auth.log`
> - **`ed25519` is preferred over RSA**: Shorter keys, faster, more secure against side-channel attacks
>
> In enterprise environments like Air Canada, SSH access to production is typically managed via AWS Systems Manager Session Manager or Azure Bastion — which eliminates the need for open port 22 entirely, using IAM-based access control with full session recording for compliance.

---

## Task 11 – Elastic IP Association

### AWS Console / CLI Steps

```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Associate with instance
aws ec2 associate-address \
  --instance-id i-0abc123def456 \
  --allocation-id eipalloc-0abc123def456

# Verify
aws ec2 describe-addresses --allocation-ids eipalloc-0abc123def456
curl http://<elastic-ip>   # Test connectivity
```

### Interview Connect: Why Elastic IP is used?

> By default, an EC2 instance gets a dynamic public IP that changes every time it's stopped and restarted. This breaks:
> - DNS records pointing to the instance
> - Firewall whitelists on client side
> - SSL certificates tied to an IP
>
> An **Elastic IP** is a static public IPv4 address you own in your AWS account. It persists across instance stop/start cycles and can be instantly remapped to another instance — critical for failover.
>
> **Production use cases**:
> - **Bastion hosts**: Clients whitelist the bastion's Elastic IP in their firewall rules — it can't change
> - **Legacy systems** that can't use DNS-based load balancing and require a fixed IP
> - **Disaster recovery**: If the primary instance fails, disassociate the EIP and associate it to the standby instance — DNS propagation is instant
>
> **Important**: Elastic IPs are free when associated with a running instance. AWS charges ~$0.005/hr when an EIP is allocated but NOT associated — so always release unused EIPs to avoid cost creep.
>
> In modern cloud architectures, I prefer DNS + ALB over Elastic IPs for scalability. Elastic IPs are for edge cases where a fixed IP is a hard requirement.

---

## Task 12 – curl Usage

### Commands

```bash
# Basic GET request
curl https://aircanada.com

# Check HTTP status code only
curl -o /dev/null -s -w "%{http_code}" https://api.aircanada.com/health

# Fetch headers only
curl -I https://aircanada.com

# POST with JSON payload
curl -X POST https://api.example.com/flights \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"origin":"YYZ","destination":"YVR"}'

# Follow redirects
curl -L https://aircanada.com

# Verbose output (shows SSL, headers, timing)
curl -v https://api.example.com/health

# Save response to file
curl -o response.json https://api.example.com/data

# Timeout after 10 seconds
curl --connect-timeout 10 --max-time 30 https://api.example.com
```

### Interview Connect: How do you test APIs from a server?

> `curl` is the standard tool for API testing from Linux servers. In production I use it for:
>
> - **Health checks in deployment scripts**: `curl -f http://localhost:8080/health || exit 1` — `-f` makes curl return a non-zero exit code on HTTP 4xx/5xx, so the deployment script fails fast
> - **Testing after a config change**: Verify the service responds before removing the old version
> - **Debugging SSL issues**: `curl -v https://endpoint` shows the full TLS handshake, certificate chain, and cipher suite
> - **Scripted monitoring**: A simple bash script curling `/health` every 60 seconds and alerting on non-200 responses — basic synthetic monitoring
> - **Testing internal endpoints** from within a VPC: When a service is not externally accessible, SSH to a server in the same subnet and curl the private IP

---

## Task 13 – wget Usage

### Commands

```bash
# Download a file
wget https://example.com/agent.tar.gz

# Download to specific path
wget -O /opt/downloads/agent.tar.gz https://example.com/agent.tar.gz

# Resume interrupted download
wget -c https://example.com/largefile.tar.gz

# Download in background
wget -b https://example.com/file.tar.gz

# Verify download integrity (compare with published checksum)
wget https://example.com/file.tar.gz
wget https://example.com/file.tar.gz.sha256
sha256sum -c file.tar.gz.sha256

# Mirror a directory (for offline archiving)
wget -r -np -nH https://example.com/docs/
```

### Interview Connect: wget vs curl?

> | Feature | wget | curl |
> |---|---|---|
> | Primary use | File downloading | Data transfer / API testing |
> | Resume downloads | Yes (`-c`) | Limited |
> | Recursive download | Yes (`-r`) | No |
> | Supports protocols | HTTP, HTTPS, FTP | HTTP, HTTPS, FTP, SFTP, SCP, SMTP, and many more |
> | Scripting / API calls | Limited | Excellent — supports all HTTP methods, headers, auth |
> | Output | File by default | stdout by default |
>
> **Rule of thumb**: Use `wget` to download files. Use `curl` to interact with APIs and HTTP services. In CI/CD pipelines, `curl` is more versatile — it's my default tool. `wget` is handy for downloading binary releases or artifacts from package registries.

---

## Task 14 – Process Listing

### Commands

```bash
# Snapshot of processes
ps aux                         # All processes, BSD style
ps aux --sort=-%cpu | head 10  # Top 10 by CPU
ps aux --sort=-%mem | head 10  # Top 10 by memory
ps -ef                         # Full-format listing (shows PPID)

# Interactive process viewer
top                            # Default process monitor
htop                           # Enhanced, color-coded (install separately)

# Find a specific process
ps aux | grep nginx
pgrep nginx                    # Returns PIDs only

# Process tree
pstree -p                      # Shows parent-child relationships
```

### Interview Connect: How do you find high CPU processes?

> In a production incident where a server is unresponsive due to high CPU:
>
> 1. `top` — immediately shows the process consuming the most CPU at the top (sorted by %CPU by default). Press `P` to sort by CPU, `M` for memory.
> 2. `ps aux --sort=-%cpu | head 5` — quick snapshot, useful for scripting alerts
> 3. `pidstat -u 1 5` — CPU usage per process sampled every 1 second, 5 times (from `sysstat` package) — better for CI scripting
> 4. Once you have the PID: `cat /proc/<PID>/cmdline` — see the exact command that process is running
> 5. `strace -p <PID>` — trace system calls of a running process to understand what it's doing
>
> In cloud environments, I'd also check CloudWatch / Azure Monitor for CPU metrics across the entire fleet — a single server check doesn't give you the cluster view.

---

## Task 15 – Process Control

### Commands

```bash
# Run process in background
./backup-script.sh &
nohup ./long-running-job.sh > /var/log/job.log 2>&1 &  # Survives logout

# List background jobs
jobs -l

# Bring background job to foreground
fg %1

# Send to background after starting
Ctrl+Z   # Suspend
bg %1    # Resume in background

# Kill process gracefully (SIGTERM — process cleans up)
kill <PID>
kill -15 <PID>

# Kill process forcefully (SIGKILL — immediate, no cleanup)
kill -9 <PID>

# Kill by name
pkill nginx
killall java

# Kill all processes matching pattern
pkill -f "python worker.py"
```

### Interview Connect: kill vs kill -9?

> `kill <PID>` sends **SIGTERM (signal 15)** — a polite termination request. The process receives the signal and can:
> - Finish in-flight requests
> - Flush buffers to disk
> - Close database connections cleanly
> - Delete lock files
> - Write final log entries
>
> `kill -9 <PID>` sends **SIGKILL** — the kernel immediately terminates the process. The process has **no opportunity to clean up**. This can cause:
> - Corrupt files if the process was mid-write
> - Orphaned child processes
> - Incomplete database transactions
> - Stale lock files that prevent the service from restarting
>
> **Rule**: Always try `kill` first, wait 10-15 seconds, then use `kill -9` only if the process hasn't exited. In systemd, `systemctl stop nginx` handles this sequence automatically — it sends SIGTERM, waits for `TimeoutStopSec` (default 90 seconds), then sends SIGKILL.
>
> In production, a process that ignores SIGTERM is often a sign of a bug (blocked I/O, deadlock) that warrants investigation, not just a forced kill.

---

## Task 16 – Disk Usage Analysis

### Commands

```bash
# Filesystem usage (high-level, per mount)
df -h                          # Human readable
df -h /var                     # Specific mount point
df -ih                         # Inode usage (important — disk can be "full" on inodes)

# Directory-level usage
du -sh /var/log                # Size of /var/log directory
du -sh /* 2>/dev/null          # Size of all top-level directories
du -h --max-depth=1 /var       # One level deep in /var

# Find largest files
find /var -type f -printf '%s %p\n' | sort -rn | head 20

# Find largest directories
du -h /var/log | sort -rh | head 10
```

### Interview Connect: How do you debug disk full issues?

> **Disk full** (`No space left on device`) is one of the most common production incidents. My approach:
>
> 1. **Confirm which filesystem is full**: `df -h` — identify the mount point at 100%
> 2. **Find the culprit directory**: `du -h --max-depth=2 /var | sort -rh | head 20`
> 3. **Common culprits in production**:
>    - `/var/log` — unbounded log files (fix: configure log rotation with `logrotate`)
>    - `/var/lib/docker` — Docker images, stopped containers, dangling volumes (`docker system prune`)
>    - `/tmp` — temp files from crashed processes
>    - Core dump files — `find / -name "core.*" -type f`
>    - Old journal logs — `journalctl --disk-usage` → `journalctl --vacuum-size=500M`
> 4. **Check inodes separately**: `df -ih` — even if block usage is fine, inode exhaustion shows as disk full. Common cause: millions of tiny temp files.
> 5. **Immediate relief**: Truncate (don't delete) a live log file: `> /var/log/app/error.log` (safe — doesn't break the file handle the app holds open)
> 6. **Long-term fix**: Configure `logrotate`, set Docker log size limits (`"log-opts": {"max-size": "100m", "max-file": "5"}`), add disk usage alerts at 80% threshold in CloudWatch/Azure Monitor.

---

## Task 17 – Disk Mounting

### Commands

```bash
# List block devices
lsblk
fdisk -l

# Format a new disk (EBS volume attached to EC2)
sudo mkfs.ext4 /dev/xvdf

# Create mount point
sudo mkdir -p /data

# Mount
sudo mount /dev/xvdf /data

# Verify
df -h /data
mount | grep /data

# Unmount
sudo umount /data
# If "device is busy":
lsof /data          # Find which process is using it
fuser -vm /data     # Alternative
```

### Interview Connect: What is mounting?

> In Linux, every disk, partition, or network share must be **mounted** to a directory in the filesystem tree before it can be accessed. Mounting attaches the filesystem on a device to a mount point (a directory).
>
> Think of it like plugging in a USB drive — the OS detects the device, but you need to "mount" it to a directory to read its contents.
>
> In cloud context: When you attach an EBS volume in AWS or a Managed Disk in Azure to a VM, the OS sees it as a block device (`/dev/xvdf`, `/dev/sdb`) but it's inaccessible until you format it (if new) and mount it to a directory. This is a common Day 1 task when setting up data volumes for databases, application storage, or log directories.

---

## Task 18 – Persistent Mounts

### Commands

```bash
# Find UUID of disk (use UUID, not /dev/xvdf — device names can change on reboot)
blkid /dev/xvdf

# Edit fstab
sudo vim /etc/fstab
```

```
# /etc/fstab entry
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /data  ext4  defaults,nofail  0  2
```

```bash
# Test fstab without rebooting
sudo mount -a

# Verify
df -h /data
```

### Interview Connect: Why persistent mounts matter?

> `/etc/fstab` defines which filesystems are mounted automatically at boot. Without it, every time the server reboots (maintenance, patching, auto-scaling lifecycle), the attached volumes would need to be remounted manually — causing service outages.
>
> **Critical details for production**:
> - **Always use UUID, never `/dev/xvdf`**: Device names (`xvdf`, `sdb`) are not guaranteed to be the same after reboot, especially in cloud environments where disks are hotplugged. UUIDs are persistent.
> - **Use `nofail` option**: If the disk is unavailable at boot (e.g., EBS volume detached), `nofail` allows the system to boot anyway instead of dropping into emergency mode. This is critical for cloud VMs.
> - **Test with `mount -a`** before rebooting — a bad fstab entry can make a server unbootable, requiring rescue mode.
> - **The 6th field (pass)**: `2` means fsck runs on this filesystem at boot (after root). `0` disables it. For cloud VMs with ephemeral disks, set to `0`.

---

## Task 19 – NFS Setup

### NFS Server

```bash
# Install NFS server
sudo apt install nfs-kernel-server -y

# Create shared directory
sudo mkdir -p /srv/nfs/shared
sudo chown nobody:nogroup /srv/nfs/shared
sudo chmod 755 /srv/nfs/shared

# Define export
sudo vim /etc/exports
```

```
# /etc/exports
/srv/nfs/shared  10.0.1.0/24(rw,sync,no_subtree_check,no_root_squash)
```

```bash
sudo exportfs -arv
sudo systemctl enable nfs-kernel-server --now
```

### NFS Client

```bash
sudo apt install nfs-common -y
sudo mkdir -p /mnt/shared
sudo mount 10.0.1.10:/srv/nfs/shared /mnt/shared

# Make persistent
echo "10.0.1.10:/srv/nfs/shared  /mnt/shared  nfs  defaults,_netdev,nofail  0  0" | sudo tee -a /etc/fstab
```

### Interview Connect: When is NFS used?

> NFS (Network File System) is used when multiple servers need to read/write the same filesystem concurrently. Common production use cases:
>
> - **Shared configuration files**: Multiple app servers reading the same config directory
> - **Kubernetes Persistent Volumes with ReadWriteMany**: Pods on different nodes sharing a volume (EFS on AWS = managed NFS, Azure Files = managed SMB/NFS)
> - **CI/CD artifact caching**: Build agents sharing a common cache directory
> - **Legacy application shared storage**: Apps that were designed for a single-server model and read/write to local paths — NFS lets them work in a multi-server cluster without code changes
>
> In AWS, I prefer **Amazon EFS** (Elastic File System) over self-managed NFS — it's fully managed, auto-scales, and integrates with EKS. In Azure, **Azure Files** with NFS 4.1 protocol serves the same purpose. Self-managed NFS is still common in on-premises or hybrid environments.

---

## Task 20 – Linux Networking Basics

### Commands

```bash
# IP addresses and interfaces
ip addr show
ip addr show eth0

# Routing table
ip route show

# Interface up/down
sudo ip link set eth0 down
sudo ip link set eth0 up

# Open ports and listening services
ss -tulnp               # TCP/UDP listening ports with PID
ss -tulnp | grep :80    # Check if port 80 is open

# Connectivity test
ping -c 4 8.8.8.8
traceroute 8.8.8.8       # Path to destination
mtr 8.8.8.8              # Live traceroute (install mtr-tiny)

# Bandwidth usage
iftop                    # Live per-connection bandwidth
nethogs                  # Bandwidth per process
```

### Interview Connect: How do you debug network issues?

> I follow the OSI model top-down (or bottom-up depending on the symptom):
>
> **Layer 1 (Physical)**: Is the interface up? `ip link show` — look for `UP` state
>
> **Layer 3 (Network)**: Can I reach the gateway? `ping <default-gateway>` → `ip route show` to find the gateway
>
> **DNS (Layer 7 prerequisite)**: Can I resolve hostnames? `nslookup api.aircanada.com` — if ping by IP works but hostname fails, it's a DNS issue
>
> **Application layer**: Is the port open and accepting connections? `telnet <host> <port>` or `nc -zv <host> 443`
>
> **Security groups / firewall**: Is traffic blocked? `ss -tulnp` on the server to confirm the service is listening, then check security group rules in AWS console or `ufw status` locally
>
> **Common tools**:
> - `ss -tulnp` — what's listening
> - `nc -zv host port` — can I reach that port
> - `tcpdump -i eth0 port 443` — packet-level capture for deep debugging
> - `curl -v` — full HTTP connection debug including TLS

---

## Task 21 – DNS & Connectivity

### Commands

```bash
# Ping (ICMP connectivity)
ping -c 4 aircanada.com
ping -c 4 8.8.8.8          # Test raw IP (rules out DNS)

# DNS lookup
nslookup aircanada.com
nslookup aircanada.com 8.8.8.8   # Query specific DNS server

dig aircanada.com                 # Detailed DNS response
dig aircanada.com MX              # Mail exchange records
dig aircanada.com +short          # Just the IP
dig @8.8.8.8 aircanada.com        # Use Google DNS

# DNS resolution path
cat /etc/resolv.conf               # Current DNS servers
cat /etc/hosts                     # Local overrides
```

### Interview Connect: What happens when you hit a URL?

> When a user types `https://aircanada.com` in a browser, the following happens:
>
> 1. **Browser cache check**: Does the browser have a cached DNS record? If yes, skip DNS.
> 2. **OS cache check**: `/etc/hosts` first, then OS DNS cache.
> 3. **DNS resolution**: Query the configured DNS resolver (`/etc/resolv.conf`). The resolver performs recursive lookup: root nameserver → `.com` TLD nameserver → Air Canada's authoritative nameserver → returns A record (IP address).
> 4. **TCP handshake**: Browser opens a TCP connection to the IP on port 443 (SYN → SYN-ACK → ACK).
> 5. **TLS handshake**: Client and server negotiate TLS version, cipher suite, and exchange certificates. Server sends its SSL certificate; client validates it against trusted CAs.
> 6. **HTTP request**: Browser sends `GET / HTTP/1.1` with headers (Host, cookies, Accept-Encoding).
> 7. **Load balancer**: The request hits an ALB/nginx which routes to a backend server based on routing rules.
> 8. **Application server**: Processes the request, queries databases/APIs, generates response.
> 9. **Response**: HTML/JSON returned, browser renders the page.
>
> Knowing this sequence is critical for debugging. "Site is down" could be a DNS issue, TLS cert expiry, load balancer misconfiguration, or application crash — each requires a different fix.

---

## Task 22 – Firewall Management (UFW)

### Commands

```bash
# Install UFW
sudo apt install ufw -y

# Default policies (deny incoming, allow outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow essential services
sudo ufw allow 22/tcp          # SSH
sudo ufw allow 80/tcp          # HTTP
sudo ufw allow 443/tcp         # HTTPS

# Allow specific source IP (for bastion or admin access)
sudo ufw allow from 10.0.0.5 to any port 22

# Deny a specific port
sudo ufw deny 8080/tcp

# Enable UFW
sudo ufw enable

# Check status and rules
sudo ufw status verbose
sudo ufw status numbered

# Delete a rule
sudo ufw delete 3              # Delete rule number 3
```

### Interview Connect: Firewall vs Security Group?

> | Feature | UFW / iptables (host firewall) | AWS Security Group / Azure NSG |
> |---|---|---|
> | Location | Runs on the OS of the individual server | Enforced at the hypervisor/network level — before traffic reaches the VM |
> | Management | Per-server configuration | Centrally managed via cloud console/Terraform |
> | Stateful | iptables can be stateful or stateless | Always stateful |
> | Scale | Manual per-server | Apply one SG to hundreds of instances |
> | Bypass risk | An attacker who compromises the OS can disable it | Cannot be disabled from within the VM |
> | Use together? | Yes — defense in depth |
>
> **Best practice (Defense in Depth)**: Use both. Security Groups/NSGs are your first line of defense at the network level. Host firewalls (UFW) add a second layer inside the VM. This way, even if a misconfigured SG accidentally opens a port, the host firewall still blocks it.
>
> In production at Air Canada, Security Groups are managed via Terraform (Infrastructure as Code), reviewed in pull requests, and audited by AWS Config rules — not changed manually in the console.

---

## Task 23 – Linux Error: Permission Denied

### Create and Fix Scenario

```bash
# Create scenario
sudo touch /etc/app-secret.conf
sudo chmod 600 /etc/app-secret.conf
sudo chown root:root /etc/app-secret.conf

# As normal user — reproduce the error
cat /etc/app-secret.conf
# Output: cat: /etc/app-secret.conf: Permission denied

# Diagnose
ls -la /etc/app-secret.conf
# -rw------- 1 root root 0 Apr 5 2026 /etc/app-secret.conf

# Fix options:
# Option 1: Change owner to your user
sudo chown dhruwang:dhruwang /etc/app-secret.conf

# Option 2: Add group read permission (if group membership is appropriate)
sudo chmod 640 /etc/app-secret.conf
sudo chgrp devops /etc/app-secret.conf
usermod -aG devops dhruwang   # Add user to group

# Option 3: Use sudo (if you have sudo rights)
sudo cat /etc/app-secret.conf
```

### Linux Permission Model

```
-  rw-  r--  r--
│   │    │    │
│   │    │    └── Others: read only
│   │    └─────── Group: read only
│   └──────────── Owner: read + write
└──────────────── File type (- = file, d = directory, l = symlink)

chmod 755 = rwxr-xr-x (owner: all, group: read+exec, others: read+exec)
chmod 644 = rw-r--r-- (owner: read+write, group: read, others: read) → typical for config files
chmod 600 = rw------- (owner: read+write only) → sensitive files (SSH keys, secrets)
chmod 700 = rwx------ (owner: all, no one else) → private scripts
```

### Interview Connect: Why permission issues occur?

> Permission errors happen when processes or users try to access files/directories they don't own and don't have explicit read/write/execute grants. Common causes in production:
>
> - **Services running as wrong user**: A script runs as `ubuntu` but the file is owned by `www-data` (nginx user) — fix by running as the correct user or adjusting group permissions
> - **Automated deployments**: A CI/CD pipeline deploys files owned by the `jenkins` user, but the app runs as `appuser` — fix by using `chown` post-deploy or running both as the same service account
> - **Overly restrictive file permissions**: SSH private keys must be `600`, not `644` — SSH itself refuses to use keys that are group/world-readable
> - **Directory execute bit**: You need execute (`x`) on a directory to traverse it (enter it), even if you have read on the files inside. A directory with `644` means you can see the names but can't `cd` into it.
> - **umask**: The default umask (`022`) determines what permissions new files get. If a deployment script creates files with `umask 000`, files may be world-writable — a security risk.

---

## Task 24 – Linux Error: Resource Issue

### Scenario: Disk Full

```bash
# Diagnose
df -h                                   # Find full filesystem
du -h --max-depth=2 /var | sort -rh | head 20   # Find culprit

# Common fixes:

# 1. Clean Docker junk (very common in CI/CD servers)
docker system prune -af --volumes

# 2. Rotate and compress old logs manually
find /var/log -name "*.log" -mtime +7 -exec gzip {} \;
find /var/log -name "*.gz" -mtime +30 -delete

# 3. Clear journal logs
journalctl --vacuum-size=200M
journalctl --vacuum-time=7d

# 4. Truncate an actively-written log without breaking the file handle
> /var/log/app/access.log    # Truncate to 0 bytes (safe)

# 5. Find and remove core dumps
find / -name "core" -type f -size +100M -delete 2>/dev/null

# Verify after cleanup
df -h
```

### Interview Connect: How do you approach production issues?

> I follow a structured **DETECT → DIAGNOSE → REMEDIATE → PREVENT** framework:
>
> **DETECT**: Alert fires (CloudWatch alarm at 85% disk usage), or a service throws `No space left on device`. Don't wait until 100%.
>
> **DIAGNOSE**: Don't guess — measure. `df -h` → `du` → `find`. Identify the largest files or directories. Check recent changes: new deployments, log verbosity increase, Docker image builds.
>
> **REMEDIATE**: Apply the minimum change needed to restore service. Avoid drastic actions under pressure. Communicate to the team what you're doing.
>
> **PREVENT**:
> - Configure `logrotate` for all application logs
> - Set Docker daemon log limits in `/etc/docker/daemon.json`
> - Add disk usage CloudWatch alarm at 75% (warning) and 90% (critical)
> - Regularly prune Docker artifacts in a cron job
> - Use larger EBS volumes or auto-scaling storage (EFS) for workloads with unpredictable log growth
>
> Document everything in a post-incident review — not to blame, but to prevent recurrence.

---

## Task 25 – Shell Scripting & Automation

### Production-Grade Shell Script: Website & Service Monitor

```bash
#!/bin/bash
# =============================================================================
# Script   : service-monitor.sh
# Purpose  : Monitor website availability and service health
# Usage    : ./service-monitor.sh
# Schedule : */5 * * * * /opt/scripts/service-monitor.sh
# Author   : Dhruwang | Air Canada Infrastructure Team
# =============================================================================

set -euo pipefail   # Exit on error, unset variable, pipe failure
IFS=$'\n\t'         # Safe word splitting

# --- Configuration ---
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly LOG_FILE="/var/log/service-monitor.log"
readonly TIMESTAMP="$(date '+%Y-%m-%d %H:%M:%S')"
readonly ALERT_EMAIL="infra-alerts@aircanada.com"

# Services to monitor: "name|url|expected_http_code"
SERVICES=(
  "AirCanada-Homepage|https://aircanada.com|200"
  "Booking-API|https://api.aircanada.com/health|200"
  "Internal-Jenkins|http://jenkins.internal:8080/login|200"
)

# Local services (systemd)
SYSTEM_SERVICES=("nginx" "docker" "ssh")

# --- Logging ---
log() {
  local level="$1"
  shift
  echo "[$TIMESTAMP] [$level] $*" | tee -a "$LOG_FILE"
}

# --- HTTP Check ---
check_http() {
  local name="$1"
  local url="$2"
  local expected_code="$3"

  local http_code
  http_code=$(curl -o /dev/null -s -w "%{http_code}" \
    --connect-timeout 10 --max-time 30 "$url" 2>/dev/null || echo "000")

  if [[ "$http_code" == "$expected_code" ]]; then
    log "INFO" "OK     | $name | HTTP $http_code | $url"
    return 0
  else
    log "ERROR" "FAIL   | $name | HTTP $http_code (expected $expected_code) | $url"
    send_alert "$name" "$url" "$http_code" "$expected_code"
    return 1
  fi
}

# --- Service Check ---
check_service() {
  local service="$1"

  if systemctl is-active --quiet "$service"; then
    log "INFO" "OK     | systemd:$service | ACTIVE"
    return 0
  else
    log "ERROR" "FAIL   | systemd:$service | INACTIVE"
    send_alert "$service" "systemd" "INACTIVE" "ACTIVE"

    # Attempt auto-restart
    log "WARN"  "Attempting restart of $service..."
    if systemctl restart "$service"; then
      log "INFO" "RECOVERED | $service restarted successfully"
    else
      log "ERROR" "CRITICAL  | $service restart failed — manual intervention required"
    fi
    return 1
  fi
}

# --- Alert ---
send_alert() {
  local name="$1"
  local url="$2"
  local actual="$3"
  local expected="$4"

  local message="ALERT: $name is DOWN
URL/Service : $url
Expected    : $expected
Actual      : $actual
Timestamp   : $TIMESTAMP
Host        : $(hostname)"

  # Email alert (requires mail/sendmail configured)
  echo "$message" | mail -s "[ALERT] $name is DOWN on $(hostname)" "$ALERT_EMAIL" 2>/dev/null || true

  # Also log to syslog for central aggregation
  logger -t service-monitor -p user.crit "ALERT: $name DOWN | expected=$expected actual=$actual"
}

# --- Main ---
main() {
  log "INFO" "=== Monitor run started ==="

  local failed=0

  # Check HTTP endpoints
  for entry in "${SERVICES[@]}"; do
    IFS='|' read -r name url code <<< "$entry"
    check_http "$name" "$url" "$code" || ((failed++))
  done

  # Check system services
  for service in "${SYSTEM_SERVICES[@]}"; do
    check_service "$service" || ((failed++))
  done

  if [[ "$failed" -eq 0 ]]; then
    log "INFO" "=== All checks passed ==="
  else
    log "ERROR" "=== $failed check(s) failed ==="
  fi

  return "$failed"
}

main "$@"
```

### Cron Job Setup

```bash
# Make script executable
chmod +x /opt/scripts/service-monitor.sh

# Create log file with correct permissions
sudo touch /var/log/service-monitor.log
sudo chown ubuntu:ubuntu /var/log/service-monitor.log

# Edit crontab
crontab -e
```

```cron
# Crontab entries
# Run monitor every 5 minutes
*/5 * * * * /opt/scripts/service-monitor.sh >> /var/log/service-monitor.log 2>&1

# Daily disk usage report at 7 AM
0 7 * * * df -h >> /var/log/disk-report.log 2>&1

# Weekly cleanup of old logs every Sunday at 2 AM
0 2 * * 0 find /var/log -name "*.log" -mtime +30 -exec gzip {} \;
```

```bash
# Verify cron jobs
crontab -l

# Test script manually first
bash -x /opt/scripts/service-monitor.sh   # -x enables trace mode for debugging
```

### Cron Expression Reference

```
*  *  *  *  *  command
│  │  │  │  │
│  │  │  │  └── Day of week (0=Sun, 6=Sat)
│  │  │  └───── Month (1-12)
│  │  └──────── Day of month (1-31)
│  └─────────── Hour (0-23)
└────────────── Minute (0-59)

Examples:
*/5 * * * *        → Every 5 minutes
0 */6 * * *        → Every 6 hours
0 2 * * *          → Daily at 2 AM
0 2 * * 0          → Every Sunday at 2 AM
0 0 1 * *          → First day of every month at midnight
```

### Interview Connect: Why automation is critical in DevOps?

> Manual tasks don't scale. If you have 5 servers, you can manually SSH and check services. With 500 servers, you need automation.
>
> Shell scripting and cron automation:
> - **Eliminate human error**: Scripts run the same way every time. Humans make typos at 2 AM during incidents.
> - **Enforce consistency**: The same commands run on every server, not "mostly the same."
> - **Enable faster response**: An automated monitor that detects and restarts a failed service at 3 AM means no PagerDuty escalation, no incident, no SLO breach.
> - **Free up engineers for high-value work**: Toil (repetitive manual work) is the enemy of SRE teams. Every automated task reduces toil.
> - **Auditability**: Scripts are version-controlled in Git — you know exactly what ran, when, and who changed it.
>
> At Air Canada's scale, shell scripts are the foundation, but they're complemented by Ansible for configuration management, Terraform for infrastructure, and Python for complex automation logic — which are covered in later modules.

---

## Summary: Key Linux Interview Topics for Air Canada

| Topic | What Interviewers Test |
|---|---|
| Permissions | `chmod`, `chown`, octal notation, understanding why a service fails to start |
| Process management | Finding runaway processes, graceful vs forceful kill, `systemctl` |
| Disk management | Debugging disk full, inodes, `df` vs `du`, fstab |
| Networking | `ss`, `ip`, DNS resolution, TCP connection debugging |
| SSH | Key-based auth, bastion host pattern, SSH config file, port forwarding |
| Logging | `tail -f`, `journalctl`, log locations, real-time filtering |
| Shell scripting | Variables, conditions, loops, error handling (`set -euo pipefail`), cron |
| Troubleshooting | Structured methodology — don't guess, measure first |

### Critical Commands to Know Cold

```bash
# The commands every Cloud Infrastructure Analyst must know instantly:
ss -tulnp                    # What's listening on what port
df -h && du -sh /*           # Disk usage diagnosis
tail -f /var/log/syslog      # Live log monitoring
ps aux --sort=-%cpu | head   # Top CPU consumers
systemctl status <service>   # Service health
journalctl -u <service> -f   # Service logs
ip addr show                 # Network interfaces
curl -o /dev/null -s -w "%{http_code}" <url>  # HTTP health check
chmod / chown / ls -la       # Permissions
kill / kill -9               # Process control
```
