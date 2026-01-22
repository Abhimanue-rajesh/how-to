# Login and Setup of Git and GitHub in AWS Instance

This guide walks you through setting up Git and GitHub authentication using SSH keys on an AWS instance.

## Prerequisites

- Access to an AWS instance via console/SSH
- A GitHub account

## Step 1: Generate SSH Key Pair

Enter the following command in the console of the instance to generate an SSH key pair:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

**Note:** Replace `your-email@example.com` with your actual email address.

When prompted, enter a passphrase (optional but recommended for security).

## Step 2: Start the SSH Agent

Start the SSH agent using the following command:

```bash
eval "$(ssh-agent -s)"
```

## Step 3: Add the SSH Key

Add the generated key to the SSH agent:

```bash
ssh-add ~/.ssh/id_ed25519
```

**Note:** This command will prompt you for the passphrase if you set one during key generation.

## Step 4: Retrieve Your SSH Public Key

Display your SSH public key using:

```bash
cat ~/.ssh/id_ed25519.pub
```

Your SSH public key will look similar to this (this is just an example - your key will be different):

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOqlsj986Er9ob9u5DPIohmxPHsmxkgwauYtOJFAyOzG your-email@example.com
```

## Step 5: Add SSH Key to GitHub

1. Go to your GitHub account
2. Navigate to **Settings** > **SSH and GPG keys**
3. Click **New SSH key**
4. Paste your SSH public key
5. Give it a descriptive name (include the instance name in the naming for easy identification)
6. Click **Add SSH key**

## Step 6: Test GitHub Connection

Test the connection to GitHub from your instance:

```bash
ssh -T git@github.com
```

You should see a success message confirming the connection.

## Step 7: Clone Repository

Once the connection is verified, you can clone repositories using SSH:

```bash
git clone git@github.com:username/repository.git
```

**Note:** Replace `username/repository.git` with your actual GitHub username and repository name.

## Troubleshooting

- If you encounter permission issues, ensure the SSH key has the correct permissions: `chmod 600 ~/.ssh/id_ed25519`
- If the SSH agent isn't running, restart it using `eval "$(ssh-agent -s)"` and add the key again
- Verify your SSH key is added to GitHub by checking the SSH and GPG keys section in your GitHub settings

