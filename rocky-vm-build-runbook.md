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


---

## Recent update — Host-only networking and recovery 

### Connect the two VMs — in VirtualBox on my Mac

I added a shared host-only network so Ubuntu and Rocky could reach each other. **Adapter 1 stayed as NAT** for internet access and the original SSH forwarding rules.

1. Shut down both VMs using `sudo poweroff` inside each VM.
2. Open **VirtualBox Manager → Tools → Network → Host-only Networks**. An existing network called **HostNetwork** was available, with subnet mask `255.255.255.0`.
3. For each powered-off VM, open **Settings → Network → Adapter 2**. Tick **Enable Network Adapter**, choose **Host-only Network**, select **HostNetwork** and keep **Cable Connected** ticked. Click **OK**.
4. Start both VMs, log in and run `ip -br address` inside each one. This shows a brief list of interfaces and their IP addresses.

| VM | NAT interface / IP | Host-only interface / IP observed |
|---|---|---|
| Ubuntu | `enp0s3` / `10.0.2.15` | `enp0s8` / `192.168.56.3/24` |
| Rocky | `enp0s3` / `10.0.2.15` | `enp0s8` / `192.168.56.2/24` |

Both VMs can have the same NAT IP because their default NAT connections are separate. For communication between them, I used their different **host-only IP addresses**. These addresses were assigned automatically and may change; check `ip -br address` after a reboot or restore. Do not include `/24` when typing an address in ping or SSH commands.

### Check Rocky’s host-only adapter — inside Rocky

```bash
ip -br address
```

**Observed result:** `enp0s8` was **UP** and had automatically received `192.168.56.2/24`. No manual address assignment or Netplan YAML file was needed on Rocky. Ubuntu needed a separate Netplan configuration because its new interface initially showed **DOWN**; that fix is recorded in the Ubuntu runbook.

### Verify ping between the VMs and SSH from my Mac

With both VMs running, I completed these tests:

| Where I ran it | Command | Result |
|---|---|---|
| Inside Ubuntu | `ping -c 4 192.168.56.2` | 4 packets sent, 4 received; 0% packet loss |
| Inside Rocky | `ping -c 6 192.168.56.3` | 6 packets sent, 6 received; 0% packet loss |
| Mac Terminal | `ssh amarachi@192.168.56.3` | Successfully logged into Ubuntu |
| Mac Terminal, after typing `exit` | `ssh amarachi@192.168.56.2` | Successfully logged into Rocky |

`-c` sets how many ping requests to send before stopping. Ping uses **ICMP** to test reachability. SSH uses **TCP port 22** by default, so these direct connections do not need `-p 2222` or `-p 2223`. The original NAT forwarding rules remain an alternative way to connect. I used each VM’s password for SSH and kept screenshots as evidence.

### Save snapshots named clean — in VirtualBox

I shut down each VM with `sudo poweroff`. In VirtualBox, I selected each powered-off VM → **Snapshots → Take**, named the new snapshot exactly **clean**, and saved it. I retained the earlier installation snapshots.

**Observed result:** Both snapshot lists showed **clean**, capturing the working host-only network setup after ping and direct SSH verification. This is the checkpoint used for the recovery exercise below.

### Break and fix — Rocky SSH failure and snapshot recovery

1. Confirm working SSH from **Mac Terminal** using `ssh amarachi@192.168.56.2`, then type `exit`.
2. Inside **Rocky**, deliberately stop SSH and disable automatic startup:

```bash
sudo systemctl disable --now sshd
```

Rocky’s service is called `sshd`, rather than Ubuntu’s `ssh`.

3. Keep the VM running. From **Mac Terminal**, test a new connection:

```bash
ssh -o ConnectTimeout=5 amarachi@192.168.56.2
```

`-o` sets an SSH option; `ConnectTimeout=5` limits connection setup to five seconds. It can fail sooner if refused, and it does not disconnect a successful session after five seconds.

**Failure result:** SSH was unavailable after disabling it.  Stopping SSH does not itself disable the network; ping is a separate reachability test.

4. In **Rocky’s VirtualBox window**, run `sudo poweroff`.
5. Select **rocky_lab → Snapshots → clean → Restore**. Untick **Create a snapshot of the current machine state** if offered for this deliberately broken state, confirm, then start the VM again. Restoring discards changes made after **clean**.
6. Retry the SSH command from **Mac Terminal**, checking the VM’s current host-only IP if necessary.

**Recovery result:** I successfully logged in again after restoring **clean**. The snapshot restored the working SSH configuration without reinstalling the operating system or manually enabling SSH again.

**Evidence:** Keep the successful login before the break, failed connection, clean snapshot and successful login after recovery.

**Update completion:** shared host-only networking, bidirectional ping, direct SSH, clean snapshot and snapshot-based SSH recovery completed.
