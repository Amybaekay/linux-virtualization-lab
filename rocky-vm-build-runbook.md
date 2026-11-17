# Rocky Linux Minimal VM — Build Runbook

**Purpose:** Build a Linux lab VM, set a hostname, create a normal user, verify SSH from the host and take a snapshot.

## 1. Build specification

| Component | Configuration |
|---|---|
| Host | 2015 MacBook Pro; macOS Monterey 12.7.6; Intel Core i7; 16 GB RAM |
| Hypervisor | VirtualBox 7.1.18 for Intel macOS |
| Installation ISO | Rocky Linux 9 Minimal, x86_64 ISO |
| VM / hostname / user | `rocky_lab` / `rockylab` / `amarachi` |
| Resources | 2 virtual CPUs; 2048 MB RAM; 25 GB dynamically allocated VDI |
| Network | Adapter 1: NAT; SSH forwarded from Mac port 2223 to guest port 22 |

## 2. Create the virtual computer — on my Mac
1. Open the official Rocky Linux download page. Select **Rocky Linux 9 → AMD/Intel (x86_64) → Minimal ISO**. Keep the downloaded file in Downloads. My first Rocky 10.2 download failed with “File wasn’t available on site”; I switched to the Rocky 9 Minimal download.
2. Open VirtualBox → **New**. Name the VM `rocky_lab`. Keep the default folder and choose the downloaded Rocky ISO under **ISO Image**. If asked for the operating system, select **Linux → Red Hat (64-bit)**, or **Red Hat 9 (64-bit)** if available.
3. Tick **Skip Unattended Installation** to install Rocky myself. Set **Memory: 2048 MB**, **Processors: 2**; leave **EFI** unticked.
4. Create a **25 GB** virtual hard disk. Keep **VDI** selected and **Pre-allocate Full Size** unticked: the disk uses space gradually. Click **Finish → Start**. At the boot menu, select **Install Rocky Linux 9** and press Enter.

## 3. Install Rocky — inside the VM window
The installer has a graphical interface even though the installed Minimal system uses a terminal.
1. Choose the language and click **Continue**. On **Installation Summary**, check the keyboard layout and time zone. Under **Software Selection**, confirm **Minimal Install**; **Installation Source** should show local media.
2. Open **Installation Destination**. Select the **25 GB VBOX HARDDISK**, keep **Automatic** storage configuration and click **Done**. This chooses the virtual disk for installation.
3. Open **Network & Host Name**. Switch the Ethernet connection **ON**, keep automatic IP settings, enter `rockylab` as the hostname, click **Apply** if shown, then **Done**.
4. Open **User Creation**. Enter my name and username `amarachi`. Tick **Make this user administrator** and **Require a password to use this account**. Enter and confirm a private password, then click **Done**. This is a normal account that can use `sudo` for administrator tasks.
5. Keep the **root account disabled** because my normal user has sudo access. Review the summary and resolve any remaining required warnings. Click **Begin Installation**.
6. When installation finishes, click **Reboot System**. If the installer starts again, disconnect the ISO using **Devices → Optical Drives → Remove disk from virtual drive**, then restart. Log in as `amarachi`. This is the installed system’s **first boot**; password typing is invisible.

## 4. Check and enable SSH — inside Rocky

My first command used Ubuntu’s service name `ssh` and returned `ssh.service does not exist`. Rocky’s service is called **`sshd`**. OpenSSH server was already installed; an installation check reported “already installed” and “Nothing to do”.

```bash
sudo systemctl enable --now sshd
systemctl is-active sshd       # Expected: active
sudo poweroff
```

`sudo` gives the command administrator permission. The first command starts SSH and enables it at future boots; the last shuts Rocky down before changing VirtualBox settings. If OpenSSH server is missing on a future build, install it with `sudo dnf install openssh-server`, then repeat the SSH commands.


## 5. Configure and test access — on my Mac

With Rocky powered off, open **rocky_lab → Settings → Network → Adapter 1 → Advanced → Port Forwarding**. Keep **NAT** and add:

| Name | Protocol | Host IP | Host port | Guest IP | Guest port |
|---|---|---|---|---|---|
| SSH | TCP | 127.0.0.1 | 2223 | Empty | 22 |

Save with **OK** in both windows. Start Rocky and wait for its login screen. In **macOS Terminal or VS Code’s terminal on my Mac**, run:

```bash
ssh -p 2223 amarachi@127.0.0.1
```

This sends the Mac’s connection on port **2223** to Rocky’s SSH port **22**. Ubuntu uses host port **2222**, so Rocky has its own port. For a new connection, the SSH fingerprint can be checked inside the Rocky VM with `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`. Compare it with the displayed fingerprint, type `yes` to accept, then enter my Rocky password.

**Observed result:** my Mac’s terminal showed `[amarachi@rockylab ~]$`; commands now ran inside Rocky. The first attempt returned **Connection refused**. After starting Rocky and retrying, the connection succeeded. Rocky must stay running during SSH access. For repeat checks, `whoami` should return `amarachi` and `hostname` should return `rockylab`.

## 6. Save the baseline

Run `sudo poweroff` from the Rocky SSH session; the connection closes. In VirtualBox, wait for **Powered Off**, select **rocky_lab → Snapshots → Take**. Save as **Rocky - Initial setup and SSH working**. Describe the hostname, normal user and successful SSH test using host port 2223.

**Observed result:** the named snapshot appeared in the list. **Current State (changed)** does not mean it failed. To roll back later, power off the VM, select the saved snapshot and choose **Restore**; changes made after the checkpoint can be lost. A snapshot is a checkpoint, not an independent backup.

**Completion:** installation, first boot, hostname, normal user, host SSH and snapshot verified.
