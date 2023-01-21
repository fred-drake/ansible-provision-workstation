# Provision Workstation Through Ansible

An ansible playbook to help install desktops and laptops.

## Usage

### Initial Arch Install
After booting from the Arch installation media, you will need to do the following in preparation for `ansible`:

- Set the root password using `passwd`.  Set the password to `root`.  Don't worry, this is temporary.
- Restart the ssh service.

```bash
$ systemctl restart sshd
```

- If installing through wireless, run `iwctl` to [set up wifi](https://wiki.archlinux.org/title/Iwd#iwctl).  You can use `ip link` to determine your wireless device.

```bash
$ iwctl station $DEVICE connect $SSID
```

🗒️ **NOTE**: The wireless device in the boot media may be different than after your installation.

- Ensure your net interfaces and install drives/partitions are properly set in your host's inventory file.  You can find this through `ip link` and `lsblk`.
- Make sure your network is connected (e.g. `ping google.com`)

At this point we are able to login remotely as root, so run the initialization playbook.

⚠️ **WARNING:** This will obliterate the data on this machine.  The initialization playbook is **NOT** idempotent.

```bash
$ ansible-playbook ansible/playbooks/arch-initial-install.yml --limit $DEVICE_NAME
```

Remove your USB boot drive during this process, as it is no longer needed.  Upon completion, the workstation will reboot.

If there is an issue and you need to re-run this, the first thing it will do is wipe the install device, so it always begins from complete scratch.

### Provisioning

After the initial arch installation, you should have a system with the absolute basics, such as:

- Console login account with non-root username and password
- Your AGE personal secret key tucked into the `~/.age` directory
- The basic libraries needed to run the remaining ansible provisioning

The drive is encrypted with LUKS, so you will be prompted for your passphrase (set in the secrets YAML file) upon boot.

The idempotent provisioning from here on out will take place directly on your workstation.
Go to your source directory on that machine, run the `direnv` commands shown below to access the `.envrc` variables, and run the provisioning playbook.

```bash
$ cd ~/Source/github/$GITHUB_USERNAME/ansible-provision-workstation
$ direnv allow .
$ ansible-playbook ansible/playbooks/site.yml --limit $DEVICE_NAME
```

🗒️ **NOTE:** Unlike the initial arch install above, this playbook **is** idempotent and can be safely executed over and over.

Although the provisioning requires `sudo` into `root`, all routines are ran locally, so there is no need to expose a SSH server on your workstation.

