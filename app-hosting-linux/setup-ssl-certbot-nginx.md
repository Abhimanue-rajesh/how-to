# Setup SSL Certificate with Certbot for Nginx

This guide explains how to set up free SSL/TLS certificates using Let's Encrypt Certbot with Nginx on your server.

**Reference:** [Certbot Instructions for Nginx on Linux (pip)](https://certbot.eff.org/instructions?ws=nginx&os=pip)

## Prerequisites

- A server running Nginx
- A domain name pointing to your server's IP address
- SSH access to your server with sudo privileges
- Your website accessible via HTTP on port 80

## Step 1: Install System Dependencies

### For APT-based distributions (Debian, Ubuntu)

```bash
sudo apt update
sudo apt install python3 python3-dev python3-venv libaugeas-dev gcc
```

### For RPM-based distributions (Fedora, CentOS)

```bash
sudo dnf install python3 python-devel augeas-devel gcc
```

**Note:** On older distributions, use `yum` instead of `dnf`. On RHEL-based distributions, you may need `python3X` instead of `python3` (e.g., `python36`).

## Step 2: Remove Existing Certbot Packages

If you have Certbot installed via OS package manager, remove it first:

```bash
# For APT-based systems
sudo apt-get remove certbot

# For RPM-based systems
sudo dnf remove certbot
# or
sudo yum remove certbot
```

## Step 3: Set Up Python Virtual Environment

Create a virtual environment for Certbot:

```bash
sudo python3 -m venv /opt/certbot/
sudo /opt/certbot/bin/pip install --upgrade pip
```

## Step 4: Install Certbot

Install Certbot and the Nginx plugin:

```bash
sudo /opt/certbot/bin/pip install certbot certbot-nginx
```

## Step 5: Create Certbot Command Link

Make the `certbot` command available system-wide:

```bash
sudo ln -s /opt/certbot/bin/certbot /usr/local/bin/certbot
```

## Step 6: Obtain SSL Certificate

You have two options:

### Option A: Automatic Configuration (Recommended)

This will automatically configure Nginx and enable HTTPS:

```bash
sudo certbot --nginx
```

Follow the prompts to:
- Enter your email address
- Agree to terms of service
- Choose whether to redirect HTTP to HTTPS (recommended: Yes)

### Option B: Manual Configuration

If you prefer to configure Nginx manually:

```bash
sudo certbot certonly --nginx
```

This will only obtain the certificate without modifying your Nginx configuration.

**Note:** Replace any domain placeholders with your actual domain name when prompted.

## Step 7: Verify Certificate

Test that the certificate was obtained successfully:

```bash
sudo certbot certificates
```

You should see your domain listed with certificate paths.

## Step 8: Test Auto-Renewal

Test that certificate renewal works:

```bash
sudo certbot renew --dry-run
```

## Step 9: Set Up Automatic Renewal

Add a cron job to automatically renew certificates:

```bash
echo "0 0,12 * * * root /opt/certbot/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && sudo certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

This will check for certificate renewal twice daily (at midnight and noon).

## Step 10: Verify HTTPS

Visit your website in a browser:

```
https://your-domain.com
```

You should see a lock icon in the address bar, indicating the site is using HTTPS.

## Updating Certbot

Keep Certbot updated by running this command monthly:

```bash
sudo /opt/certbot/bin/pip install --upgrade certbot certbot-nginx
```

If you encounter errors during upgrade, you can reinstall:

```bash
sudo rm -rf /opt/certbot
```

Then repeat the installation steps from Step 3.

## Troubleshooting

### Certificate Not Obtained

- Ensure your domain points to your server's IP address
- Verify port 80 is open and accessible
- Check that Nginx is running: `sudo systemctl status nginx`
- Verify your Nginx configuration is correct

### Renewal Fails

- Check cron job is running: `sudo systemctl status cron`
- Verify certificate paths are correct: `sudo certbot certificates`
- Check Nginx configuration for any issues: `sudo nginx -t`

### Nginx Configuration Issues

After running Certbot, verify your Nginx configuration:

```bash
sudo nginx -t
```

If there are errors, Certbot may have created a backup of your original configuration. Check:

```bash
ls -la /etc/nginx/sites-available/
```

## Related Guides

- [Deploying Django with Nginx, SQLite, and Gunicorn on AWS](django-deployment-aws-nginx-gunicorn.md)

## Additional Resources

- [Certbot Official Documentation](https://certbot.eff.org/)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)

