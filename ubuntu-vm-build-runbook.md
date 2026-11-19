# Ubuntu Server VM — Build Runbook

  
**Purpose:** Build a Linux lab VM, set a hostname, create a normal user, verify SSH from the host and take a snapshot.

## 1. Build specification

| Component | Configuration |
|---|---|
| Host | 2015 MacBook Pro; macOS Monterey 12.7.6; Intel Core i7; 16 GB RAM |
| Hypervisor | VirtualBox 7.1.18 for Intel macOS |
| Installation ISO | `ubuntu-24.04.4-live-server-amd64.iso` |
| VM / hostname / user | `ubuntu_lab` / `ubuntulab` / `amarachi` |
| Resources | 2 virtual CPUs; 2048 MB RAM; 25 GB dynamically allocated VDI |
| Network | Adapter 1: NAT; guest address observed: `10.0.2.15` |

## 2. Create the virtual computer — on my Mac
1. Install **VirtualBox 7.1.18 for Intel Macs**. Download `ubuntu-24.04.4-live-server-amd64.iso` from Ubuntu’s official website. My Mac runs Monterey 12.7.6 and has 16 GB RAM.
2. Open VirtualBox → **New**. Name it `ubuntu_lab`. Keep the default folder and choose the downloaded Ubuntu file under **ISO Image**.
3. Tick **Skip Unattended Installation** to install Ubuntu myself. Set **Memory: 2048 MB**, **Processors: 2**; leave **EFI** unticked.
4. Create a **25 GB** virtual hard disk. Keep **VDI** selected and **Pre-allocate Full Size** unticked: the disk uses space gradually. Click **Finish → Start**.

## 3. Install Ubuntu — inside the VM window
Use **Tab/arrow keys** to move, **Space** to tick options and **Enter** to confirm.
1. Choose the language and keyboard layout. If offered an installer update, choose **Continue without updating**. Select **Ubuntu Server**.
2. Keep the automatic network settings. Leave **Proxy** blank. On the download mirror screen, wait for a successful test, then choose **Done**.
3. Choose **Use an entire disk**, checking it is the **25 GB VBOX disk**. Keep **LVM** ticked (Ubuntu manages the disk space); leave encryption unticked. Select **Done**, review the summary, then **Done → Continue**. This formats the virtual disk.
4. Enter **Your name: Amarachi**, **Server’s name: ubuntulab** (the hostname), **Username: amarachi** and a private password. This creates my normal account.
5. Tick **Install OpenSSH server** (allows terminal connections) and keep password login enabled. Leave **Import SSH key** unchanged. Leave **Featured server snaps** unticked; they are optional applications.
6. When installation finishes, select **Reboot Now**. If asked to remove installation media, use the VM’s **Devices → Optical Drives → Remove disk from virtual drive**, then press Enter. Log in as `amarachi`. This is the installed system’s **first boot**. Password typing is invisible.

## 3. Install and enable SSH — inside Ubuntu

I tried to enable ssh but the initial enable command reported `ssh.service does not exist`; resolve by installing the server package:

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable --now ssh
systemctl is-active ssh       # Expected: active
sudo poweroff
```

## 4. Configure and test access — on the Mac

With the VM powered off, open the Ubuntu VM **Settings → Network → Adapter 1 → Advanced → Port Forwarding** and add:

| Name | Protocol | Host IP | Host port | Guest IP | Guest port |
|---|---|---|---|---|---|
| SSH | TCP | 127.0.0.1 | 2222 | Empty | 22 |

Save settings, start Ubuntu, then run in **macOS Terminal**:

```bash
ssh -p 2222 amarachi@127.0.0.1
```

This sends the Mac’s connection on port **2222** to Ubuntu’s SSH port **22**. On my first lab connection, accept the expected new-server prompt with `yes`, then enter my Ubuntu password. **Success:** my Mac’s Terminal shows `amarachi@ubuntulab:~$`; commands now run inside Ubuntu. If the address cannot be resolved, check it is exactly `127.0.0.1`.
For repeat checks, `whoami` should return `amarachi` and `hostname` should return `ubuntulab`. 

## 5. Save the baseline

Run `sudo poweroff` from Ubuntu (the SSH session closes). In VirtualBox, select the powered-off VM → **Snapshots → Take**. Save as **Ubuntu - Initial setup and SSH working**, describing the hostname, user and successful SSH test. **Observed result:** the named snapshot appeared in the snapshot list. To roll back later, power off the VM, select this snapshot and choose **Restore**; later changes can be lost. A snapshot is a checkpoint, not an independent backup.

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

### Activate Ubuntu’s host-only adapter — inside Ubuntu

Rocky’s new adapter received an address automatically, but Ubuntu’s `enp0s8` showed **DOWN** with no IP. I created a Netplan configuration file:

```bash
sudo nano /etc/netplan/60-host-only.yaml 
```

I entered only these lines, using spaces rather than tabs:

```yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: true
      optional: true
```

`dhcp4: true` asks for an IPv4 address automatically. `optional: true` prevents this adapter from holding up boot. The `.yaml` file stores Ubuntu’s network settings; the name `60-host-only` was our choice.

Save with **Control + O → Enter**, then exit with **Control + X**. Run each command separately:

```bash
sudo chmod 600 /etc/netplan/60-host-only.yaml
sudo netplan generate
sudo netplan apply
ip -br address
```

`chmod 600` restricts access to the file’s owner. `netplan generate` checks and generates the configuration; if it reports an error, fix the file before running `netplan apply`.

**Troubleshooting:** My first attempt returned “Invalid YAML: mapping values are not allowed in this context”. I inspected it with:

```bash
sudo cat -n /etc/netplan/60-host-only.yaml
```

I removed an extra `YAML` line, corrected `ethernet` to `ethernets` and `dhcpq4` to `dhcp4`, and checked the indentation. After applying the corrected file, `enp0s8` showed **UP** with `192.168.56.3/24`. Saving the configuration makes it available after reboot.

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

### Break and fix — Ubuntu SSH failure and snapshot recovery

1. Confirm working SSH from **Mac Terminal** using `ssh amarachi@192.168.56.3`, then type `exit`.
2. Inside **Ubuntu**, deliberately stop SSH and disable automatic startup:

```bash
sudo systemctl disable --now ssh.service ssh.socket
```

In this exercise I entered the command through an existing SSH connection, then typed `exit`. The new SSH attempt below was refused. For a repeat exercise, using the VirtualBox console keeps local access available.

3. Keep the VM running. From **Mac Terminal**, test a new connection:

```bash
ssh -o ConnectTimeout=5 amarachi@192.168.56.3
```

`-o` sets an SSH option; `ConnectTimeout=5` limits connection setup to five seconds. It can fail sooner if refused, and it does not disconnect a successful session after five seconds.

**Failure result:** SSH was unavailable after disabling it. The captured error was “Connection refused”. Stopping SSH does not itself disable the network; ping is a separate reachability test.

4. In **Ubuntu’s VirtualBox window**, run `sudo poweroff`.
5. Select **ubuntu_lab → Snapshots → clean → Restore**. Untick **Create a snapshot of the current machine state** if offered for this deliberately broken state, confirm, then start the VM again. Restoring discards changes made after **clean**.
6. Retry the SSH command from **Mac Terminal**, checking the VM’s current host-only IP if necessary.

**Recovery result:** I successfully logged in again after restoring **clean**. The snapshot restored the working SSH configuration without reinstalling the operating system or manually enabling SSH again.

**Evidence:** Keep the successful login before the break, failed connection, clean snapshot and successful login after recovery.

### Additional practice — SSH from Ubuntu into my Mac

1. In **Mac Terminal**, run `ifconfig` and check the `inet` addresses. My Mac’s host-only address was `192.168.56.4`.
2. On macOS Monterey, open **System Preferences → Sharing → Remote Login**. Enable it and allow **Only these users**, including my Mac account `admin`.
3. Inside **Ubuntu**, run:

```bash
ssh admin@192.168.56.4
```

Use the **Mac login password**, not the Ubuntu password. **Result:** I connected successfully from Ubuntu to my Mac. Commands in that session run on the Mac; `exit` returns to Ubuntu. Remote Login is a Mac setting, so restoring an Ubuntu snapshot does not change it.

**Update completion:** shared host-only networking, bidirectional ping, direct SSH, clean snapshot and snapshot-based SSH recovery completed.
