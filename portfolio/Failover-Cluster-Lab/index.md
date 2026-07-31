---
permalink: /portfolio/Failover-Cluster-Lab/
title: "Failover Cluster Lab"
layout: single
author_profile: true
---

<style>
.lab-content h2 {
  font-size: 21px;
  font-weight: bold;
  margin: 20px 0 10px 0;
}

.lab-content p {
  font-size: 17px;
  line-height: 1.6;
  margin-bottom: 15px;
}

.lab-content ul {
  font-size: 17px;
  line-height: 1.6;
  margin-bottom: 15px;
  padding-left: 20px;
}

.lab-content li {
  margin-bottom: 8px;
}

.lab-content pre {
  font-size: 15px;
  background-color: #f6f8fa;
  padding: 16px;
  border-radius: 6px;
  overflow-x: auto;
  margin-bottom: 15px;
}

.lab-content code {
  font-size: 15px;
}

.lab-content img {
  max-width: 100%;
  height: auto;
  margin: 20px 0;
}

.lab-content a {
  color: #0066cc;
  text-decoration: none;
  cursor: pointer;
}

.lab-content a:hover {
  text-decoration: underline;
}

.lab-content strong {
  font-weight: bold;
}
</style>

<div class="lab-content">

<h2>📌 Overview</h2>
<p>This lab covers building a <strong>two-node Windows Server 2025 failover cluster</strong> in a workgroup (non-domain) configuration, backed by <strong>iSCSI shared storage</strong> hosted on a Linux LIO target. The goal was to understand what actually has to be true at the storage and networking layer for Windows clustering to work — not just follow a guide end to end, but hit real failures and fix them.</p>

<p>Environment:</p>
<ul>
  <li>2x Windows Server 2025 VMs (workgroup, not domain-joined)</li>
  <li>Shared storage: iSCSI target on Linux, using LIO (Linux-IO Target)</li>
  <li>Quorum: Disk witness, backed by the same iSCSI storage</li>
  <li>Host: Cisco UCS C220 M3, running Proxmox VE</li>
</ul>

<h2>🗄️ Storage Design</h2>
<p>Before any cluster configuration, the underlying physical storage on the UCS host needed to be laid out correctly across 8 available drive bays:</p>
<ul>
  <li><strong>RAID 1</strong> (2x HDD) — OS/boot volume</li>
  <li><strong>RAID 10</strong> (4x SSD) — active VM storage, chosen for rebuild characteristics (mirrors, not parity recalculation) and better random I/O under cluster workloads</li>
  <li><strong>RAID 1</strong> (2x HDD) — cold storage and backups</li>
</ul>

<h2>🔧 Objectives</h2>
<ul>
  <li>Deploy a two-node workgroup failover cluster on Windows Server 2025</li>
  <li>Configure iSCSI shared storage using a Linux-based target</li>
  <li>Select and configure appropriate cluster quorum</li>
  <li>Pass cluster validation, including SCSI-3 Persistent Reservation checks</li>
  <li>Bring the cluster online with functional shared storage</li>
</ul>

<h2>⚖️ Quorum Design</h2>
<p>Since this was a workgroup cluster (no Active Directory), quorum configuration required extra attention. A <strong>disk witness</strong> was used, backed by a small LUN on the iSCSI target — the right call for a single-site cluster with shared storage already available. A cloud witness would be the better fit for a multi-site cluster without shared storage, which wasn't the case here.</p>

<h2>🚧 The SCSI-3 Persistent Reservation Problem</h2>
<p>Windows failover clustering requires the shared disk to pass <strong>SCSI-3 Persistent Reservation (PR) validation</strong> — a mechanism that lets both cluster nodes coordinate access to the shared disk and prevents split-brain scenarios where both nodes think they own the disk simultaneously.</p>

<p>Cluster validation kept failing at this exact check. The shared iSCSI LUN was visible to both nodes, but the PR handshake wasn't completing.</p>

<p><strong>Root cause:</strong> the issue was on the Linux iSCSI target side, in the Target Portal Group (TPG) attributes:</p>
<ul>
  <li><code>generate_node_acls</code> needed to be <strong>enabled</strong>, so the target would properly authenticate the initiators (the Windows nodes) rather than rejecting the reservation request outright</li>
  <li><code>auth_attr_enforcing</code> needed to be <strong>disabled</strong>, since strict authentication enforcement was interfering with how the cluster nodes negotiated the SCSI-3 reservation</li>
</ul>

<p>Once both settings were corrected on the LIO target, cluster validation passed cleanly and the cluster came online with fully functional shared storage.</p>

<h2>📈 Results</h2>
<ul>
  <li>Cluster validation passed cleanly after correcting the iSCSI target's TPG attributes</li>
  <li>Two-node failover cluster came online with fully functional shared storage</li>
  <li>Disk witness quorum confirmed healthy</li>
  <li>Storage I/O stabilized after replacing the SMR drive and failing SSD</li>
</ul>


<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>SCSI-3 PR validation issues are almost always a <strong>storage-side configuration problem</strong>, not a Windows problem — when clustering fails at this step, the fix usually lives in the iSCSI target's authentication and ACL settings</li>
  <li>RAID level choice should be driven by <strong>workload</strong>, not just redundancy needs — RAID 10 for active cluster/VM storage was the right call specifically because of its rebuild behavior and random I/O performance</li>
  <li>Drive <strong>technology</strong> matters as much as capacity — an SMR drive that looks fine on paper can silently degrade performance under a clustering/VM workload</li>
  <li>Quorum design should match cluster topology — a disk witness made sense here because this was a single-site cluster with shared storage already in place</li>
</ul>

</div>
