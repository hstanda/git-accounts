# Setup Multiple Git Accounts with Directories on a Single Machine
## Introduction
- Multiple projects for each git account
- Each  git account is assigned to one directory
- Add multiple projects to a designated directory
- Once set simple git commands in each directory with no extra arguments are needed
- Any Git repository should work ( I have tested on GitHub and GitLab )
- I am adding examples of two git accounts
- these commands are executed on Linux

## Setup summary
- [Set up SSH keys for multiple accounts](#step-1-create-ssh-keys)
- [Add keys to the SSH agent](#step-2-add-keys-to-the-ssh-agent)
- [Add SSH keys to your Git account](#step-3-add-ssh-keys-to-your-git-account)
- [Configure SSH for multiple accounts](#step-4-configure-ssh-for-multiple-accounts)
- [Test SSH connection](#step-5-test-ssh-connection)
- [Configure Git user details](#step-6-configure-git-user-details)
- [Update global Git configuration](#step-7-update-global-git-configuration)
- [Verify configurations and file names](#step-8-verify-configurations-and-file-names)
- [Linux Troubleshoot](#linux-troubleshoot)

## Prerequisites
- Basic knowledge of Git and command-line interface (CLI)
- Git installed on your machine
- The .ssh directory is at the root directory for the logged-in user
- knowledge using the command line

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
- `git-account1-email@gmail.com` is an email on the git repository account

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
Print key content on the terminal with the cat command
```bash
cat ~/.ssh/git-account2-ssh-file-name
```
- Copy the content of the public key file and add it to your respective Git account (GitHub, GitLab, etc.). Refer to [GitHub's documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) for instructions.
- Make sure you are adding keys to the correct account

### Step 4: Configure SSH for Multiple Accounts
The ~/.ssh/config file is used to configure per-user SSH client settings. By creating entries in this file, you can specify various SSH options and settings for different hosts or groups of hosts, making it easier to manage multiple SSH connections.
- Edit or create the SSH config file (~/.ssh/config):
- on executing cat command `cat ~/.ssh/config` config file should look like this

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
- `HostName`: is the repository cloud base URL.
- `IdentityFile`: Specifies the file containing the private key to use for authentication. This points to the specific SSH key file for each GitHub account (~/.ssh/git-account1-ssh-file-name for the first account and ~/.ssh/git-account2-ssh-file-name for the second).
- `User`: Specifies the username to use when logging in. For GitHub, this is typically git.
- `IdentitiesOnly`: When set to yes, this option ensures that only the specified IdentityFile (and not any other identities in the agent) will be used for authentication.
- if does not exist create the file in the .ssh directory
- in this file, we are assigning a GitHub account to its SSH key
- For example `gitHub-account-1` is being assigned to `git-account1-ssh-file-name` key which was created in Step 1

### Step 5: Test SSH Connection
```bash
ssh -T github-account-1
ssh -T github-account-2
```
- `ssh`: This command is used to initiate an SSH (Secure Shell) connection to a remote host.
- `-T`: This option disables pseudo-terminal allocation. It is typically used when you want to execute a command on the remote server without opening an interactive shell.
- `github-account-1`: This is the alias defined in your ~/.ssh/config file. It specifies which host configuration to use for this connection.

### Step 6: Configure Git User Details
- Here we are creating two configuration files for each git account
- Config files for this example
```bash
nano ~/.github-account1-config
```
content of the `.github-account1-config` file should look like this
```bash
[user]
   name = account1
   email = git-account1-email@gmail.com
[core]
   sshCommand = "ssh -i ~/.ssh/git-account1-ssh-file-name"
```
```bash
nano ~/.github-account2-config
```
content of the `.github-account2-config` file should look like this
```bash
[user]
   name = account2
   email = git-account2-email@gmail.com
[core]
   sshCommand = "ssh -i ~/.ssh/git-account2-ssh-file-name"
```
- The [user] section sets the global Git username and email address.    These settings are used for commits and other Git operations.
- [core] Sets the SSH command that Git should use when connecting to remote repositories.
   The -i option specifies the identity file (private SSH key) for authentication.
   ~/.ssh/git-account1-ssh-file-name is the SSH private key file path for the specific GitHub account (git-account1).
  
### Step 7: Update Global Git Configuration
This configuration is used to conditionally include other configuration files based on the directory of the Git repository you're working in.

Edit the global Git configuration file (~/.gitconfig):
```bash
[includeIf "gitdir:~/Desktop/account1-work-directory/"]
   path = ~/.github-account1-config


[includeIf "gitdir:~/Desktop/account1-work-directory/"]
   path = ~/.github-account2-config
```
IMPORTANT
- `Desktop/account1-work-directory/` is the path to the respective repository for each git account
- The `.gitconfig` file is in the root directory for account1
- `.github-account1-config` config path for account1

### Step 8: Verify configurations and file names
```bash
cat ~/.gitconfig
cat ~/.ssh/config
cat ~/.github-account1-config
cat ~/.github-account2-config
ls -la ~/.ssh/
```

### Linux Troubleshoot
- ERROR: “ssh: Could not resolve hostname github.com: Temporary failure in name resolution”
   Solution/Command: “sudo systemctl restart systemd-resolved”
- Initial the ssh-agent
   Solution/Command: eval "$(ssh-agent -s)"
- Test the connection to the host added on the config file
   ssh -T github-account-1
   ssh -T github-account-2
- Permissions 0644 for `account1-public-key-name` are too open. It is required that your private key files are NOT accessible by others.
   chmod 600 ~/.ssh/account2-public-key-name.pub
- When creating multiple files it's possible to miss the names and location of the file. Check all file names and content following step 8