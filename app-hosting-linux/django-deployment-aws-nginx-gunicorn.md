# Deploying Django with Nginx, SQLite, and Gunicorn on AWS

This guide walks you through deploying a Django application on AWS EC2 using Nginx as a reverse proxy, Gunicorn as the WSGI server, and SQLite as the database.

**Reference:** This guide is based on the [DigitalOcean Django deployment tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu).

## Prerequisites

- An AWS account
- A Django project ready for deployment
- Basic knowledge of Linux command line

## Step 1: Create AWS EC2 Instance

### Login to AWS Account

1. Log in to your Amazon AWS account
2. Navigate to the AWS Management Console

### Change Region

1. In the top-right corner, select your region
2. Change the region to **UAE (Middle East - Bahrain)** or your preferred region

### Create EC2 Instance

1. Go to the **EC2 Dashboard**
2. Click **Launch Instance**
3. Configure the instance with the following settings:
   - **Name**: Choose a descriptive name for your instance
   - **AMI**: Select **Ubuntu** (latest LTS version recommended)
   - **Instance Type**: Choose based on your requirements (t2.micro for testing, larger for production)
   - **Key Pair**: Create a new key pair or select an existing one
     - Click **Create new key pair**
     - Give it a name
     - Download the `.pem` file and store it securely
   - **Network Settings**: 
     - Create a new security group or use an existing one
     - Configure security group rules:
       - **SSH (22)**: Allow from your IP or anywhere (0.0.0.0/0) for testing
       - **HTTP (80)**: Allow from anywhere (0.0.0.0/0)
       - **HTTPS (443)**: Allow from anywhere (0.0.0.0/0) if using SSL
   - **Storage**: Configure disk size as needed (minimum 8GB recommended)

4. Click **Launch Instance**

## Step 2: Connect to the Instance

### Set Up SSH Key Permissions

Before connecting, you need to set the correct permissions on your `.pem` key file. Follow the guide: [Setting Up SSH Key Permissions for AWS EC2](aws/ssh-key-permissions-setup.md)

### Using SSH (Linux/Mac)

```bash
ssh -i /path/to/your-key.pem ubuntu@your-instance-ip
```

### Using SSH (Windows)

Use PuTTY or Windows Subsystem for Linux (WSL) with the same command format.

**Note:** Replace `/path/to/your-key.pem` with the actual path to your downloaded key file, and `your-instance-ip` with your EC2 instance's public IP address.

## Step 3: Update and Upgrade System

Once connected to the instance, update the package list and upgrade installed packages:

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

## Step 4: Install Basic Packages

Install Python, pip, virtual environment, and Nginx:

```bash
sudo apt install python3-pip python3-venv nginx -y
```

## Step 5: Setup Git

Follow the guide: [Login & Setup Git in Server](git-related/login-and-setup-git-in-server.md)

This includes:
- Registering username and email
- Creating SSH key to link with GitHub
- Connecting GitHub with the SSH key
- Cloning the repository with SSH

### Quick Git Setup Commands

After following the linked guide, you should have:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

**Note:** Replace `Your Name` and `your-email@example.com` with your actual details.

## Step 6: Clone Your Repository

Clone your Django project repository:

```bash
cd ~
git clone git@github.com:username/repository.git
cd repository
```

**Note:** Replace `username/repository.git` with your actual GitHub username and repository name.

## Step 7: Create Virtual Environment

Create and activate a Python virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

Your prompt should now show `(venv)` indicating the virtual environment is active.

## Step 8: Install Requirements

Install all Python dependencies from your requirements file:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

**Note:** Ensure your `requirements.txt` includes `gunicorn` and `django`. If not, add them:

```bash
pip install gunicorn django
```

## Step 9: Configure Django Settings

### Update ALLOWED_HOSTS

Edit your Django `settings.py` file:

```bash
nano your_project/settings.py
```

Add your server's IP address and domain name to `ALLOWED_HOSTS`:

```python
ALLOWED_HOSTS = ['your-server-ip', 'your-domain.com', 'localhost']
```

**Note:** Replace `your-server-ip` and `your-domain.com` with your actual values.

### Configure Static Files

Ensure your `settings.py` has:

```python
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

## Step 10: Run Database Migrations

Create and apply database migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

## Step 11: Create Superuser

Create a Django superuser account:

```bash
python manage.py createsuperuser
```

Follow the prompts to set up your admin account.

## Step 12: Collect Static Files

Collect all static files into the `staticfiles` directory:

```bash
python manage.py collectstatic --noinput
```

## Step 13: Test Development Server

Test that your Django application runs correctly:

```bash
python manage.py runserver 0.0.0.0:8000
```

Visit `http://your-server-ip:8000` in your browser to verify the application works. Press `Ctrl+C` to stop the server.

**Note:** If you encounter any issues, resolve them before proceeding to production setup.

## Step 14: Test Gunicorn

Test that Gunicorn can serve your Django application:

```bash
gunicorn --bind 0.0.0.0:8000 your_project.wsgi:application
```

**Note:** Replace `your_project` with your actual Django project name (the directory containing `wsgi.py`).

Visit `http://your-server-ip:8000` to verify. Press `Ctrl+C` to stop.

## Step 15: Create Gunicorn Socket File

Create a systemd socket file for Gunicorn. See the configuration file: [gunicorn.socket](config-files/gunicorn.socket)

Create the file:

```bash
sudo nano /etc/systemd/system/gunicorn.socket
```

Copy the contents from the linked configuration file and adjust the paths as needed.

## Step 16: Create Gunicorn Service File

Create a systemd service file for Gunicorn. See the configuration file: [gunicorn.service](config-files/gunicorn.service)

Create the file:

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Copy the contents from the linked configuration file and adjust:
- Project path
- Virtual environment path
- User and group
- Project name

## Step 17: Start and Enable Gunicorn

Start the Gunicorn socket and service:

```bash
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl start gunicorn.service
sudo systemctl enable gunicorn.service
```

Check the status:

```bash
sudo systemctl status gunicorn.socket
sudo systemctl status gunicorn.service
```

## Step 18: Configure Nginx

### Create Nginx Configuration File

See the configuration file: [nginx.conf](config-files/nginx.conf)

Create the Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/your_project
```

Copy the contents from the linked configuration file and adjust:
- Server name (domain or IP)
- Project paths
- Static files location

### Enable the Site

Create a symbolic link to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/your_project /etc/nginx/sites-enabled/
```

**Note:** Replace `your_project` with your actual project name.

### Test Nginx Configuration

Test that the Nginx configuration is valid:

```bash
sudo nginx -t
```

### Restart Nginx

If the test is successful, restart Nginx:

```bash
sudo systemctl restart nginx
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

## Step 19: Configure Firewall

Allow HTTP and HTTPS traffic through the firewall:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable
```

Check firewall status:

```bash
sudo ufw status
```

## Step 20: Verify Deployment

1. Visit `http://your-server-ip` or `http://your-domain.com` in your browser
2. Your Django application should be accessible
3. Test the admin panel at `http://your-server-ip/admin`

## Troubleshooting

### Check Gunicorn Logs

```bash
sudo journalctl -u gunicorn.socket
sudo journalctl -u gunicorn.service
```

### Check Nginx Logs

```bash
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/nginx/access.log
```

### Check Gunicorn Socket

```bash
sudo systemctl status gunicorn.socket
file /run/gunicorn.sock
```

### Restart Services

If you make changes to configuration files:

```bash
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
sudo systemctl restart nginx
```

### Permission Issues

If you encounter permission issues:

```bash
sudo chown -R www-data:www-data /path/to/your/project
sudo chmod -R 755 /path/to/your/project
```

**Note:** Replace `/path/to/your/project` with your actual project path.

## Security Considerations

- Change default SSH port (optional but recommended)
- Set up SSL/TLS certificates using Let's Encrypt
- Regularly update system packages: `sudo apt-get update && sudo apt-get upgrade`
- Use environment variables for sensitive settings (SECRET_KEY, database credentials)
- Configure proper firewall rules
- Use strong passwords for superuser accounts
- Consider using PostgreSQL instead of SQLite for production

## Next Steps

- Set up SSL/TLS certificates
- Configure domain name DNS settings
- Set up automated backups
- Configure log rotation
- Set up monitoring and alerts
- Consider using a process manager like Supervisor for additional control

