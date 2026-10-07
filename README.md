# Linux Server Administration Lab

Built and secured an Ubuntu Server VM: users and permissions, SSH key authentication, UFW firewall, and automated Bash backups with cron.

## Tools
Ubuntu Server 24.04 LTS, UTM (virtualization on macOS), SSH, UFW, Bash, cron

## What I Did

### 1. Built the server
Installed Ubuntu Server in a UTM virtual machine on my Mac, updated all packages, and managed it from my Mac's Terminal over SSH.

### 2. Users, groups, and permissions
- Created a user (`jsmith`) and a `helpdesk` group, and added the user to the group
- Practiced help desk tasks: password resets and locking/unlocking accounts
- Created a shared folder (`/srv/helpdesk`) with `chmod 770` so only the helpdesk group can use it
- Tested it: the group member could create files, and my account (not in the group) got "Permission denied"
<img width="1738" height="224" alt="1-permissions-test" src="https://github.com/user-attachments/assets/53374f20-5899-4cb2-afd1-d969003fba72" />


### 3. Secured SSH access
- Set up SSH key authentication from my Mac
- Disabled password logins so the server only accepts my key
- Tested it: a forced password login was refused, and a key login worked
<img width="1738" height="201" alt="2 ssh key only test" src="https://github.com/user-attachments/assets/0a2bdc7e-0a6a-4f56-8a83-b7fb56eeefe3" />


### 4. Firewall
Enabled UFW to deny all incoming traffic except SSH (port 22).
<img width="1738" height="457" alt="3 firewall status" src="https://github.com/user-attachments/assets/cd1c932a-ae65-4568-9638-0a847372c2c3" />



### 5. Automated backups
Wrote a Bash script that compresses the shared folder into a timestamped backup, logs whether it worked, and deletes backups older than 7 days. Scheduled it to run nightly with cron.
<img width="1738" height="260" alt="backup and cron" src="https://github.com/user-attachments/assets/1e981698-8d26-404a-8da1-d13f25d56146" />



## Problems I Hit and How I Fixed Them

### 1. SSH wouldn't restart after a config change
**Error:**
```
Job for ssh.service failed because the control process exited with error code.
/etc/ssh/sshd_config.d/50-cloud-init.conf line 1: no argument after keyword "PasswordAuthentication"
```
**Cause:** While disabling password logins, I deleted `yes` but never typed `no`, leaving the setting blank. SSH refuses to start with an invalid config.

**Fix:** I rewrote the line as `PasswordAuthentication no`, tested the config with `sudo sshd -t`, and restarted SSH.

**Lesson:** Always test the config before restarting, and keep a second session open. Mine kept me from getting locked out.

### 2. "Missing privilege separation directory"
**Error:**
```
Missing privilege separation directory: /run/sshd
```
**Cause:** Because SSH had crashed, its temporary runtime folder had been cleaned up.

**Fix:** Restarting the service recreated it, and SSH came back as `active (running)`.

### 3. SSH key saved under the wrong name
**Cause:** When `ssh-keygen` asked where to save the key, I typed my next command instead of pressing Enter, so the key was saved to a file named after that command.

**Fix:** I deleted the misnamed files (using quotes, since the name had spaces), regenerated the key in the default location, and ran `ssh-copy-id` separately.

**Lesson:** When a command is asking a question, whatever you type becomes the answer.

### 4. Typo in a group name
**Error:**
```
chown: invalid group: 'root:hekpdesk'
```
**Fix:** Linux told me the group didn't exist. I spotted the typo and reran it with `helpdesk`.

## What I Learned
I learned how groups control access. I got to be on both sides of "Permission denied": the user in the group could create files, and my account outside the group was blocked. I also learned that error messages are helpful, because they showed me exactly what was missing. Finally, I learned that an SSH key has two halves: a private key that never leaves my Mac, and a public key that goes on the server so it can verify it's really me.

## How I Learned
I used Claude (an AI assistant) as a tutor while building this lab. It explained concepts and gave me step-by-step guidance, but I ran every command myself and troubleshot my own errors. Using AI to learn faster and solve problems is a skill I plan to keep using on the job.
