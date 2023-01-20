# Provision Workstation Through Ansible

An ansible playbook to help install desktops and laptops.

## Usage

### Initial Arch Install
After booting from the Arch installation media, you will need to do the following in preparation for `ansible`:

- Set the root password using `passwd`.  Set the password to `root`.  Don't worry, this is temporary.
- Restart the ssh service.

```systemctl restart sshd```

- If installing through wireless, run `iwctl` to [set up wifi](https://wiki.archlinux.org/title/Iwd#iwctl).  You can use `ip link` to determine your wireless device.

```iwctl station $DEVICE connect $SSID```

🗒️ **NOTE**: The wireless device in the boot media may be different than after your installation.

- Ensure your net interfaces and install drives/partitions are properly set in your host's inventory file.  You can find this through `ip link` and `lsblk`.
- Make sure your network is connected (e.g. `ping google.com`)

At this point we are able to login remotely as root, so run the initialization playbook.

⚠️ **WARNING:** This will obliterate the data on this machine.  The initialization playbook is **NOT** idempotent.

```bash
ansible-playbook ansible/playbooks/arch-initial-install.yml --limit $DEVICE_NAME
```

### Provisioning

After the initial arch installation, you should have a system with the absolute basics, such as:

- Console login account with non-root username and password
- The most basic libraries needed to run the regular ansible provisioning

Go to your source directory on that machine (defined in your inventory host variables) and run the provisioning playbook.

```bash
ansible-playbook ansible/playbooks/site.yml --limit _$DEVICENAME_
```

🗒️ **NOTE:** Unlike the initial arch install above, this playbook **is** idempotent and can be safely executed over and over.

Although the provisioning requires `sudo` into `root`, all routines are ran locally, so there is no need to expose a SSH server on your workstation.

