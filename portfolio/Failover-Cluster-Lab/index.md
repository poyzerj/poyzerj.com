---
permalink:	/portfolio/Failover-Cluster-Lab/
title:	"Failover Cluster Lab"
layout:	single
author_profile:	true
---

<style> .lab-content h2 { font-size: 21px; font-weight: bold; margin: 20px 0 10px 0; } .lab-content p { font-size: 17px; line-height: 1.6; margin-bottom: 15px; } .lab-content ul { font-size: 17px; line-height: 1.6; margin-bottom: 15px; padding-left: 20px; } .lab-content li { margin-bottom: 8px; } .lab-content pre { font-size: 15px; background-color: #f6f8fa; padding: 16px; border-radius: 6px; overflow-x: auto; margin-bottom: 15px; } .lab-content code { font-size: 15px; } .lab-content img { max-width: 100%; height: auto; margin: 20px 0; } .lab-content a { color: #0066cc; text-decoration: none; cursor: pointer; } .lab-content a:hover { text-decoration: underline; } .lab-content strong { font-weight: bold; } .modal { display: none; position: fixed; z-index: 1000; left: 0; top: 0; width: 100%; height: 100%; overflow: hidden; background-color: rgba(0,0,0,0); padding: 40px; display: flex; align-items: center; justify-content: center; } .modal-content { background-color: #ffffff; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); padding: 0; border: none; outline: none; width: 90%; max-width: 900px; max-height: 60vh; border-radius: 8px; box-shadow: 0 8px 32px rgba(0,0,0,0.2); display: flex; flex-direction: column; overflow: hidden; } .modal-header { padding: 16px 20px; background-color: #ffffff; border-bottom: none; border-radius: 8px 8px 0 0; display: flex; justify-content: space-between; align-items: center; flex-shrink: 0; } .modal-header h3 { margin: 0; font-size: 19px; font-weight: bold; } .close { color: #aaa; font-size: 28px; font-weight: bold; cursor: pointer; line-height: 20px; } .close:hover, .close:focus { color: #000; } .modal-body { padding: 0; overflow-y: auto; flex: 1; background-color: #ffffff; } .modal-body::-webkit-scrollbar { width: 12px; } .modal-body::-webkit-scrollbar-track { background: #ffffff; } .modal-body::-webkit-scrollbar-thumb { background: #cccccc; border-radius: 6px; } .modal-body::-webkit-scrollbar-thumb:hover { background: #999999; } .modal-body > div { padding: 20px; } .modal-body pre { background-color: #ffffff; padding: 0; border-radius: 0; overflow-x: auto; margin: 0; font-size: 14px; line-height: 1.5; text-align: left; } .modal-body code { font-family: 'Courier New', monospace; font-size: 14px; text-align: left; display: block; } .loading { text-align: center; padding: 40px; color: #666; } </style>
## Overview

This project covers building a two-node Windows Server 2025 failover cluster in a workgroup (non-domain) configuration, backed by iSCSI shared storage, and then migrating the entire environment from a nested Hyper-V lab host to a Proxmox VE host — including the storage layer itself, moving from a physical disk-based setup to a Linux-based iSCSI target (LIO).

The goal wasn't just to get a cluster online — it was to understand what actually has to be true at the storage and networking layer for Windows clustering to work, and to build that understanding by hitting real failures and fixing them, rather than just following a guide end to end.

## Environment

- **Cluster nodes:** 2x Windows Server 2025 VMs (workgroup, not domain-joined)
- **Shared storage:** iSCSI target hosted on Linux, using LIO (Linux-IO Target)
- **Quorum:** Disk witness, backed by the same iSCSI storage
- **Hypervisor (final):** Proxmox VE
- **Storage design:** RAID 1 for OS/boot, RAID 10 for VM storage, RAID 1 for cold storage/backups, on a Cisco UCS C220 M3 host

## Phase 1: Storage Design

Before any cluster configuration, the underlying physical storage on the UCS host needed to be laid out correctly. With 8 drive bays available:

- 2x smaller HDDs in RAID 1 for the OS/boot volume
- 4x SSDs in RAID 10 for active VM storage (chosen over RAID 5/6 specifically for the rebuild characteristics — RAID 10 rebuilds from a mirror rather than recalculating parity, and handles random I/O better under cluster workloads)
- 2x remaining HDDs in RAID 1 for cold storage and backups

## Phase 2: Cluster Build and Quorum Design

With Proxmox and the base VMs in place, the next step was building the actual Windows failover cluster. Since this was a workgroup cluster (no Active Directory), quorum configuration required extra attention — a disk witness was used, backed by a small LUN carved out on the iSCSI target, since this was a single-site cluster with shared storage already available. (A cloud witness would be the better fit for a multi-site cluster without shared storage — not the case here.)

## Phase 3: The SCSI-3 Persistent Reservation Problem

This is where the project got interesting. Windows failover clustering requires the shared disk to pass **SCSI-3 Persistent Reservation (PR) validation** — a mechanism that lets both cluster nodes coordinate access to the shared disk and prevents split-brain scenarios where both nodes think they own the disk simultaneously.

Cluster validation kept failing at this exact check. The shared iSCSI LUN was visible to both nodes, but the PR handshake wasn't completing.

**Root cause:** the issue was on the Linux iSCSI target side, in the Target Portal Group (TPG) attributes:

- `generate_node_acls` needed to be **enabled**, so the target would properly authenticate the initiators (the Windows nodes) rather than rejecting the reservation request outright
- `auth_attr_enforcing` needed to be **disabled**, since strict authentication enforcement was interfering with how the cluster nodes negotiated the SCSI-3 reservation

Once both settings were corrected on the LIO target, cluster validation passed cleanly and the cluster came online with fully functional shared storage.

This was a good reminder that Windows clustering and Linux-based iSCSI targets don't always "just work" together out of the box — the reservation mechanism Windows expects has to be explicitly supported and correctly configured on the storage side, which isn't always obvious from Windows' side of the error message alone.

## Phase 4: Storage Reliability Issues

During the build, the environment also surfaced a second, unrelated problem: one of the physical drives in the storage array was a Seagate ST4000DM004 — an **SMR (Shingled Magnetic Recording)** drive — being used for VM/cluster storage. SMR drives use overlapping tracks for higher density, which makes them poor performers for sustained random write workloads like active VM storage, since writes can trigger costly internal re-shingling operations.

Around the same time, a Samsung 860 EVO SSD in the environment began throwing Event ID 51 errors, indicating early failure.

Rather than working around either issue, both drives were replaced — VM/cluster storage was moved off the SMR drive entirely, and the failing SSD was swapped out before continuing. I/O performance stabilized immediately afterward. This reinforced a lesson that's now baked into how I approach storage design going forward: drive *technology* (SMR vs. CMR) matters as much as raw capacity when planning storage for latency-sensitive workloads like clustering.

*This cluster was later migrated from its original Hyper-V lab host to Proxmox VE — see the [Hyper-V to Proxmox Migration]({{ site.url }}/portfolio/hyper-v-to-proxmox-migration) project for that story.*

## Lessons Learned

- **SCSI-3 PR validation issues are almost always a storage-side configuration problem**, not a Windows problem — when clustering fails at this step, the fix usually lives in the iSCSI target's authentication and ACL settings, not in Windows Server itself.
- **RAID level choice should be driven by workload, not just redundancy needs** — RAID 10 for active cluster/VM storage was the right call specifically because of its rebuild behavior and random I/O performance, not just because it's "more redundant."
- **Drive technology matters as much as capacity** — an SMR drive that looks fine on paper (capacity, price) can silently degrade performance under a clustering/VM workload in a way that's easy to misattribute to something else.
- **Quorum design should match cluster topology** — a disk witness made sense here because this was a single-site cluster with shared storage already in place; a different topology would call for a different witness type.

*Full step-by-step configuration guide available on request.*
