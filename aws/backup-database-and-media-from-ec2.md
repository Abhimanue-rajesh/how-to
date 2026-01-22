# Backup Database and Media Files from EC2 Instance

This guide explains how to backup your database and media files from an AWS EC2 instance to your local system using SCP (Secure Copy Protocol).

## Prerequisites

- Access to an EC2 instance
- SSH key pair (`.pem` file) with correct permissions
- SCP installed on your local machine (included with SSH on Linux/Mac, available on Windows via WSL or Git Bash)
- Knowledge of the file paths on both the EC2 instance and your local system

**Note:** If you haven't set up SSH key permissions yet, follow the guide: [Setting Up SSH Key Permissions for AWS EC2](ssh-key-permissions-setup.md)

## Important Notes

- The user executing the command should have the `.pem` file accessible from the terminal's current directory, or use the full path to the key file
- Ensure you have write permissions to the destination directory on your local system
- For large files or directories, the transfer may take some time depending on your internet connection

## Backup Database File

### Locate Your Database

The database file is typically located in your Django project directory. Common locations:
- `/home/ubuntu/your-project/db.sqlite3`
- `/home/ubuntu/your-project/your_project/db.sqlite3`

**Note:** Replace the path with your actual database file location.

### Backup Command

From your local terminal, navigate to the directory containing your `.pem` file, or use the full path to the key file:

#### Linux/Mac

```bash
scp -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/db.sqlite3 /path/to/local/backup/directory
```

#### Windows (Git Bash or WSL)

```bash
scp -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/db.sqlite3 "D:\YourProject\DatabaseBackups"
```

**Note:** 
- Replace `your-key.pem` with your actual key file name (or full path if not in current directory)
- Replace `your-instance-ip` with your EC2 instance's public IP address
- Replace `/home/ubuntu/your-project/db.sqlite3` with the actual path to your database file on the server
- Replace the destination path with your desired local backup location

### Example

```bash
scp -i oceano.pem ubuntu@3.28.242.169:/home/ubuntu/Oceano-Django/db.sqlite3 "D:\Oceano\DatabaseBackups"
```

## Backup Media Files

Media files are typically stored in a `media` directory within your Django project. Since this is a directory, you need to use the `-r` (recursive) flag to copy all files and subdirectories.

### Backup Command

#### Linux/Mac

```bash
scp -r -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/media /path/to/local/backup/directory
```

#### Windows (Git Bash or WSL)

```bash
scp -r -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/media "D:\YourProject\MediaBackups"
```

**Note:** 
- The `-r` flag is used for recursive transfer of directories
- Replace all placeholders with your actual values
- Ensure the destination directory exists or SCP will create it

### Example

```bash
scp -r -i oceano.pem ubuntu@3.28.242.169:/home/ubuntu/Oceano-Django/media "D:\Oceano\MediaBackups"
```

## Backup Specific Files

You can also backup individual files using the same SCP command without the `-r` flag.

### Example: Backup Excel File

```bash
scp -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/raffle_draw_data.xlsx "D:\YourProject\RaffleDraw"
```

**Note:** Replace the file path and destination with your actual values.

## Command Structure

The general SCP command structure is:

```bash
scp [options] -i key-file user@remote-host:remote-path local-path
```

### Common Options

- `-i`: Specify the identity (private key) file
- `-r`: Recursive copy for directories
- `-v`: Verbose mode (shows progress)
- `-P`: Specify port (if not using default port 22)

## Best Practices

### Organize Backups by Date

Create dated backup directories to keep multiple backups:

```bash
# Create a dated directory
mkdir "D:\YourProject\DatabaseBackups\$(date +%Y-%m-%d)"

# Backup with date in filename
scp -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/db.sqlite3 "D:\YourProject\DatabaseBackups\$(date +%Y-%m-%d)\db.sqlite3"
```

### Use Absolute Paths

Always use absolute paths to avoid confusion:

```bash
scp -i /full/path/to/your-key.pem ubuntu@your-instance-ip:/full/path/to/file /full/path/to/destination
```

### Verify Backups

After backing up, verify the files were copied correctly:

```bash
# Check file size
ls -lh "D:\YourProject\DatabaseBackups\db.sqlite3"

# Compare file sizes (if possible)
# Server: du -h /home/ubuntu/your-project/db.sqlite3
# Local: Check file properties
```

## Troubleshooting

### Permission Denied Error

If you get a permission denied error:

1. Check that your `.pem` file has correct permissions:
   - Linux/Mac: `chmod 400 your-key.pem`
   - Windows: See [SSH Key Permissions Guide](ssh-key-permissions-setup.md)

2. Verify you have write permissions to the destination directory

### Connection Timeout

If the connection times out:

1. Check that your EC2 instance is running
2. Verify the security group allows SSH (port 22) from your IP
3. Check your internet connection
4. Verify the instance IP address is correct

### File Not Found

If you get a "file not found" error:

1. Verify the file path on the server is correct
2. Check file permissions on the server
3. Ensure the file exists by SSHing into the server first:
   ```bash
   ssh -i your-key.pem ubuntu@your-instance-ip
   ls -la /home/ubuntu/your-project/db.sqlite3
   ```

### Destination Directory Issues

- Ensure the destination directory exists, or SCP will create it
- On Windows, use quotes around paths with spaces
- Use forward slashes or escaped backslashes in Windows paths when using Git Bash/WSL

### Large File Transfers

For large files or directories:

- Use `-v` flag to see progress: `scp -v -r -i ...`
- Consider compressing files on the server first, then transferring
- Use `rsync` for incremental backups of large directories

## Automated Backup Script

You can create a backup script to automate the process:

### Linux/Mac Script

```bash
#!/bin/bash
DATE=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_DIR="$HOME/backups/$DATE"
mkdir -p "$BACKUP_DIR"

scp -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/db.sqlite3 "$BACKUP_DIR/"
scp -r -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/media "$BACKUP_DIR/"

echo "Backup completed: $BACKUP_DIR"
```

### Windows Batch Script

```batch
@echo off
for /f "tokens=2-4 delims=/ " %%a in ('date /t') do (set mydate=%%c-%%a-%%b)
set BACKUP_DIR=D:\YourProject\Backups\%mydate%
mkdir "%BACKUP_DIR%"

scp -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/db.sqlite3 "%BACKUP_DIR%\"
scp -r -i your-key.pem ubuntu@your-instance-ip:/home/ubuntu/your-project/media "%BACKUP_DIR%\"

echo Backup completed: %BACKUP_DIR%
```

**Note:** Replace all placeholders with your actual values and adjust paths as needed.

## Security Considerations

- Never commit backup scripts with hardcoded credentials to version control
- Store `.pem` files securely
- Consider using environment variables for sensitive information
- Regularly rotate and update your SSH keys
- Use strong passwords for database files if encrypted

## Related Guides

- [Setting Up SSH Key Permissions for AWS EC2](ssh-key-permissions-setup.md)
- [Deploying Django with Nginx, SQLite, and Gunicorn on AWS](../app-hosting-linux/django-deployment-aws-nginx-gunicorn.md)

