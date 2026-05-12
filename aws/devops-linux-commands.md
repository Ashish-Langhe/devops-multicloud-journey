# ⚡ AWS DevOps Linux Command Arsenal
> **14-year battle-tested reference** — Every command a Senior AWS DevOps Engineer reaches for, daily.

[![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?style=flat-square&logo=amazon-aws)](https://aws.amazon.com)
[![Linux](https://img.shields.io/badge/Linux-Commands-FCC624?style=flat-square&logo=linux&logoColor=black)](https://linux.org)
[![Shell](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat-square&logo=gnu-bash)](https://www.gnu.org/software/bash/)
[![Maintained](https://img.shields.io/badge/Maintained-Yes-success?style=flat-square)]()

---

## 📋 Table of Contents

| # | Category | Commands |
|---|----------|----------|
| 1 | [🔐 Access & Permissions](#-access--permissions) | chmod, chown, sudo, visudo |
| 2 | [⚙️ Service Management](#️-service-management) | systemctl, service |
| 3 | [🔥 Process & PID Control](#-process--pid-control) | ps, pkill, kill, htop, nohup |
| 4 | [🌐 Network & Connectivity](#-network--connectivity) | netstat, ss, curl, dig, ping |
| 5 | [💾 File, Disk & Storage](#-file-disk--storage) | df, lsblk, mount, find, grep |
| 6 | [☁️ AWS CLI Operations](#️-aws-cli-operations) | s3, ec2, ssm, ecr, logs, rds |
| 7 | [🚨 Error & Recovery](#-error--recovery) | journalctl, dmesg, lsof, strace, fsck |
| 8 | [🐳 Docker & Containers](#-docker--containers) | docker ps, exec, logs, prune |
| 9 | [📦 Package Management](#-package-management) | yum, apt, pip |
| 10 | [🔒 Security & Hardening](#-security--hardening) | iptables, fail2ban, auditd |

---

## 🔐 Access & Permissions

> Foundation of everything. Wrong permissions = broken deployments, failed SSH, security breaches.

### Switch to Root

```bash
sudo su -                        # Full root shell (loads root environment)
sudo -i                          # Interactive root shell (preferred on Amazon Linux / Ubuntu)
sudo -l                          # List what sudo commands current user can run
```

> ⚠️ **Use with caution** — Always prefer `sudo <cmd>` over full root shell for auditability.

---

### SSH Key Permissions (EC2 Must-Have)

```bash
chmod 400 ~/.ssh/mykey.pem       # Strict read-only — required for SSH to EC2
chmod 600 ~/.ssh/authorized_keys # User-only read/write for authorized_keys
chmod 700 ~/.ssh/                # Secure .ssh directory

# SSH into EC2
ssh -i ~/.ssh/mykey.pem ec2-user@<EC2-PUBLIC-IP>
ssh -i ~/.ssh/mykey.pem ubuntu@<EC2-PUBLIC-IP>    # Ubuntu AMIs
```

> 💡 Without `chmod 400`, SSH throws: **"UNPROTECTED PRIVATE KEY FILE"** and refuses to connect.

---

### File & Directory Permissions

```bash
chmod 755 /opt/app/start.sh      # Owner: rwx, Group: r-x, Others: r-x (scripts, dirs)
chmod 644 /etc/nginx/nginx.conf  # Owner: rw-, Group: r--, Others: r-- (config files)
chmod -R 755 /var/www/html       # Recursive permission change
chmod +x deploy.sh               # Add execute bit to script

# Numeric quick reference:
# 400 = r--------  (PEM keys)
# 600 = rw-------  (private files)
# 644 = rw-r--r--  (configs)
# 755 = rwxr-xr-x  (scripts, dirs)
# 777 = rwxrwxrwx  (NEVER use in production)
```

---

### Ownership Control

```bash
chown ec2-user:ec2-user file.txt          # Change owner and group
chown -R ec2-user:nginx /var/www/html     # Recursive ownership change
chown root:root /etc/cron.d/myjob         # Set root ownership for cron
ls -la /var/www/html                      # Verify ownership
```

---

### Sudoers Management

```bash
visudo                                    # Safely edit /etc/sudoers (validates syntax)
cat /etc/sudoers.d/deploy                 # Check custom sudoer files

# Inside visudo — grant passwordless sudo:
deployer ALL=(ALL) NOPASSWD: ALL

# Grant only specific commands:
deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```

> 🔑 Never edit `/etc/sudoers` directly — use `visudo` to prevent lockouts.

---

## ⚙️ Service Management

> The **systemctl** lifecycle — master this and you control every daemon on your EC2.

### Start / Stop / Restart

```bash
sudo systemctl start nginx               # Start service
sudo systemctl stop nginx                # Stop service
sudo systemctl restart nginx             # Stop + Start (brief downtime)
sudo systemctl reload nginx              # Reload config (no downtime for nginx)
sudo systemctl try-restart nginx         # Restart only if already running
```

---

### Enable / Disable (Auto-start on Boot)

```bash
sudo systemctl enable nginx              # Auto-start on reboot
sudo systemctl disable nginx             # Remove from auto-start
sudo systemctl enable --now nginx        # Enable + Start immediately (one command)
sudo systemctl disable --now nginx       # Disable + Stop immediately
```

> ⚠️ **Critical EC2 gotcha** — Services do NOT auto-start after reboot unless `enabled`. Always run `enable` after installing a service.

---

### Status & Inspection

```bash
sudo systemctl status nginx              # Service status, PID, recent logs
sudo systemctl is-active nginx           # Returns "active" or "inactive"
sudo systemctl is-enabled nginx          # Returns "enabled" or "disabled"
sudo systemctl list-units --type=service # All loaded services
sudo systemctl list-units --failed       # Only failed services
```

---

### After Editing Unit Files

```bash
# Edit service file:
sudo vim /etc/systemd/system/myapp.service

# Always reload after editing:
sudo systemctl daemon-reload
sudo systemctl restart myapp

# View the unit file:
sudo systemctl cat nginx
```

---

### Sample systemd Service File

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Node.js Application
After=network.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/server.js
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

---

## 🔥 Process & PID Control

> When things go wrong — find it, inspect it, kill it.

### View Running Processes

```bash
ps aux                                    # All processes (all users)
ps aux | grep nginx                       # Filter by name
ps aux | grep -v grep | grep nginx        # Exclude grep itself from results
ps -ef --sort=-%cpu | head -20            # Top CPU consumers
ps -ef --sort=-%mem | head -20            # Top memory consumers
pgrep nginx                               # Just get the PID(s)
pgrep -a node                             # PID + full command
```

---

### Real-time Monitoring

```bash
top                                       # Live process view (press q to quit)
top -u ec2-user                           # Only show processes by specific user
htop                                      # Interactive (install: sudo yum install htop -y)
watch -n 2 'ps aux | grep nginx'          # Refresh every 2 seconds
```

---

### Kill Processes

```bash
# Graceful kill (SIGTERM — process can clean up):
kill PID
kill $(pgrep nginx)

# Force kill (SIGKILL — immediate, no cleanup):
kill -9 PID
kill -9 $(pgrep -f 'node server.js')

# Kill by name:
pkill nginx                               # SIGTERM to all matching
pkill -9 nginx                            # SIGKILL to all matching
pkill -f 'python3 app.py'                 # Match full command string

# Kill everything on a port:
sudo lsof -ti:8080 | xargs kill -9

# Kill all processes of a user:
pkill -u deploy_user
```

> 🚨 **Signal Reference:**
> | Signal | Number | Meaning |
> |--------|--------|---------|
> | SIGTERM | 15 | Graceful shutdown (default) |
> | SIGKILL | 9 | Force kill — cannot be caught |
> | SIGHUP | 1 | Hangup / reload config |
> | SIGINT | 2 | Interrupt (Ctrl+C) |

---

### Background Jobs & Persistence

```bash
# Run in background:
./long_script.sh &
echo "PID: $!"                            # Print background job PID

# Immune to SSH disconnect:
nohup python3 app.py > /var/log/app.log 2>&1 &
nohup ./deploy.sh &>> deploy.log &

# Manage background jobs:
jobs                                      # List jobs in current session
fg %1                                     # Bring job 1 to foreground
bg %2                                     # Send job 2 to background
disown %1                                 # Detach job 1 from shell completely

# Screen / tmux (persist across SSH):
screen -S mysession                       # New named session
screen -r mysession                       # Reattach
tmux new -s deploy                        # tmux session
tmux attach -t deploy                     # Reattach tmux
```

---

## 🌐 Network & Connectivity

> VPC, SGs, ports — if it's not working, these commands tell you why.

### Port & Socket Inspection

```bash
sudo ss -tlnp                             # Listening TCP ports + process (modern)
sudo ss -tlnp | grep :80                  # Check if port 80 is bound
sudo netstat -tlnp                        # Same (older systems)
sudo netstat -an | grep ESTABLISHED       # Active connections
sudo lsof -i :8080                        # What process owns port 8080
sudo lsof -i -P -n | grep LISTEN         # All listening ports
```

---

### Connectivity Testing

```bash
# HTTP tests:
curl -I http://localhost                   # Headers only (check status code)
curl -v https://api.example.com            # Verbose (show TLS, headers, body)
curl -o /dev/null -s -w "%{http_code}" http://localhost  # Just status code
curl -X POST -H "Content-Type: application/json" \
  -d '{"key":"val"}' http://localhost/api  # POST request test

# TCP port test:
telnet mydb.rds.amazonaws.com 5432        # Can we reach RDS port?
nc -zv myhost.com 443                     # Netcat port check (faster)
nc -zv -w 3 10.0.1.5 3306                 # With 3s timeout

# ICMP / routing:
ping -c 4 8.8.8.8                         # 4 pings to Google DNS
traceroute google.com                     # Hop-by-hop route
mtr google.com                            # Continuous traceroute (install first)
```

---

### DNS Resolution

```bash
nslookup myapp.example.com                # Basic DNS lookup
dig myapp.example.com                     # Detailed DNS query
dig +short mydb.cluster.rds.amazonaws.com # Just the IP
dig @8.8.8.8 myapp.example.com            # Query using specific DNS server
dig myapp.example.com MX                  # MX records
dig myapp.example.com TXT                 # TXT/SPF records
host myapp.example.com                    # Simple forward lookup
```

---

### Network Interface Info

```bash
ip addr show                              # All interfaces and IPs
ip addr show eth0                         # Specific interface
ip route show                             # Routing table
ip route get 10.0.0.1                     # Which interface routes to IP
ifconfig                                  # Legacy (still works on AL1)
```

---

### Firewall (iptables)

```bash
sudo iptables -L -n -v                    # List all rules with packet counts
sudo iptables -L INPUT -n                 # Just INPUT chain
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # Allow port 80
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT   # Delete rule
sudo iptables -F                          # ⚠️ FLUSH all rules (careful!)
sudo iptables-save > /etc/iptables.rules  # Save rules
sudo iptables-restore < /etc/iptables.rules  # Restore rules
```

---

## 💾 File, Disk & Storage

> EBS volumes, log management, disk emergencies — these ops keep instances healthy.

### Disk Space & Usage

```bash
df -h                                     # All mounts, human-readable
df -h /                                   # Just root volume
df -i                                     # Inode usage (another cause of "disk full")
du -sh /var/log/*                         # Size of each log directory
du -sh * | sort -rh | head -20            # Top 20 largest items
du -sh /var/log /opt /home                # Compare directories
ncdu /                                    # Interactive disk usage (install: yum install ncdu)
```

---

### Block Devices & Volumes (EBS)

```bash
lsblk                                     # List all block devices + mount points
lsblk -f                                  # Include filesystem type + UUID
blkid                                     # Show UUIDs (needed for /etc/fstab)
fdisk -l                                  # Detailed partition info

# After attaching new EBS volume:
lsblk                                     # Identify device name (xvdf, nvme1n1p1...)
sudo file -s /dev/xvdf                    # Check if formatted (data = unformatted)
sudo mkfs.ext4 /dev/xvdf                  # ⚠️ Format (DESTROYS existing data)
sudo mkdir -p /mnt/data
sudo mount /dev/xvdf /mnt/data
df -h /mnt/data                           # Verify mount

# Persist mount across reboots:
sudo blkid /dev/xvdf                      # Get UUID
sudo vim /etc/fstab
# Add line: UUID=xxxx /mnt/data ext4 defaults,nofail 0 2
sudo mount -a                             # Test fstab (before rebooting)
```

---

### Log Management

```bash
tail -f /var/log/nginx/error.log          # Stream log live
tail -f /var/log/syslog                   # System log stream
tail -n 200 /var/log/nginx/access.log     # Last 200 lines
head -n 50 /var/log/app.log               # First 50 lines
cat /var/log/nginx/error.log | less       # Page through file
zcat /var/log/nginx/access.log.gz | tail  # Read compressed log

# Search logs:
grep "ERROR" /var/log/app.log             # Find errors
grep -rn "OOM" /var/log/                  # Recursive search
grep -c "404" /var/log/nginx/access.log   # Count occurrences
grep -i "fail\|error\|killed" /var/log/syslog  # Case-insensitive multi-pattern

# Log rotation:
sudo logrotate -f /etc/logrotate.conf     # Force log rotation
```

---

### Find & Search Files

```bash
find / -name "nginx.conf" 2>/dev/null     # Find file by name
find /var/log -name "*.log" -mtime +7     # Logs older than 7 days
find /var/log -name "*.log" -mtime +7 -delete  # Delete old logs
find / -perm /4000 -type f 2>/dev/null    # Find SUID files (security audit)
find /opt -user root -type f              # Files owned by root
find . -size +100M                        # Files larger than 100MB
locate nginx.conf                         # Fast name search (uses database)
which nginx                               # Full path of command
whereis nginx                             # Binary + man page locations
```

---

## ☁️ AWS CLI Operations

> The command-line superpower for every AWS resource. Configure once, automate everything.

### Setup & Identity

```bash
aws configure                             # Interactive setup (key, secret, region, output)
aws configure --profile staging           # Named profile setup
aws configure list                        # Show current config
aws configure list-profiles              # All configured profiles

# Verify who you are:
aws sts get-caller-identity               # Account ID, User/Role ARN
aws sts get-caller-identity --profile prod  # Check specific profile

# Use a profile:
export AWS_PROFILE=staging                # Set for entire session
aws s3 ls --profile prod                  # One-off profile usage
```

---

### EC2 Instances

```bash
# List instances (formatted):
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].{
    ID:InstanceId,
    Name:Tags[?Key==`Name`]|[0].Value,
    IP:PrivateIpAddress,
    PublicIP:PublicIpAddress,
    State:State.Name,
    Type:InstanceType
  }' \
  --output table

# Filter by state:
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].InstanceId' \
  --output text

# Start / Stop / Terminate:
aws ec2 start-instances --instance-ids i-0abc1234567890def
aws ec2 stop-instances --instance-ids i-0abc1234567890def
aws ec2 terminate-instances --instance-ids i-0abc1234567890def  # ⚠️ PERMANENT

# Get instance metadata (from inside EC2):
curl http://169.254.169.254/latest/meta-data/instance-id
curl http://169.254.169.254/latest/meta-data/local-ipv4
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

---

### S3 Operations

```bash
# List:
aws s3 ls                                 # All buckets
aws s3 ls s3://my-bucket/                 # Bucket contents
aws s3 ls s3://my-bucket/ --recursive --human-readable

# Copy & Sync:
aws s3 cp file.txt s3://my-bucket/path/   # Upload file
aws s3 cp s3://my-bucket/path/file.txt .  # Download file
aws s3 sync /var/www/html s3://my-bucket/ # Sync directory to S3
aws s3 sync s3://my-bucket/ /var/www/html # Sync S3 to local
aws s3 sync s3://my-bucket/ /local/ \
  --exclude "*.tmp" --delete              # Sync with exclusions + delete

# Remove:
aws s3 rm s3://my-bucket/file.txt         # Delete single file
aws s3 rm s3://my-bucket/ --recursive    # ⚠️ Delete all objects in bucket

# Presigned URL (temporary public access):
aws s3 presign s3://my-bucket/report.pdf --expires-in 3600  # 1 hour
```

---

### CloudWatch Logs

```bash
# Stream logs live (like tail -f):
aws logs tail /aws/lambda/my-function --follow
aws logs tail /ecs/my-cluster --follow
aws logs tail /aws/lambda/my-fn --since 30m  # Last 30 minutes

# Query logs (CloudWatch Insights):
aws logs start-query \
  --log-group-name /aws/lambda/my-function \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | limit 50'

# Get query results:
aws logs get-query-results --query-id <query-id-from-above>

# List log groups:
aws logs describe-log-groups --query 'logGroups[].logGroupName' --output text
```

---

### Systems Manager (SSH-less Access)

```bash
# Open shell session (no port 22, no keypair needed):
aws ssm start-session --target i-0abc1234567890def

# Run command on one instance:
aws ssm send-command \
  --instance-ids i-0abc1234567890def \
  --document-name AWS-RunShellScript \
  --parameters commands=["sudo systemctl restart nginx"]

# Run command on many instances (by tag):
aws ssm send-command \
  --targets "Key=tag:Environment,Values=production" \
  --document-name AWS-RunShellScript \
  --parameters commands=["df -h"]

# Get command output:
aws ssm get-command-invocation \
  --command-id <command-id> \
  --instance-id i-0abc1234567890def
```

---

### Secrets Manager

```bash
# Get secret value:
aws secretsmanager get-secret-value \
  --secret-id prod/myapp/db \
  --query SecretString \
  --output text

# Parse JSON secret directly:
aws secretsmanager get-secret-value \
  --secret-id prod/myapp/db \
  --query SecretString \
  --output text | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['password'])"

# Create / update secret:
aws secretsmanager create-secret \
  --name prod/myapp/api-key \
  --secret-string "mysupersecretkey"

aws secretsmanager put-secret-value \
  --secret-id prod/myapp/api-key \
  --secret-string "newvalue"
```

---

### ECR (Container Registry)

```bash
# Authenticate Docker to ECR:
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS \
  --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Build, tag, push:
docker build -t myapp:latest .
docker tag myapp:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# List images in repo:
aws ecr list-images --repository-name myapp --output table

# Delete old images:
aws ecr batch-delete-image \
  --repository-name myapp \
  --image-ids imageTag=old-tag
```

---

### RDS

```bash
# List instances:
aws rds describe-db-instances \
  --query 'DBInstances[].{
    ID:DBInstanceIdentifier,
    Engine:Engine,
    Status:DBInstanceStatus,
    Endpoint:Endpoint.Address,
    Port:Endpoint.Port
  }' \
  --output table

# Create snapshot:
aws rds create-db-snapshot \
  --db-instance-identifier my-prod-db \
  --db-snapshot-identifier my-prod-db-backup-$(date +%Y%m%d)

# Reboot instance:
aws rds reboot-db-instance --db-instance-identifier my-prod-db
```

---

### Auto Scaling Groups

```bash
# Describe ASG:
aws autoscaling describe-auto-scaling-groups \
  --query 'AutoScalingGroups[].{
    Name:AutoScalingGroupName,
    Min:MinSize,
    Max:MaxSize,
    Desired:DesiredCapacity
  }' \
  --output table

# Scale out / in:
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name my-asg \
  --desired-capacity 10                   # Scale to 10 instances

# Suspend / resume scaling:
aws autoscaling suspend-processes --auto-scaling-group-name my-asg \
  --scaling-processes Launch Terminate
aws autoscaling resume-processes --auto-scaling-group-name my-asg \
  --scaling-processes Launch Terminate
```

---

### ALB / Load Balancer

```bash
# Check target health:
aws elbv2 describe-target-health \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-tg/abc123 \
  --query 'TargetHealthDescriptions[].{
    Target:Target.Id,
    Port:Target.Port,
    Health:TargetHealth.State,
    Reason:TargetHealth.Reason
  }' \
  --output table

# List load balancers:
aws elbv2 describe-load-balancers \
  --query 'LoadBalancers[].{Name:LoadBalancerName,DNS:DNSName,State:State.Code}' \
  --output table
```

---

### CloudFormation

```bash
# Deploy stack:
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name my-stack \
  --parameter-overrides Env=prod InstanceType=t3.medium \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM

# Describe stack:
aws cloudformation describe-stacks --stack-name my-stack \
  --query 'Stacks[0].StackStatus'

# Get stack outputs:
aws cloudformation describe-stacks --stack-name my-stack \
  --query 'Stacks[0].Outputs' --output table

# Delete stack:
aws cloudformation delete-stack --stack-name my-stack

# Validate template:
aws cloudformation validate-template --template-body file://template.yaml
```

---

## 🚨 Error & Recovery

> The firefighting toolkit. When the pager goes off at 3am — these save you.

### Service Logs (journalctl)

```bash
journalctl -u nginx                       # All logs for nginx service
journalctl -u nginx -n 100                # Last 100 lines
journalctl -u nginx -f                    # Follow (live tail)
journalctl -u nginx --since "1 hour ago"  # Time-based filter
journalctl -u nginx --since "2024-01-01 10:00:00"
journalctl -xe                            # Last boot errors (critical context)
journalctl -b                             # All logs from current boot
journalctl -b -1                          # Logs from previous boot
journalctl -p err -b                      # Only errors from current boot
journalctl --disk-usage                   # How much space logs use
journalctl --vacuum-time=7d               # Delete logs older than 7 days
```

---

### Kernel & OOM Messages

```bash
dmesg                                     # Full kernel ring buffer
dmesg | tail -50                          # Last 50 kernel messages
dmesg | grep -i "oom\|killed\|error\|fail" # Filter critical issues
dmesg -T | tail -30                       # With human-readable timestamps
dmesg -w                                  # Follow kernel messages live
cat /var/log/kern.log | grep -i oom       # OOM killer events
```

---

### Memory Analysis

```bash
free -h                                   # RAM + Swap overview
cat /proc/meminfo                         # Detailed memory info
vmstat -s                                 # Virtual memory stats
vmstat 2 10                               # Stats every 2s, 10 times
sar -r 1 5                                # Memory utilization (sysstat)

# Drop page cache (emergency memory recovery):
sudo sync
sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'

# Find what's using memory:
ps aux --sort=-%mem | head -10
smem -r | head -10                        # Accurate memory by process
```

---

### Port Conflicts

```bash
sudo lsof -i :80                          # What's using port 80
sudo lsof -i :8080 -i :8443              # Multiple ports
sudo lsof -i -P -n | grep LISTEN         # All listening ports with PIDs

# Kill whatever is using a port:
sudo lsof -ti:8080 | xargs kill -9
sudo fuser -k 8080/tcp                    # Alternative approach
```

---

### Deep Process Debugging

```bash
strace -p PID                             # Trace syscalls of running process
strace -p PID -s 200 -o /tmp/trace.log   # Write trace to file
strace -e trace=file,network ./app        # Trace only file/network syscalls
ltrace -p PID                             # Trace library calls
lsof -p PID                               # All files/sockets opened by PID
cat /proc/PID/status                      # Process status from kernel
cat /proc/PID/environ | tr '\0' '\n'      # Process environment variables
```

---

### Filesystem Recovery

```bash
# MUST unmount before fsck:
sudo umount /dev/xvdf

# Check and auto-repair:
sudo fsck -y /dev/xvdf                    # -y = yes to all repairs
sudo fsck -n /dev/xvdf                    # Dry-run (check only, no repair)
sudo e2fsck -f /dev/xvdf                  # Force check even if clean
sudo xfs_repair /dev/xvdf                 # For XFS filesystems

# After repair:
sudo mount /dev/xvdf /mnt/data
df -h /mnt/data
```

---

### System Resources (CPU / Load)

```bash
uptime                                    # Load average (1, 5, 15 min)
cat /proc/loadavg                         # Raw load average
nproc                                     # Number of CPU cores
lscpu                                     # CPU details
iostat -x 1 5                             # Disk I/O stats every 1s, 5 times
iotop                                     # Real-time I/O by process
sar -u 1 10                               # CPU utilization over time
```

---

## 🐳 Docker & Containers

> Container operations on EC2, ECS, or any Docker host.

### Container Lifecycle

```bash
docker ps                                 # Running containers
docker ps -a                              # All containers (inc. stopped)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

docker start my-app                       # Start stopped container
docker stop my-app                        # Graceful stop (SIGTERM, 10s timeout)
docker kill my-app                        # Immediate stop (SIGKILL)
docker restart my-app                     # Stop + Start
docker rm my-app                          # Remove stopped container
docker rm -f my-app                       # Force remove (even if running)
docker rm -f $(docker ps -aq)             # ⚠️ Remove ALL containers
```

---

### Images

```bash
docker images                             # List local images
docker images | grep none                 # Dangling images (wasted space)
docker pull nginx:latest                  # Pull from registry
docker rmi nginx:latest                   # Remove image
docker rmi $(docker images -q)            # ⚠️ Remove all images
docker image prune -a                     # Remove all unused images
docker history my-app:latest             # Show image layers
docker inspect my-app:latest             # Full image metadata
```

---

### Logs & Debugging

```bash
docker logs my-app                        # All logs
docker logs my-app --tail 100             # Last 100 lines
docker logs -f my-app                     # Follow (live tail)
docker logs -f my-app --since 1h          # Since 1 hour ago
docker logs my-app 2>&1 | grep -i error   # Filter errors

# Shell into running container:
docker exec -it my-app bash               # Bash shell
docker exec -it my-app sh                 # Sh (if bash not available)
docker exec -it my-app env                # Print env variables
docker exec -it my-app cat /etc/hosts     # Run any command

# Shell into stopped container:
docker run --rm -it --entrypoint bash my-app:latest
```

---

### Performance & Resource Usage

```bash
docker stats                              # Live CPU/MEM for all containers
docker stats my-app --no-stream           # One-shot stats
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker top my-app                         # Processes inside container
docker inspect my-app | grep -i memory    # Memory limits set
```

---

### Cleanup (Purge Everything)

```bash
docker system df                          # Show disk usage by Docker
docker system prune                       # Remove stopped containers + dangling images
docker system prune -af                   # ⚠️ Remove ALL unused images + containers
docker volume prune                       # Remove unused volumes
docker network prune                      # Remove unused networks
```

---

## 📦 Package Management

### Amazon Linux (yum / dnf)

```bash
sudo yum update -y                        # Update all packages
sudo yum install nginx -y                 # Install package
sudo yum install -y htop nc nmap strace   # Multiple packages
sudo yum remove nginx                     # Remove package
sudo yum search httpd                     # Search for package
sudo yum info nginx                       # Package details
sudo yum list installed                   # All installed packages
sudo yum list installed | grep nginx      # Check if installed
sudo yum clean all                        # Clear cache
```

### Ubuntu / Debian (apt)

```bash
sudo apt update                           # Refresh package list
sudo apt upgrade -y                       # Upgrade all packages
sudo apt install nginx -y                 # Install
sudo apt remove nginx                     # Remove (keep config)
sudo apt purge nginx                      # Remove + delete config
sudo apt autoremove                       # Remove unused dependencies
sudo apt-cache search nginx               # Search
sudo dpkg -l | grep nginx                 # Check if installed
```

### Python (pip)

```bash
pip3 install boto3                        # Install package
pip3 install -r requirements.txt          # Install from file
pip3 list                                 # Installed packages
pip3 show boto3                           # Package info
pip3 freeze > requirements.txt            # Export installed packages
pip3 install --upgrade boto3              # Upgrade package
```

---

## 🔒 Security & Hardening

### SSH Security

```bash
# Check who's logged in:
who                                       # Current sessions
w                                         # Detailed session info
last | head -20                           # Login history
lastb | head -20                          # Failed login attempts
tail -f /var/log/secure                   # Auth log (Amazon Linux)
tail -f /var/log/auth.log                 # Auth log (Ubuntu)
grep "Failed password" /var/log/secure | wc -l  # Count brute force attempts

# Harden SSH config:
sudo vim /etc/ssh/sshd_config
# Key settings:
# PermitRootLogin no
# PasswordAuthentication no
# MaxAuthTries 3
# AllowUsers ec2-user deploy

sudo systemctl restart sshd               # Apply changes
```

---

### Auditd & Compliance

```bash
sudo auditctl -l                          # List audit rules
sudo ausearch -k sudo_log                 # Search audit logs by key
sudo aureport --auth                      # Authentication report
sudo aureport --failed                    # Failed event report
sudo tail -f /var/log/audit/audit.log     # Live audit log
```

---

### Open Files & Limits

```bash
ulimit -a                                 # Current user limits
ulimit -n                                 # Open file limit
cat /proc/sys/fs/file-max                 # System-wide file limit
sysctl fs.file-max                        # Same via sysctl

# Raise limits temporarily:
ulimit -n 65536

# Raise permanently (EC2 production):
sudo vim /etc/security/limits.conf
# Add:
# * soft nofile 65536
# * hard nofile 65536
```

---

## 🎯 Quick Reference Card

### Emergency Runbook

```bash
# ---- SYSTEM DOWN? ----
systemctl status <service>                # 1. What failed?
journalctl -xe                            # 2. Why did it fail?
free -h && df -h                          # 3. Out of memory or disk?
top                                       # 4. CPU spike?
dmesg | tail -20                          # 5. Kernel/hardware issue?

# ---- PORT NOT RESPONDING? ----
sudo ss -tlnp | grep :<port>              # Is process listening?
curl -v http://localhost:<port>           # Can we hit it locally?
sudo iptables -L -n | grep <port>         # Blocked by iptables?
# Then check AWS Security Groups in console.

# ---- DISK FULL? ----
df -h                                     # Which mount?
du -sh /var/log/* | sort -rh | head -10   # Find the culprit
find /var/log -name "*.log" -mtime +3 -delete  # Delete old logs
docker system prune -af                   # Clear Docker if applicable

# ---- HIGH CPU? ----
top -b -n 1 | head -20                    # Snapshot top processes
ps aux --sort=-%cpu | head -10            # CPU hogs
strace -p <PID>                           # What is it doing?

# ---- PROCESS ZOMBIE/HUNG? ----
ps aux | grep Z                           # Zombie processes
kill -9 <PID>                             # Force kill
pkill -f <process_name>                   # Kill by name
sudo lsof -ti:<port> | xargs kill -9     # Kill by port
```

---

## 📌 Tips & Best Practices

> 💡 **Always `enable` services** after installing on EC2 — otherwise they won't survive a reboot.

> 💡 **Use `ss` instead of `netstat`** — faster and more accurate on modern kernels.

> 💡 **Prefer `systemctl reload`** over `restart` for nginx/apache — it has zero downtime.

> 💡 **`nohup` + `&`** is your friend for long-running EC2 scripts launched over SSH.

> 💡 **`aws ssm start-session`** eliminates the need for bastion hosts and open SSH ports.

> 💡 **Always run `sudo mount -a`** after editing `/etc/fstab` to test before rebooting.

> ⚠️ **Never run `kill -9` as first option** — always try graceful `kill` or `systemctl stop` first.

> ⚠️ **`chmod 777` is never the answer** — it's a security disaster. Use proper permissions.

> ⚠️ **`rm -rf /`** — you already know. Lock it with `alias rm='rm -i'` in `.bashrc`.

---

## 🤝 Contributing

Found a command missing? Have a better example? PRs are welcome!

1. Fork the repo
2. Add commands in the correct category
3. Follow the existing format (command, comment, use-case note)
4. Submit a pull request

---

## 📄 License

MIT — Use freely, credit appreciated.

---

*Built with 14 years of production battle scars on AWS. May your on-call nights be short and your logs be clear.* 🚀
