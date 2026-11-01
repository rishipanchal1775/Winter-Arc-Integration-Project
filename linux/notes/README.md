
# Day 31/90 — Linux + Networking Revision Lab

**Challenge:** DevOps Winter Arc  
**Day:** 31  
**Environment:** WSL Ubuntu  
**Web Server:** Nginx  
**HTTP Port:** 80  
**Application URL:** http://localhost

---

## 1. Lab Objectives

The objective of this lab was to revise Linux administration and networking fundamentals by performing practical tasks in WSL Ubuntu.

### Tasks performed

1. Linux environment and system checks.
2. User, group, and permission inspection.
3. CPU, memory, and disk monitoring.
4. Running an application as a Linux service.
5. Process and service management.
6. Checking listening ports.
7. Nginx and HTTP networking verification.
8. Safe troubleshooting exercise.
9. Inspecting service logs.
10. Interview preparation.

---

# 2. Task A — Linux Environment and System Checks

## 2.1 Check Linux distribution

```bash
cat /etc/os-release
```

Displays the Linux distribution and version.

## 2.2 Check current user

```bash
whoami
```

Displays the currently logged-in username.

```bash
id
```

Displays user ID, group ID, and supplementary groups.

## 2.3 Check hostname

```bash
hostname
```

Displays the system hostname.

```bash
hostnamectl
```

Displays hostname and system information when supported by the environment.

## 2.4 Check Linux kernel

```bash
uname -r
```

Displays the kernel version.

```bash
uname -a
```

Displays complete kernel and system information.

## 2.5 Check system uptime

```bash
uptime
```

Displays how long the system has been running and system load averages.

## 2.6 Check CPU information

```bash
lscpu
```

Displays CPU architecture, processors, cores, and other CPU information.

## 2.7 Check memory usage

```bash
free -h
```

Displays total, used, free, and available memory in human-readable format.

## 2.8 Check disk space

```bash
df -h
```

Displays filesystem disk usage.

```bash
du -sh ~
```

Displays the total size of the home directory.

## 2.9 Create the Day 31 project directory

```bash
mkdir -p ~/DevOps-Winter-Arc/Day31-Linux-Networking
```

Navigate into the directory:

```bash
cd ~/DevOps-Winter-Arc/Day31-Linux-Networking
```

Check the current directory:

```bash
pwd
```

## 2.10 Check directory ownership and permissions

```bash
ls -la
```

Displays files, directories, ownership, and permissions.

```bash
stat .
```

Displays detailed information about the current directory.

Create a test directory and file:

```bash
mkdir -p notes
touch notes/permission-test.txt
```

Check permissions:

```bash
ls -l notes/
```

Change file permissions:

```bash
chmod 644 notes/permission-test.txt
```

Verify:

```bash
ls -l notes/
```

### Linux permission reference

| Permission | Meaning |
|---|---|
| r | Read |
| w | Write |
| x | Execute |
| - | No permission |

### Permission 644

- Owner: Read and Write
- Group: Read
- Others: Read

### Permission 755

- Owner: Read, Write, Execute
- Group: Read, Execute
- Others: Read, Execute

---

# 3. Task B — Run the Application as a Linux Service

For this lab, Nginx was used as the temporary web application.

## 3.1 Update Ubuntu packages

```bash
sudo apt update
```

Refreshes the Ubuntu package information.

## 3.2 Install Nginx and curl

```bash
sudo apt install -y nginx curl
```

- Nginx: Web server.
- curl: Command-line HTTP client.

## 3.3 Check systemd availability

```bash
ps -p 1 -o comm=
```

If the output is `systemd`, systemd is running as the init system.

## 3.4 Start Nginx

If systemd is available:

```bash
sudo systemctl start nginx
```

Alternative when systemd is unavailable:

```bash
sudo service nginx start
```

## 3.5 Enable Nginx

```bash
sudo systemctl enable nginx
```

Configures Nginx to start automatically when the system boots under systemd.

## 3.6 Check Nginx status

```bash
sudo systemctl status nginx --no-pager
```

Alternative:

```bash
sudo service nginx status
```

Expected state: active (running).

## 3.7 Check running Nginx processes

```bash
ps aux | grep nginx
```

Alternative:

```bash
pgrep -a nginx
```

Displays running Nginx processes and process IDs.

## 3.8 Check listening ports

```bash
sudo ss -lntp
```

Filter port 80:

```bash
sudo ss -lntp | grep ':80'
```

### ss command options

| Option | Meaning |
|---|---|
| -l | Listening sockets |
| -n | Numeric addresses and ports |
| -t | TCP sockets |
| -p | Process information |

## 3.9 Inspect Nginx logs

View recent service logs:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

View logs from the last 30 minutes:

```bash
sudo journalctl -u nginx --since "30 minutes ago"
```

View access logs:

```bash
sudo tail -n 50 /var/log/nginx/access.log
```

View error logs:

```bash
sudo tail -n 50 /var/log/nginx/error.log
```

---

# 4. Task C — Nginx / HTTP Networking Check

## 4.1 Create a custom HTML page

```bash
sudo tee /var/www/html/index.html > /dev/null <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>DevOps Winter Arc - Day 31</title>
</head>
<body>
    <h1>Day 31 - Linux and Networking Lab</h1>
    <h2>DevOps Winter Arc</h2>
    <p>Application running successfully on WSL Ubuntu.</p>
    <p>Web Server: Nginx</p>
    <p>HTTP Port: 80</p>
</body>
</html>
EOF
```

The HTML file is stored in Nginx's default document root.

## 4.2 Verify the HTML page

```bash
cat /var/www/html/index.html
```

Displays the page contents.

## 4.3 Test Nginx configuration

```bash
sudo nginx -t
```

Checks Nginx configuration syntax.

## 4.4 Restart Nginx

```bash
sudo systemctl restart nginx
```

Alternative:

```bash
sudo service nginx restart
```

## 4.5 Test HTTP response

```bash
curl -I http://127.0.0.1
```

Expected response:

```text
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html
```

HTTP 200 OK means the server successfully processed the request.

## 4.6 Display webpage content using curl

```bash
curl -s http://127.0.0.1
```

Display only the first few lines:

```bash
curl -s http://127.0.0.1 | head
```

## 4.7 Access the application from Windows

Open Chrome or Edge and visit:

```text
http://localhost
```

WSL localhost forwarding allows Windows to access the Nginx service running inside WSL.

---

# 5. Task D — Safe Troubleshooting Drill

## Objective

Intentionally stop Nginx, diagnose why the application is unavailable, restore the service, and verify recovery.

## 5.1 Verify initial application health

```bash
curl -I http://127.0.0.1
```

Expected: HTTP 200 OK.

Check service:

```bash
sudo systemctl is-active nginx
```

Expected:

```text
active
```

## 5.2 Introduce a controlled failure

Stop Nginx:

```bash
sudo systemctl stop nginx
```

Alternative:

```bash
sudo service nginx stop
```

## 5.3 Observe the failure

```bash
curl -I --max-time 5 http://127.0.0.1
```

Expected: connection refused or connection failure.

Check service status:

```bash
sudo systemctl status nginx --no-pager
```

Check port 80:

```bash
sudo ss -lntp | grep ':80'
```

Nginx should no longer be listening on port 80.

## 5.4 Identify the root cause

| Test | Result |
|---|---|
| HTTP request | Connection failure |
| Nginx status | Inactive / stopped |
| Port 80 | No Nginx listener |

**Root cause:** The Nginx service was deliberately stopped.

## 5.5 Fix the issue

Start Nginx again:

```bash
sudo systemctl start nginx
```

Alternative:

```bash
sudo service nginx start
```

## 5.6 Verify recovery

```bash
sudo systemctl status nginx --no-pager
```

```bash
sudo ss -lntp | grep ':80'
```

```bash
curl -I http://127.0.0.1
```

Expected: Nginx active, port 80 listening, and HTTP 200 OK.

Open the browser again:

```text
http://localhost
```

### Troubleshooting methodology

1. Symptom
2. Evidence
3. Root cause
4. Fix
5. Verification

---

# 6. Troubleshooting Reference

## 6.1 Connection refused

Possible causes:

- Nginx is stopped.
- Application is not running.
- Wrong port is being used.
- No process is listening on the requested port.

Commands:

```bash
sudo systemctl status nginx
sudo ss -lntp
pgrep -a nginx
sudo systemctl start nginx
curl -I http://127.0.0.1
```

## 6.2 HTTP 403 Forbidden

Possible causes:

- Incorrect file permissions.
- Incorrect directory permissions.
- Nginx cannot read the requested file.

Inspect permissions:

```bash
ls -ld /var/www/html
ls -l /var/www/html/
```

Inspect error logs:

```bash
sudo tail -n 30 /var/log/nginx/error.log
```

Use appropriate permissions:

```bash
sudo chmod 755 /var/www/html
sudo chmod 644 /var/www/html/index.html
```

Avoid using `chmod 777` as a general solution.

## 6.3 Nginx fails to start

Check configuration:

```bash
sudo nginx -t
```

Check service:

```bash
sudo systemctl status nginx --no-pager
```

Check logs:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

Check whether port 80 is already occupied:

```bash
sudo ss -lntp | grep ':80'
```

## 6.4 Works locally but not remotely

Check WSL IP:

```bash
hostname -I
```

Check listening interfaces:

```bash
sudo ss -lntp
```

Check:

- Application listening address.
- WSL networking.
- Windows firewall.
- Linux firewall.
- Network connectivity.

## 6.5 Disk full

Check disk space:

```bash
df -h
```

Check home directory size:

```bash
du -sh ~
```

Find large directories:

```bash
du -h --max-depth=1 ~ 2>/dev/null | sort -h
```

Check Nginx log directory:

```bash
sudo du -sh /var/log/nginx
```

---

# 7. Linux Commands Quick Reference

| Purpose | Command |
|---|---|
| Current user | `whoami` |
| User and groups | `id` |
| Hostname | `hostname` |
| Kernel version | `uname -r` |
| System information | `uname -a` |
| Uptime | `uptime` |
| CPU information | `lscpu` |
| Memory usage | `free -h` |
| Disk usage | `df -h` |
| Directory listing | `ls -la` |
| File permissions | `stat filename` |
| Change permissions | `chmod 644 filename` |
| Running processes | `ps aux` |
| Find process | `pgrep -a nginx` |
| Start service | `sudo systemctl start nginx` |
| Stop service | `sudo systemctl stop nginx` |
| Restart service | `sudo systemctl restart nginx` |
| Service status | `sudo systemctl status nginx` |
| Listening ports | `sudo ss -lntp` |
| Test HTTP | `curl -I http://127.0.0.1` |
| Test Nginx config | `sudo nginx -t` |
| Service logs | `sudo journalctl -u nginx` |
| Access logs | `sudo tail -n 50 /var/log/nginx/access.log` |
| Error logs | `sudo tail -n 50 /var/log/nginx/error.log` |

---

# 8. Interview Questions and Answers

## Q1. What does systemctl do?

**Answer:**

`systemctl` is a command-line utility used to manage and inspect services controlled by systemd.

It can start, stop, restart, enable, disable, and check the status of services.

Examples:

```bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl status nginx
```

## Q2. How do you find a process listening on a port?

**Answer:**

Use the `ss` command to inspect listening network sockets and identify the associated process.

Example:

```bash
sudo ss -lntp
```

To check port 80:

```bash
sudo ss -lntp | grep ':80'
```

## Q3. Where can you inspect systemd service logs?

**Answer:**

Use `journalctl` to inspect logs for systemd services.

Example:

```bash
sudo journalctl -u nginx
```

To display the latest 50 entries:

```bash
sudo journalctl -u nginx -n 50 --no-pager
```

## Q4. What is the difference between localhost and host IP?

**Answer:**

- `localhost` refers to the local machine, usually through the loopback interface.
- A host IP identifies a network interface and can be used to access a service over a network if the service is listening on a reachable interface and firewall rules permit it.

In this WSL lab:

```text
127.0.0.1 = WSL loopback address
localhost = local machine hostname
```

## Additional Practice Questions

### Q5. What is Nginx?

Nginx is a web server that can also act as a reverse proxy, load balancer, and HTTP cache.

### Q6. What is the difference between a process and a service?

A process is a running instance of a program. A service is a program managed to provide a system or application function, often running in the background.

### Q7. What does HTTP 200 mean?

HTTP 200 OK means the server successfully processed the request.

### Q8. What does HTTP 403 mean?

HTTP 403 Forbidden means the server understood the request but refuses to authorize access to the requested resource.

### Q9. What does HTTP 404 mean?

HTTP 404 Not Found means the requested resource could not be found.

### Q10. What is the purpose of curl?

`curl` is a command-line tool used to transfer data and test network services, including HTTP and HTTPS endpoints.

### Q11. What is the purpose of ss?

`ss` displays socket information, including listening ports, network connections, and associated processes.

### Q12. What is the difference between chmod 644 and chmod 755?

- `644`: Owner can read and write; group and others can read.
- `755`: Owner can read, write, and execute; group and others can read and execute.

### Q13. Why do we check service logs?

Logs help identify errors, failures, startup problems, and other events that explain service behavior.

### Q14. What is a listening port?

A listening port is a network port on which a process waits for incoming connections.

### Q15. How do you troubleshoot an application that is not responding?

1. Check application/service status.
2. Check whether the process is running.
3. Check listening ports.
4. Test connectivity using curl.
5. Inspect application and service logs.
6. Identify the root cause.
7. Apply a fix.
8. Verify recovery.

---

#Key Learnings

During Day 31, I practiced:

- Linux system information and resource monitoring.
- User and group identification.
- Linux file permissions and ownership.
- Nginx installation and service management.
- Process inspection.
- Listening port identification.
- HTTP testing using curl.
- Nginx configuration validation.
- Service log inspection using journalctl.
- Troubleshooting a stopped service.
- Restoring service availability and verifying recovery.

**Day 31 Learning Principle:**

Learn → Build → Break → Fix → Explain.
