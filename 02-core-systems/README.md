# Chapter 2 – Core Systems

> Building the first two machines of the check-in zone in Hyper-V and connecting them on an isolated network.

**Status:** Completed – lab version 0.1

---

## Host and hypervisor

| Item | Choice | Why |
|---|---|---|
| Hypervisor | **Microsoft Hyper-V** (Windows 11 Pro) | Type-1 hypervisor, the same technology used on Windows Server in enterprise environments |
| Storage | Dedicated NVMe drive (`E:\Lab`) | Keeps the lab separate from the host operating system |
| Virtual switch | `Checkin-Zone` – **Private** | VMs can talk only to each other: no internet, no access to my home network |

> During installation, each VM was temporarily connected to Hyper-V's *Default Switch* to download updates, then moved to the private switch.

## Virtual machines

| Setting | `checkin-srv` | `checkin-pc01` |
|---|---|---|
| Role | Check-in system (supplier-provided) | Check-in desk operator's workstation |
| OS | Ubuntu Server LTS | Windows 11 Enterprise (evaluation) |
| Generation | 2 (UEFI) | 2 (UEFI) |
| vCPU / RAM | 2 / 4 GB (static) | 2 / 4 GB (static) |
| Disk | 40 GB (dynamic VHDX) | 64 GB (dynamic VHDX) |
| Secure Boot template | Microsoft UEFI Certificate Authority | Microsoft Windows |
| Virtual TPM | – | Enabled (required by Windows 11) |
| IP address | 10.10.2.66/27 (static) | 10.10.2.80/27 (static) |
| Gateway | 10.10.2.65 (future firewall) | 10.10.2.65 (future firewall) |

## Security choices during setup

- **No obvious usernames** such as `admin` or `root`, which are the first ones attackers try
- **Strong, unique passwords** stored in a password manager
- **Minimal installation:** no optional packages on the server – every extra service is extra attack surface
- **OpenSSH server installed** for administration, as defined in the network design (password authentication will be replaced by key-based authentication during hardening)
- **Local account** on the workstation instead of a personal Microsoft account; all optional telemetry and privacy settings disabled
- **Fully updated** before being connected to the private network
- **Checkpoints** (`base-install`, `v0.1-network`) to roll back safely after experiments

## Static IP configuration

There is no DHCP server on the private network yet, so addresses are assigned manually according to the [addressing plan](../01-network-design/README.md).

**Server – netplan** (`/etc/netplan/50-cloud-init.yaml`, permissions `600`):

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 10.10.2.66/27
      routes:
        - to: default
          via: 10.10.2.65
```

cloud-init network management was disabled so it cannot overwrite this file at boot:

```bash
echo 'network: {config: disabled}' | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

The configuration was applied with `sudo netplan try`, which rolls back automatically if the change is not confirmed – a safe habit on remote servers.

**Workstation:** IPv4 set manually via `ncpa.cpl` (10.10.2.80, mask 255.255.255.224, gateway 10.10.2.65).

## Connectivity test

From `checkin-pc01`:

```
ping 10.10.2.66
```

The server replies: the two machines communicate on the isolated check-in network.

> **Note:** Windows Defender Firewall blocks inbound ICMP echo requests by default, so a ping from the server to the workstation is not answered. This is expected behavior, not a fault – and the first defensive control visible in the lab.

## Lessons learned

- Hyper-V Generation 2 VMs only trust Microsoft-signed bootloaders by default: Linux needs the **Microsoft UEFI Certificate Authority** template, otherwise it will not boot
- Windows 11 VMs need a **virtual TPM** and at least **2 vCPUs**
- The keyboard layout selected during installation must match the physical keyboard, or passwords containing symbols may not work
- On an isolated network without DHCP, every host needs a **manually planned static address**
