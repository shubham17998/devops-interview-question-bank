# Linux / Shell Scripting Interview Questions

## Shell Scripting Basics
Q: What is Shebang?
Ans: The `#!` line at the top of a script (e.g., `#!/usr/bin/env bash`) telling the OS which interpreter to use to execute the file when run directly (e.g., `./script.sh`).

Q: What is IFS (Internal Field Separator)?
Ans: A shell variable defining the character(s) used to split a string into words/fields during word-splitting (default: space, tab, newline) — commonly changed temporarily (e.g., `IFS=','`) when parsing delimited data like CSV lines.

Q: How do you delete files older than 15 days using shell scripting?
Ans: `find /path/to/dir -type f -mtime +15 -delete` (or pipe to `xargs rm` for more control: `find /path -type f -mtime +15 -print0 | xargs -0 rm -f`).

Q: How do you search for files containing a specific string within a directory, and print only the matching file names?
Ans: `grep -rl "search_string" /path/to/dir` (`-r` recursive, `-l` print only filenames of matches, not the matching lines themselves).

Q: How would you validate command-line arguments in a shell script?
Ans: Check `$#` for the expected argument count, and validate each `$1`, `$2`, etc. against expected format/type, exiting with a usage message on failure:
```bash
if [ "$#" -ne 2 ]; then
    echo "Usage: $0 <arg1> <arg2>"
    exit 1
fi
```

Q: Write a shell script to find the largest and smallest elements of an integer array.
Ans:
```bash
#!/usr/bin/env bash
arr=(4 8 15 16 23 42)
max=${arr[0]}
min=${arr[0]}
for n in "${arr[@]}"; do
    (( n > max )) && max=$n
    (( n < min )) && min=$n
done
echo "Max: $max, Min: $min"
```

## User & Package Management
Q: How do you create a new user in Linux, and where is the default home directory created?
Ans: `useradd -m username` (`-m` creates the home directory) — the default home directory is created at `/home/username`.

Q: How do you create a new user and add them to the sudo group?
Ans: `useradd -m -G sudo username` (Debian/Ubuntu) or `useradd -m -G wheel username` (RHEL/CentOS), then set a password with `passwd username`.

Q: How do you check the list of installed packages?
Ans: Debian/Ubuntu: `dpkg -l` or `apt list --installed`. RHEL/CentOS: `rpm -qa` or `yum list installed`/`dnf list installed`.

Q: What command do you use to check the kernel version?
Ans: `uname -r` (add `-a` for full kernel/system info).

## File System
Q: How do you find all files smaller than 5MB?
Ans: `find /path -type f -size -5M`.

Q: What is the difference between a hard link and a soft (symbolic) link?
Ans: A **hard link** is another directory entry pointing to the same inode (same underlying data) — deleting the original file doesn't remove the data as long as a hard link remains; can't span filesystems/reference directories. A **soft (symbolic) link** is a separate file that just stores a path to the target — breaks ("dangling link") if the target is deleted/moved, but can span filesystems and link to directories.

## System Diagnostics
Q: Commands to check running processes?
Ans: `ps aux` (snapshot of all processes), `top`/`htop` (live, interactive view), `pgrep <name>` (find PIDs by name).

Q: How do you check if a specific process is running, and how do you stop it?
Ans: Check: `pgrep -f <process_name>` or `ps aux | grep <process_name>`. Stop: `kill <pid>` (graceful, SIGTERM) or `kill -9 <pid>` (forceful, SIGKILL) if it doesn't respond; `pkill <name>` to kill by name directly.

Q: Commands to check memory usage?
Ans: `free -h` (human-readable summary), `/proc/meminfo` (detailed breakdown), `top`/`htop` (per-process + system-wide, live).

Q: Commands to check disk usage?
Ans: `df -h` (filesystem-level free/used space), `du -sh <dir>` (size of a specific directory/file, summarized), `du -sh * | sort -rh` (find the largest items in a directory).

Q: How do you check if an application is listening on a port?
Ans: `ss -tulnp | grep <port>` (modern), or the older `netstat -tulnp | grep <port>`; `lsof -i :<port>` also works and shows the owning process.

Q: How do you check application updates?
Ans: Depends on how it was installed: package manager (`apt list --upgradable`, `yum check-update`), or application-specific version check commands — generally, compare the currently installed version against the latest available from its source/repo.

Q: How do you check system load?
Ans: `uptime` (shows load averages for 1/5/15 minutes) or `top`/`htop` (load average shown in the header, alongside per-core CPU usage).

Q: What is nohup?
Ans: A command that runs another command immune to hangups (SIGHUP) — so it keeps running even after the terminal/SSH session that launched it closes; typically combined with `&` to also background it: `nohup ./script.sh &`.

Q: What is cron?
Ans: A time-based job scheduler daemon on Unix-like systems that runs commands/scripts automatically at specified times/intervals, defined in crontab files (`crontab -e`) using the standard 5-field cron syntax (minute hour day month weekday).

Q: What is OOMKilled?
Ans: A status indicating a process was terminated by the Linux kernel's Out-Of-Memory killer, which forcibly kills processes (choosing by an "OOM score") when the system runs critically low on memory to prevent a total system freeze — in Kubernetes, a container hitting its memory limit gets OOMKilled and shows exit code 137.

## Troubleshooting
Q: How do you troubleshoot high CPU on Linux?
Ans: `top`/`htop` to identify the top CPU-consuming process, `ps -eo pid,%cpu,cmd --sort=-%cpu | head` for a snapshot; drill into a specific process with `strace -p <pid>` (syscalls) or `perf top` (kernel-level profiling) if the cause isn't obvious from the process name/command alone; check for runaway loops, excessive logging, or a stuck/looping thread.

Q: How do you troubleshoot a disk full issue?
Ans: `df -h` to identify which filesystem/mount is full, then `du -sh /path/* | sort -rh | head` recursively down into that mount to find the largest directories/files; check for common culprits — oversized log files, old core dumps, Docker image/build cache, or a runaway process writing endlessly — then clean up or rotate/truncate as appropriate.

Q: How do you optimize an on-prem VM?
Ans: Right-size CPU/memory allocation based on actual usage (avoid over/under-provisioning), ensure disk I/O isn't the bottleneck (check with `iostat`), tune kernel parameters relevant to the workload (e.g., network buffer sizes, file descriptor limits), remove unnecessary background services, and keep the OS/kernel patched for performance fixes.

Q: How do you debug a server where the application is not working?
Ans: Check if the process is actually running (`ps`/`systemctl status`), check application logs for errors, verify it's listening on the expected port (`ss -tulnp`), test connectivity locally (`curl localhost:<port>`), check resource exhaustion (CPU/memory/disk), verify configuration/environment variables, and check for recent changes (deploy, config change) that correlate with when it broke.

Q: How do you verify an application using curl?
Ans: `curl -v http://localhost:<port>/health` (or the relevant endpoint) — `-v` shows the full request/response including headers and status code, useful to confirm the app is actually responding correctly (not just that the port is open).

Q: How do you test connectivity using ping?
Ans: `ping <host>` sends ICMP echo requests and reports round-trip time/packet loss — confirms basic network reachability (note: doesn't confirm a specific service/port is actually working, and can be blocked by firewalls even when the host is otherwise reachable).

## Networking
Q: How do you troubleshoot communication failures with an external server?
Ans: Check basic reachability (`ping`, though it may be blocked), test the actual port/service (`curl`, `telnet host port`, or `nc -zv host port`), check DNS resolution (`dig`/`nslookup`), check local firewall/security group rules on both ends, trace the network path (`traceroute`/`mtr`) to see where it fails, and check for TLS/certificate issues if it's an HTTPS endpoint.

Q: How do you analyze 500 errors without a log aggregator?
Ans: SSH into the affected host/container directly and `tail -f`/`grep` the application and web server logs around the error timestamps, cross-reference with `dmesg`/OOM logs and resource usage (`top`, `free`) at that time, and check upstream/downstream dependency availability if the errors point to a timeout/connection failure.

## GCP / GKE
Q: What is Compute Engine?
Ans: GCP's Infrastructure-as-a-Service virtual machine offering — analogous to AWS EC2, letting you provision customizable VMs with a chosen OS, machine type, and disks.

Q: How would you set a password for a user on a Compute Engine instance?
Ans: SSH into the instance and use `sudo passwd username` (Linux), or for Windows instances, use the GCP Console's "Set Windows password" feature (which resets/generates a new password via the guest agent) since RDP requires a password rather than key-based auth.

Q: What is GCP VPC?
Ans: Google Cloud's virtual network construct — similar to AWS VPC, but **global** by default (a single VPC can span all regions with regional subnets), providing private networking, firewall rules, and routing for GCP resources.

Q: Difference between GCP VPC and AWS VPC?
Ans: A GCP VPC is **global** — one VPC can have subnets in multiple regions worldwide, with implicit global routing between them. An AWS VPC is **regional** — scoped to a single region, requiring VPC Peering/Transit Gateway to connect VPCs across regions.

Q: What are Firewall Rules in GCP?
Ans: Stateful, VPC-level rules (not tied to a subnet like AWS NACLs) that allow/deny traffic based on direction, IP ranges, protocols/ports, and target tags/service accounts — analogous to AWS Security Groups but applied at the VPC level with tag-based targeting.

Q: What is a Network Endpoint Group (NEG) in GCP?
Ans: A GCP construct representing a group of backend endpoints (VM IPs, serverless services, or container-native pods) that a load balancer can route to — used especially for container-native (GKE) load balancing, routing directly to pod IPs rather than through node-level NodePorts.

Q: What is GKE?
Ans: Google Kubernetes Engine — GCP's managed Kubernetes service, analogous to AWS EKS, handling control-plane management/upgrades/scaling for you.

Q: Difference between EKS and GKE?
Ans: Both are managed Kubernetes offerings. GKE (from Google, the original creators of Kubernetes) generally offers a more mature/integrated experience — Autopilot mode (fully managed nodes), faster new-version availability, and tighter native integration with GCP networking/IAM. EKS integrates deeply with the broader AWS ecosystem (IAM/IRSA, ALB Ingress Controller, Fargate) — the choice mostly comes down to which cloud ecosystem you're already standardized on.

Q: How do you secure GKE?
Ans: Use Workload Identity (GKE's equivalent of IRSA, binding Kubernetes service accounts to GCP IAM service accounts), enable private clusters (nodes with no public IP), apply Kubernetes RBAC and Network Policies, enable Binary Authorization (only run signed/verified images), keep node auto-upgrade enabled, and restrict the control plane's authorized networks.

Q: How does GKE Autoscaling work?
Ans: GKE supports Cluster Autoscaler (adds/removes nodes based on pending unschedulable pods and node utilization), Horizontal Pod Autoscaler (scales pod replica count based on metrics), Vertical Pod Autoscaler (adjusts pod resource requests/limits), and Autopilot mode (GCP fully manages node provisioning/scaling for you, billing per-pod resource usage instead of per-node).

Q: How many IP ranges are required for a VPC-native GKE cluster, and what is each range used for?
Ans: Three: the **node** IP range (primary subnet range, for the VMs themselves), the **pod** IP range (a secondary range, allocated in chunks to each node for its pods' IPs), and the **service** IP range (a secondary range for ClusterIP Service addresses) — VPC-native (alias IP-based) clusters allocate real, routable VPC IPs from these secondary ranges rather than using an overlay network.

## Hands-on Exercises

### Exercise 1: Navigation
**Objective:** Practice moving around the filesystem.
**Steps:**
1. Change directory to `/tmp`.
2. Move to the parent directory.
3. Change to your home directory.
4. Move to the parent directory, twice.
5. Where are you now? Verify with a command.
6. Change back to the last directory you visited.

**Solution:**
```
cd /tmp
cd ..
cd ~
cd ..
cd ..
pwd        # /
cd -
```

### Exercise 2: Create & Destroy
**Objective:** Practice basic file and directory manipulation.
**Steps:**
1. Create a file called `x`.
2. Create a directory called `content`.
3. Move `x` into `content`.
4. Create a file inside `content` called `y`.
5. Create the nested directory structure `content/dir1/dir2/dir3`.
6. Remove the `content` directory entirely.

**Solution:**
```
touch x
mkdir content
mv x content
touch content/y
mkdir -p content/dir1/dir2/dir3
rm -rf content
```

### Exercise 3: Argument Check
**Objective:** Practice conditional logic on script arguments.
**Task:** Write a script that checks if its first argument is the string "pizza". If so, print "with pineapple?"; otherwise print "I want pizza!".

**Solution:**
```bash
#!/usr/bin/env bash
[[ ${1} == "pizza" ]] && echo "with pineapple?" || echo "I want pizza!"
```

### Exercise 4: Factors
**Objective:** Practice arithmetic conditionals in shell scripts.
**Task:** Write a script that, given a number, prints "one factor" if 2 is a factor, appends "...actually two!" if 3 is also a factor, or prints the number itself if neither is a factor.

**Solution:**
```bash
#!/usr/bin/env bash
(( $1 % 2 )) || res="one factor"
(( $1 % 3 )) || res+="...actually two!"
echo ${res:-$1}
```

### Exercise 5: File Sizes
**Objective:** Practice looping over files and formatting output.
**Task:** Print the name and size of every file and directory in the current path, using at least one `for` loop.

**Solution:**
```bash
#!/usr/bin/env bash
for i in $(ls -S1); do
    echo "$i: $(du -sh "$i" | cut -f1)"
done
```

### Exercise 6: Is the Host Alive?
**Objective:** Practice scripting a basic health check with alerting.
**Task:** Write a script that determines whether a given host is up or down, and sends an alert if it's down.

**Solution:**
```bash
#!/usr/bin/env bash
SERVERIP=<IP Address>
NOTIFYEMAIL=test@example.com

ping -c 3 "$SERVERIP" > /dev/null 2>&1
if [ $? -ne 0 ]; then
    mailx -s "Server $SERVERIP is down" -t "$NOTIFYEMAIL" < /dev/null
fi
```
