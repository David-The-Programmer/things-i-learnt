# Setting up SSH

This is a reference/guide to setting up SSH between 2 Arch Linux systems.

SSH or Secure Shell is a network protocol used to access the shell of computer A via another computer B. The computer being accessed, computer B, is known as the SSH server or host. The computer used to access computer B, computer A, is known as the SSH client.

Firstly we setup the SSH client by doing:

1. Creation of a new entry in KeePassXC for the SSH client key
2. Generation of SSH client key
3. Addition of generated SSH client key to entry in KeePassXC
4. Set up of the `ssh-agent` service on the SSH client

Secondly we setup the SSH server/host by doing:

5. Copying generated SSH client public key to SSH server
6. Testing SSH login
7. Disabling password authentication on the SSH server

## Prerequisites

Packages that need to be installed:
- [KeePassXC](https://github.com/keepassxreboot/keepassxc) on the SSH client
- [OpenSSH](https://archlinux.org/packages/core/x86_64/openssh/) on both SSH client and host/server

```sh
yay -S keepassxc openssh
```

KeePassXC is an offline password manager that stores passwords as an encrypted file. It is also capable of storing ssh keys as well. We would store the SSH client private key there to ensure it is encrypted and not stored as plain text, which would not be secure.

## 1. Creation of new entry in KeePassXC for SSH client key
1. On the SSH client, go into the KeePassXC GUI and create a new entry. The title of the entry should be: `SSH Client Key`.
2. Leave all other fields blank.
3. Save the entry by clicking the `Ok` button.

## 2. Generation of SSH Client Keys
1. On the SSH client, run the following command: `ssh-keygen -t ed25519 -C "SSH Client Key"`
2. When prompted about the location to save the SSH keys, press enter to accept the default location shown. The keys will be shredded later.
3. When prompted about the passphrase, press enter. There is no need for a passphrase as the private key will be stored in the encrypted KeePassXC database.

## 3. Addition of generated SSH client key to KeePassXC entry
1. On the SSH client, go to KeepassXC, double click on the entry to edit. 
2. Go to the `Advanced` menu on the left. Go to the `Attachments` section. Click on `Add` to add the **private** key. **The private key is the SSH client key**. By default, the location of the key would be `~/.ssh/id_ed25519`.
3. Go to the `SSH Agent` menu on the left. 
4. Select the options: 
    - `Add key to agent when the database is opened/unlocked`
    - `Remove key from agent when the database is closed/locked`
    
    Having these options enabled will ensure that the key is cached the `ssh-agent` only when the database was unlocked by the user and removed when the database is locked. This prevents unauthorised usage of the key by the `ssh-agent`. 
5. Do not enable the option `Require user confirmation when this key is used`.
6. In the `SSH Agent Menu`, ensure the key file attached previously is selected in the dropdown menu for `Attachment`.
7. Click on the `Add to Agent` button to ensure that the key is added to the `ssh-agent` for the first time.
8. Click on `Ok` to save the edits to the entry.
9. Ensure that the comment shown in the `SSH Agent` menu matches the one written when generating the SSH key, which if you follow this guide would be "SSH Client Key".

## 4. Set up of the `ssh-agent` service
1. On the SSH client, create a file in `~/.config/systemd/user/ssh-agent.service`. The file should have the content below.
    ```
    [Unit]
    Description=SSH key agent
    
    [Service]
    Type=simple
    Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
    ExecStart=/usr/bin/ssh-agent -D -a $SSH_AUTH_SOCK
    
    [Install]
    WantedBy=default.target
    ```
2. Add `SSH_AUTH_SOCK="${XDG_RUNTIME_DIR}/ssh-agent.socket"` to `~/.config/environment.d/ssh_auth_socket.conf`
3. Run the command `systemctl --user enable --now ssh-agent` to enable the `ssh-agent` service.
4. Reboot for the configurations above to take effect.
5. Running `ssh-add -l` should output `The agent has no identities.` when the KeePassXC database is locked. When it is unlocked, `ssh-add -l` should output the SSH key fingerprint, as shown in under `Fingerprint` in the `SSH Agent` menu of the KeePassXC entry.


## 5. Copying generated SSH client public key to SSH server
1. On the SSH client, run the following command to send the public key to the SSH host/server:
    ```
    ssh-copy-id -i ~/.ssh/id_ed25519.pub <username>@<remote_host/server>
    ```
    Note that the flag `-i` should have the path to the SSH client **public key file, ending with .pub and not the private one**.

    When prompted if you want to continue, type `yes` and press the Enter key.

    You should then be prompted to enter the SSH server/host password. Subsequently, there should be an output saying that 1 key was added.

    The public key file by default would be added as a file at `~/.ssh/authorized_keys` in the SSH server/host.

2. Now that the private key file is added and securely stored in the KeePassXC database and public key has been sent to the SSH server/host, we can shred both keys on the SSH client by running: `shred -vzu -n5 ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub`. This will ensure that the key files are not stored in plain text, which is a highly unsecure way to store them.

## 6. Testing SSH Login
1. On the SSH client, ensure you unlocked the KeePassXC database first.
2. Run the `ssh` commmand to login to the SSH server via the SSH client.
    ```
    ssh <username>@<remote_host/server>
    ```
   If prompted if you want to continue, type `yes` and press the Enter key.
3. Now you should be logged into the SSH server/host.

## 7. Disabling password authentication on the SSH server

Disabling password authentication or password based logins prevents malicious users from login via bruteforce. Since we have enabled SSH via SSH keys, we do not require password authentication anymore. This also allows us to login to the SSH server/host with greater convenience, without having to type in the password of the SSS host everytime.

Password authentication can be disabled by:

1. On the SSH server/host, open the SSH config file found at `/etc/ssh/sshd_config` with `sudo vim`.
2. Uncomment `PasswordAuthentication` and set its value to `no`.
3. Save and close the file. Restart the SSH daemon by running `sudo systemctl restart sshd`.

**Note the config file is `sshd_config`, not `ssh_config`. Take note of the `d`**.

Test the disabling of password authentication by running the `ssh` command after locking the KeepassXC database. You should get the following output:
```
<user>@<remote-host>: Permission denied (publickey)
```
Unlock the KeepassXC database, and run the `ssh` command again. You should be able to successful log into the SSH server/host.


## Useful links

https://peterbabic.dev/blog/make-ssh-prompt-password-keepassxc/

https://www.padok.fr/en/blog/ssh-keys-keepassxc

https://ferrario.me/using-keepassxc-to-manage-ssh-keys/

https://www.freecodecamp.org/news/securely-erasing-a-disk-and-file-using-linux-command-shred

https://unix.stackexchange.com/questions/339840/how-to-start-and-use-ssh-agent-as-systemd-service

https://github.com/keepassxreboot/keepassxc/issues/7718

https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server
