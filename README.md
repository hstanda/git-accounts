# Setup Multiple Git Accounts with Directories on a Single Machine
## Introduction
- Multiple projects for each git account
- Each  git account assigned to one directory
- Add multiple projects to desginated directory
- Once setup simple git commands in each directory with no extra arguments needed
- Any git repository should work ( I have tested on GitHub and GitLab )
- I am adding example of two git accounts
- these commands are executed on Linux

## Setup summery
- setup ssh keys for multiple accounts
- Add Keys to the SSH Agent
- Add SSH Keys to Your Git Account
- Configure SSH for Multiple Accounts
- Test SSH Connection
- Configure Git User Details
- Update Global Git Configuration
- Verify Configurations
- Troubleshooting setup

## Prerequisites
- Basic knowledge of Git and command-line interface (CLI)
- Git installed on your machine
- .ssh directory is at root directory for logged in user 
- knowledge using commandline

### Step 1: Create SSH Keys
Navigate to the SSH directory:
```bash
cd ~/.ssh
```
Create keys for both accounts
```bash
ssh-keygen -t rsa -C "git-account1-email@gmail.com" -f "git-account1-ssh-file-name"
ssh-keygen -t rsa -C "git-account2-email@gmail.com" -f "git-account2-ssh-file-name"
```
- `git-account1-ssh-file-name` is SSH key names
- `git-account1-email@gmail.com` is email on git repository account

### Step 2: Add Keys to the SSH Agent
Before adding the key, ensure the SSH agent is running. You can start it with

```bash
eval "$(ssh-agent -s)"
```
Add the Private Key to the SSH Agent

```bash
ssh-add ~/.ssh/git-account1-ssh-file-name
ssh-add ~/.ssh/git-account2-ssh-file-name
```
- `ssh-add` This is the command used to add SSH private keys to the SSH authentication agent.
- The SSH agent keeps your private key secure and uses it to authenticate connections without exposing the key.
- Once the key is added, you won’t need to enter the passphrase for each SSH connection.

### Step 3: Add SSH Keys to Your Git Account
Print key content on the terminal with cat command
```bash
cat ~/.ssh/git-account2-ssh-file-name
```
- Copy the content of the public key file and add it to your respective Git account (GitHub, GitLab, etc.). Refer to [GitHub's documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) for instructions.
- Make sure you are adding keys to correct account

### Step 4: Configure SSH for Multiple Accounts
The ~/.ssh/config file is used to configure per-user SSH client settings. By creating entries in this file, you can specify various SSH options and settings for different hosts or groups of hosts, making it easier to manage multiple SSH connections.
- Edit or create the SSH config file (~/.ssh/config):
- on excuting cat command `cat ~/.ssh/config` config file should look like this

#### create/update ~/.ssh/config file
```bash
# GitHub Account 1
Host github-account-1
    HostName github.com
    User git
    IdentityFile ~/.ssh/git-account1-ssh-file-name
    IdentitiesOnly yes

# GitHub Account 2
Host github-account-2
    HostName github.com
    User git
    IdentityFile ~/.ssh/git-account2-ssh-file-name
    IdentitiesOnly yes
```
- `Host`:  is the alias you will use when making an SSH connection to this GitHub account.
- `HostName`: is repository cloud base URL.
- `IdentityFile`: Specifies the file containing the private key to use for authentication. This points to the specific SSH key file for each GitHub account (~/.ssh/git-account1-ssh-file-name for the first account and ~/.ssh/git-account2-ssh-file-name for the second).
- `User`: Specifies the username to use when logging in. For GitHub, this is typically git.
- `IdentitiesOnly`: When set to yes, this option ensures that only the specified IdentityFile (and not any other identities in the agent) will be used for authentication.
- if does not exist create the file in .ssh directory
- in this file we are assigning github account to its ssh key
- example `GitHub Account 1` is being assigned to `git-account1-ssh-file-name` key which was created in step 1

### Step 5: Test SSH Connection

```bash
ssh -T github-account-1
ssh -T github-account-2
```
- `ssh`: This is the command used to initiate an SSH (Secure Shell) connection to a remote host.
- `-T`: This option disables pseudo-terminal allocation. It is typically used when you want to execute a command on the remote server without opening an interactive shell.
- `github-account-1`: This is the alias defined in your ~/.ssh/config file. It specifies which host configuration to use for this connection.