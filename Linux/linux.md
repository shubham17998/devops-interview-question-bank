# Linux / Shell Scripting Interview Questions

## Shell Scripting Basics
1. What is Shebang?
2. What is IFS (Internal Field Separator)?
3. How do you delete files older than 15 days using shell scripting?
4. How do you search for files containing a specific string within a directory, and print only the matching file names?
5. How would you validate command-line arguments in a shell script?
6. Write a shell script to find the largest and smallest elements of an integer array.

## User & Package Management
7. How do you create a new user in Linux, and where is the default home directory created?
8. How do you create a new user and add them to the sudo group?
9. How do you check the list of installed packages?
10. What command do you use to check the kernel version?

## File System
11. How do you find all files smaller than 5MB?
12. What is the difference between a hard link and a soft (symbolic) link?

## System Diagnostics
13. Commands to check running processes?
14. How do you check if a specific process is running, and how do you stop it?
15. Commands to check memory usage?
16. Commands to check disk usage?
17. How do you check if an application is listening on a port?
18. How do you check application updates?
19. How do you check system load?
20. What is nohup?
21. What is cron?
22. What is OOMKilled?

## Troubleshooting
23. How do you troubleshoot high CPU on Linux?
24. How do you troubleshoot a disk full issue?
25. How do you optimize an on-prem VM?
26. How do you debug a server where the application is not working?
27. How do you verify an application using curl?
28. How do you test connectivity using ping?

## Networking
29. How do you troubleshoot communication failures with an external server?
30. How do you analyze 500 errors without a log aggregator?

## GCP / GKE
31. What is Compute Engine?
32. How would you set a password for a user on a Compute Engine instance?
33. What is GCP VPC?
34. Difference between GCP VPC and AWS VPC?
35. What are Firewall Rules in GCP?
36. What is a Network Endpoint Group (NEG) in GCP?
37. What is GKE?
38. Difference between EKS and GKE?
39. How do you secure GKE?
40. How does GKE Autoscaling work?
41. How many IP ranges are required for a VPC-native GKE cluster, and what is each range used for?

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
