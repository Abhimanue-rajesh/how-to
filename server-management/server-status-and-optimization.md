# Server Status Check and Optimization

This guide covers commands to check server status and various optimization techniques for Linux servers.

## Check Server Status

Use the following commands to monitor your server's current status:

### Memory Usage

```bash
free -h
```

Displays memory usage in human-readable format (shows total, used, free, and available memory).

### Disk Usage

```bash
df -h
```

Shows disk space usage for all mounted filesystems in human-readable format.

### System Uptime

```bash
uptime
```

Displays how long the system has been running, current time, number of users, and system load averages.

## Create 2GB Swap File

If your server needs additional swap space, follow these steps to create a 2GB swap file:

### Step 1: Allocate Swap File

```bash
sudo fallocate -l 2G /swapfile
```

**Note:** You can change `2G` to a different size (e.g., `4G`, `1G`) based on your requirements.

### Step 2: Set Correct Permissions

```bash
sudo chmod 600 /swapfile
```

### Step 3: Set Up Swap Space

```bash
sudo mkswap /swapfile
```

### Step 4: Enable Swap File

```bash
sudo swapon /swapfile
```

### Step 5: Make Swap Persistent After Reboot

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

This ensures the swap file is automatically enabled on system reboot.

## Tune Kernel to Avoid Hard Freezes

Optimize kernel parameters to prevent system freezes:

### Set Swappiness

```bash
sudo sysctl vm.swappiness=10
```

**Note:** Lower values (like 10) mean the system will swap less aggressively. Adjust based on your needs (default is usually 60).

### Set Cache Pressure

```bash
sudo sysctl vm.vfs_cache_pressure=50
```

**Note:** This controls how aggressively the kernel reclaims memory used for caching directory and inode objects.

### Make Kernel Tuning Persistent

To persist these settings after reboot:

```bash
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
```

**Note:** Adjust the values (`10` and `50`) based on your server's requirements.

## Limit Systemd Journal Size

Prevent journal logs from consuming too much disk space:

### Edit Journal Configuration

```bash
sudo nano /etc/systemd/journald.conf
```

### Update Configuration Values

Find and modify the following lines (remove the `#` to uncomment if needed):

```
SystemMaxUse=100M
RuntimeMaxUse=50M
```

**Note:** Adjust the values (`100M` and `50M`) based on your disk space and logging requirements.

After making changes, restart the journal service:

```bash
sudo systemctl restart systemd-journald
```

## SSH Keepalive (Prevent Dead Sessions)

Configure SSH to prevent connection timeouts and dead sessions:

### Edit SSH Configuration

```bash
sudo nano /etc/ssh/sshd_config
```

### Add Keepalive Settings

Add or modify the following lines:

```
ClientAliveInterval 60
ClientAliveCountMax 3
```

**Note:** 
- `ClientAliveInterval 60` sends a keepalive message every 60 seconds
- `ClientAliveCountMax 3` allows 3 missed responses before disconnecting
- Adjust these values based on your network conditions and requirements

### Restart SSH Service

```bash
sudo systemctl restart ssh
```

Or on some systems:

```bash
sudo systemctl restart sshd
```

## Verification

After making changes, verify the configurations:

- **Swap file**: Check with `free -h` or `swapon --show`
- **Kernel parameters**: Verify with `sysctl vm.swappiness` and `sysctl vm.vfs_cache_pressure`
- **Journal size**: Check with `journalctl --disk-usage`
- **SSH keepalive**: Test by maintaining an SSH connection idle for the configured interval

## Troubleshooting

- If swap file creation fails, ensure you have enough disk space: `df -h`
- If kernel parameters don't persist, verify `/etc/sysctl.conf` syntax
- If SSH changes don't take effect, check for syntax errors: `sudo sshd -t`
- Always test SSH connections after modifying SSH configuration to avoid locking yourself out

