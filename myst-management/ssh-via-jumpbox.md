Are there servers firewalled (or in a DMZ) which cannot be SSH'd directly? These servers are usually public facing such as OHS and can only be connected via a jumpbox. Below are examples of configuring SSH tunnels.

Note that the SSH configuration below is for the `adminserver` because Myst connects to hosts via the AdminServer.



# Prerequisite

### Check SSH Version

Check your SSH version by running as certain commands have compatibility requirements.

```shell
ssh -V
```

### Setup SSH Keys for Passwordless Login

Obviously we need all SSH connections to not have a prompt otherwise you'd be entering a password for each SSH jump.

These steps assume you don't have an SSH key. If you already have a key then skip steps 1 and 2.

1. Generate the private and public keys
   1. SSH to the `adminserver` as the `oracle` user
   2. `ssh-keygen -t rsa -b 4096`
   3. Leave the defaults by pressing the `Enter` key
2. Copy the public key to the **Jumpbox**
   1. `ssh-copy-id oracle@adminserver`
   2. Enter the password for the `oracle` user
   3. ***NOTE:*** If the above command does not work due to `ssh-copy-id` not existing then try:
      `cat ~/.ssh/id_rsa.pub | ssh oracle@jumpbox "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"`
3. Copy the public key to the **DMZ Host1**
   1. SSH to the **Jumpbox** as the `oracle` user
   2. `ssh-copy-id oracle@jumpbox`
   3. Enter the password for the `oracle` user
   4. ***NOTE:*** If the above command does not work due to `ssh-copy-id` not existing then try:
      `cat ~/.ssh/id_rsa.pub | ssh oracle@dmzhost1 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"`



# SSH Jump Methods

Update or create the `oracle` user's `$HOME/.ssh/config` with **one** of these methods.

### ProxyJump (OpenSSH 7.3+)

```
Host dmzhost1
    ProxyJump adminserver,jumpbox
```

### ProxyCommand - Single

Use this method if running a version lower than `OpenSSH 7.3`.

```
Host dmzhost1
    ProxyCommand ssh adminserver -A ssh jumpbox -W %h:%p
```

### ProxyCommand - Multiple

Use this method if the `single` method does not work.

```
Host dmzhost1
    ProxyCommand ssh jumpbox -W %h:%p

Host jumpbox
    ProxyCommand ssh adminserver -W %h:%p
```



# Validate SSH jump

Test the SSH command.

1. SSH to the `adminserver` and the `oracle` user
2. Run the command to validate
    ```
    ssh dmzhost1
    ```

