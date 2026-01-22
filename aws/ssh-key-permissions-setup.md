# Setting Up SSH Key Permissions for AWS EC2

This guide explains how to set the correct permissions on your AWS EC2 key pair (`.pem`) file so that SSH will accept it for authentication.

## Why This Is Needed

SSH requires that private key files have restrictive permissions for security reasons. If your `.pem` file has permissions that are too open (allowing others to read it), SSH will refuse to use it and display an error like:

```
Permissions 0644 for 'your-key.pem' are too open.
It is required that your private key files are NOT accessible by others.
```

## Prerequisites

- Downloaded AWS EC2 key pair file (`.pem` file)
- Access to terminal/command line

## Locate Your Key File

First, identify where you saved your `.pem` file. Common locations include:
- `~/Downloads/your-key.pem` (Linux)
- `C:\Users\YourUsername\Downloads\your-key.pem` (Windows)
- `~/Desktop/your-key.pem`

**Note:** Replace `your-key.pem` with the actual name of your downloaded key file.

## Set Permissions on Linux

### Using chmod Command

Navigate to the directory containing your `.pem` file and set the permissions:

```bash
chmod 400 your-key.pem
```

Or alternatively:

```bash
chmod 600 your-key.pem
```

**Note:** 
- `400` gives read-only permission to the owner only
- `600` gives read and write permission to the owner only
- Both are acceptable for SSH key files

### Verify Permissions

Check that the permissions were set correctly:

```bash
ls -l your-key.pem
```

You should see output similar to:

```
-r-------- 1 username username 1674 Jan 22 10:30 your-key.pem
```

Or with `600`:

```
-rw------- 1 username username 1674 Jan 22 10:30 your-key.pem
```

The important part is that only the owner has read (and optionally write) permissions.

### Test the Connection

After setting the correct permissions, test your SSH connection:

```bash
ssh -i /path/to/your-key.pem ubuntu@your-instance-ip
```

**Note:** Replace `/path/to/your-key.pem` with the actual path to your key file, and `your-instance-ip` with your EC2 instance's public IP address.

## Set Permissions on Windows

### Using File Properties (Windows)

1. Right-click on your `.pem` file
2. Select **Properties**
3. Go to the **Security** tab

![Image 1: Right-click menu showing Properties option](images/1.png)


*Figure 1: Right-click on the .pem file and select Properties to access file security settings.*

4. Click **Advanced**

![Image 2: Go to advanced setting](images/2.png)

*Figure 2: In the Properties window, navigate to the Security tab and click the Advanced button to modify file permissions.*

5. Disable inheritance and remove all users except yourself

![Image 3: Disable Inheritance](images/3.png)

*Figure 3: Click the disable inheritance*

![Image 4: Click remove](images/4.png)

*Figure 4: Click the remove button*

6. Add the current user to the list with appropriate permissions 

![Image 5: Add Button](images/5.png)

*Figure 5: Click the add button*

![Image 6: Select a principal](images/6.png)

*Figure 6: Click the "Select a principal" button*

![Image 7: Username of the current user as the object](images/7.png)

*Figure 7: Enter the username of the current user as the object*

![Image 8: Select all the checkboxes](images/8.png)

*Figure 8: Select all the checkboxes for basic permissions*

**Note:** For SSH key files, you typically only need **Read** permission. However, selecting all permissions (Full control, Modify, Read & execute, Read, Write) will also work. The key requirement is that only you have access to the file.

7. Click **Apply** to apply changes

![Image 9: Click the "Apply" button](images/9.png)

*Figure 9: Click the "Apply" button to save the permission changes*

### Test the Connection

After setting the correct permissions, test your SSH connection using WSL:

```bash
ssh -i /mnt/c/path/to/your-key.pem ubuntu@your-instance-ip
```

**Note:** 
- Replace `/mnt/c/path/to/your-key.pem` with the actual path to your key file
- Replace `your-instance-ip` with your EC2 instance's public IP address
- On Windows with WSL, use `/mnt/c/` prefix for Windows paths (e.g., `/mnt/c/Users/YourUsername/Downloads/your-key.pem`)

## Troubleshooting

### Permission Denied Error

If you still get permission errors:

1. Verify the file path is correct
2. Check permissions again: `ls -l your-key.pem` (Linux) or file properties (Windows)
3. Ensure you're the owner of the file
4. Try setting permissions again with `chmod 400`

### File Not Found Error

- Verify the exact path to your `.pem` file
- Use absolute paths instead of relative paths
- On Windows with WSL, remember to use `/mnt/c/` prefix for Windows drives

### Wrong Username

- For Ubuntu instances, the default username is usually `ubuntu`
- For Amazon Linux, it's usually `ec2-user`
- For other distributions, check AWS documentation for the default username

### Key File Format Issues

- Ensure the file is in the correct format (PEM format)
- The file should start with `-----BEGIN RSA PRIVATE KEY-----` or `-----BEGIN PRIVATE KEY-----`
- Do not convert the file to other formats (like PPK for PuTTY) unless you're specifically using PuTTY

## Security Best Practices

- Never share your `.pem` file with others
- Never commit your `.pem` file to version control
- Store your key file in a secure location
- Consider using SSH agent to avoid specifying the key file path each time
- Use `400` permissions (read-only) instead of `600` if you don't need to modify the file

## Additional Notes

- If you're using PuTTY on Windows, you'll need to convert the `.pem` file to `.ppk` format using PuTTYgen
- Some cloud IDEs and SSH clients handle permissions automatically
- You can also use SSH config files to simplify connections (create `~/.ssh/config`)
