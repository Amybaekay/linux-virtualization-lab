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
