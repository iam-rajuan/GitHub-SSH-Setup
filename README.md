# GitHub SSH Setup Guide

This guide explains how to configure GitHub SSH access on Windows, Linux, and macOS for any Git repository that uses an SSH remote such as:

```text
git@github.com:your-username/your-repo.git
```

It is written for the common failure case:

```text
git@github.com: Permission denied (publickey).
```

That error means GitHub did not receive a valid SSH key for your account.

## Overview

The setup is the same on every operating system:

1. Check whether you already have a usable SSH key.
2. Create a dedicated GitHub SSH key if needed.
3. Add the public key to your GitHub account.
4. Configure SSH so `github.com` uses the correct key.
5. Test the connection.
6. Push normally from Git, VS Code, or any Git client.

## Recommended standard

Use these conventions unless you have a strong reason not to:

- Key type: `ed25519`
- Key filename: `id_ed25519_github`
- GitHub key type: `Authentication Key`
- SSH host entry: `github.com`

Using a dedicated GitHub key avoids conflicts with server keys, EC2 `.pem` files, and older personal SSH setups.

## GitHub-side step

No matter which operating system you use, after creating the key:

1. Copy the contents of your public key file.
2. Open `GitHub -> Settings -> SSH and GPG keys -> New SSH key`
3. Choose `Authentication Key`
4. Give it a recognizable title, such as `Office Laptop`, `MacBook Pro`, or `Ubuntu Workstation`
5. Paste the public key and save

## Windows

### 1. Check existing SSH files

In PowerShell:

```powershell
Get-ChildItem $HOME\.ssh
```

Look for files like:

- `id_ed25519`
- `id_ed25519.pub`
- `id_ed25519_github`
- `id_ed25519_github.pub`

If you only see server keys such as `.pem` files, create a dedicated GitHub key.

### 2. Create a GitHub SSH key

On many Windows systems, OpenSSH is installed but not added to `PATH`. If `ssh-keygen` is not recognized, use the full executable path.

Standard command:

```powershell
ssh-keygen -t ed25519 -C "your-email@example.com" -f $HOME\.ssh\id_ed25519_github
```

Fallback when `ssh-keygen` is not recognized:

```powershell
C:\Windows\System32\OpenSSH\ssh-keygen.exe -t ed25519 -C "your-email@example.com" -f $HOME\.ssh\id_ed25519_github
```

This creates:

- `C:\Users\YourUser\.ssh\id_ed25519_github`
- `C:\Users\YourUser\.ssh\id_ed25519_github.pub`

### 3. Copy the public key

```powershell
Get-Content $HOME\.ssh\id_ed25519_github.pub
```

Copy the full output and add it to GitHub.

### 4. Configure SSH

Edit:

`C:\Users\YourUser\.ssh\config`

Add:

```sshconfig
Host github.com
  HostName github.com
  User git
  IdentityFile C:\Users\YourUser\.ssh\id_ed25519_github
  IdentitiesOnly yes
```

### 5. Test the connection

If `ssh` is available in `PATH`:

```powershell
ssh -T git@github.com
```

If it is not:

```powershell
C:\Windows\System32\OpenSSH\ssh.exe -T git@github.com
```

Successful output looks like:

```text
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

### 6. Windows notes

- If `ssh-agent` is unavailable or requires Administrator privileges, you can still work without it by using `IdentityFile` in your SSH config.
- If normal SSH fails but this works:

```powershell
C:\Windows\System32\OpenSSH\ssh.exe -i $HOME\.ssh\id_ed25519_github -T git@github.com
```

then the key is valid and the problem is usually your SSH config.

## Linux

### 1. Check existing SSH files

In your terminal:

```bash
ls -la ~/.ssh
```

Look for key pairs such as:

- `id_ed25519`
- `id_ed25519.pub`
- `id_ed25519_github`
- `id_ed25519_github.pub`

### 2. Create a GitHub SSH key

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/id_ed25519_github
```

This creates:

- `~/.ssh/id_ed25519_github`
- `~/.ssh/id_ed25519_github.pub`

### 3. Set correct permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519_github
chmod 644 ~/.ssh/id_ed25519_github.pub
```

### 4. Copy the public key

```bash
cat ~/.ssh/id_ed25519_github.pub
```

Copy the full output and add it to GitHub.

### 5. Configure SSH

Edit:

`~/.ssh/config`

Add:

```sshconfig
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  IdentitiesOnly yes
```

Then protect the config file:

```bash
chmod 600 ~/.ssh/config
```

### 6. Optional: load the key into the SSH agent

Start the agent and add the key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_github
```

This is convenient, but not strictly required if your SSH config already points to the key correctly.

### 7. Test the connection

```bash
ssh -T git@github.com
```

Successful output:

```text
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

## macOS

### 1. Check existing SSH files

In Terminal:

```bash
ls -la ~/.ssh
```

### 2. Create a GitHub SSH key

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/id_ed25519_github
```

### 3. Configure SSH

Edit:

`~/.ssh/config`

Add:

```sshconfig
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
  AddKeysToAgent yes
  UseKeychain yes
  IdentitiesOnly yes
```

### 4. Add the key to the macOS agent and Keychain

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_github
```

If your macOS version does not support that flag, use:

```bash
ssh-add ~/.ssh/id_ed25519_github
```

### 5. Copy the public key

```bash
cat ~/.ssh/id_ed25519_github.pub
```

Copy the full output and add it to GitHub.

### 6. Test the connection

```bash
ssh -T git@github.com
```

Successful output:

```text
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

## Common verification commands

Use these checks after setup.

### Check the remote URL

```bash
git remote -v
```

You want to see SSH-style remotes such as:

```text
origin  git@github.com:your-username/your-repo.git (fetch)
origin  git@github.com:your-username/your-repo.git (push)
```

### Change an existing remote to SSH

```bash
git remote set-url origin git@github.com:your-username/your-repo.git
```

### Test SSH directly

```bash
ssh -T git@github.com
```

## Troubleshooting

### `Permission denied (publickey)`

Usually caused by one of these:

- The public key was not added to the correct GitHub account
- SSH is using the wrong private key
- `~/.ssh/config` does not point `github.com` to the correct key
- File permissions on `~/.ssh` are too open on Linux or macOS

### The key works only with `-i`

If this works:

```bash
ssh -i ~/.ssh/id_ed25519_github -T git@github.com
```

but this fails:

```bash
ssh -T git@github.com
```

then your SSH config is missing, incorrect, or being overridden by another `Host github.com` block.

### `ssh-keygen` is not recognized on Windows

Use:

```powershell
C:\Windows\System32\OpenSSH\ssh-keygen.exe
```

and:

```powershell
C:\Windows\System32\OpenSSH\ssh.exe
```

### `ssh-agent` is not running on Windows

You can often continue without it if your SSH config includes:

```sshconfig
IdentityFile C:\Users\YourUser\.ssh\id_ed25519_github
IdentitiesOnly yes
```

### Multiple keys exist in `~/.ssh`

This is common and not a problem by itself. The safest pattern is:

- keep server keys and GitHub keys separate
- use a dedicated GitHub key filename
- define `Host github.com` explicitly in `~/.ssh/config`

## Quick setup recap

If you want the shortest possible repeatable flow:

1. Generate a dedicated GitHub key.
2. Copy the `.pub` file contents into GitHub SSH keys.
3. Add a `Host github.com` block to your SSH config.
4. Run `ssh -T git@github.com`
5. Confirm the success message.
6. Push normally.

## Example success message

```text
Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.
```

Once you see that message, GitHub SSH authentication is working.
