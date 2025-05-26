

# Problem 3: Diagnose Me Doctor
## Initial Assessment
The VM is running Ubuntu 24.04 with 64GB storage, serving as an NGINX load balancer, and currently at 99% storage usage. This is critical because:

- May cause service interruptions

- Could prevent logging and monitoring

- Might lead to system instability

## Solution 
We are facing a situation where the storage usage on the VM has reached 99%. Since the VM is running only one service NGINX we must identify and resolve the cause promptly.
## 1. Verify Storage Usage 
<pre>
df -h  # Check overall disk usage
</pre>
## 2. Identify Which Directory Is Consuming Space
<pre>
sudo du -sh /* 2>/dev/null | sort -h  # Check root directory
</pre>
This will show which directories (such as /var, /usr, /home) are using the most space.  

For example the /var comsuming most space so we can check more detailed in that directory
<pre>
sudo du -sh /var/* 2>/dev/null | sort -h  # Check /var which often contains logs
</pre>


## 3. Check NGINX Specific Areas
Check NGINX logs
sudo du -sh /var/log/nginx/*
ls -lah /var/log/nginx/

Check NGINX cache
sudo du -sh /var/cache/nginx/*
## 4. Examine Log Rotation Configuration
<pre>
cat /etc/logrotate.d/nginx
cat /etc/logrotate.conf
</pre>

## 5. Verify Temporary Files
<pre>
sudo du -sh /tmp/*
</pre>

## 6. Stale Docker Images/Containers (If Docker is installed)
Even if NGINX is running as a system service, Docker might still be installed and unused.

Cause: 
- Orphaned Docker images or containers.
- Volumes not removed after container deletion.
- Recovery Steps:

Check disk usage:

docker system df
Clean up unused objects:

docker system prune -af --volumes

# Potential Root Causes, Impacts, and Solutions
## 1. NGINX Access/Error Logs Growing Uncontrolled
### Cause:

- Log rotation not properly configured

- Increased traffic causing log explosion

- Debug logging left enabled

Impact:

- Disk fills completely causing service failure

- Inability to write new logs

- System instability

### Recovery:

<pre>
### Immediate action:
sudo truncate -s 0 /var/log/nginx/access.log
sudo truncate -s 0 /var/log/nginx/error.log

### Long-term solution:
sudo nano /etc/logrotate.d/nginx  # Configure proper rotation
sudo systemctl restart nginx
</pre>

## 2. NGINX Cache Filling Up
### Cause:

- Proxy cache growing without limits

- Cache purge mechanism not working

### Impact:

- Reduced performance as cache becomes too large

- Eventually fills disk completely

### Recovery:

<pre>
### Clear cache:
sudo rm -rf /var/cache/nginx/*

### Configure proper cache limits in nginx.conf:
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m use_temp_path=off;
</pre>


## 3. Temporary Files Not Being Cleaned
### Cause:

- Failed package installations

- System updates leaving temporary files

- NGINX temporary files not being rotated

### Impact:

- Wasted storage space

- Potential file system corruption

### Recovery:

<pre>
#### Clean temporary files:
sudo apt-get clean
sudo rm -rf /tmp/*
sudo systemctl restart nginx
</pre>

# Preventive Measures
## 1. Implement Monitoring:

Set up alerts for storage >80%

Monitor inode usage

Log Management:

Configure proper log rotation

Consider centralized logging solution

## 2. Regular Maintenance:

<pre>
# Add to cron:
0 3 * * * apt-get clean
0 4 * * * find /tmp -type f -atime +1 -delete
</pre>

## 3. Configuration Review:

Audit NGINX configuration for proper cache settings

Verify debug logging is disabled in production

## 4. Storage Planning:

Consider increasing storage if consistently near capacity

Implement separate partitions for /var/log

