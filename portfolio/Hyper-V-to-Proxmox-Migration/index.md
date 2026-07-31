---
permalink: /portfolio/Hyper-V-to-Proxmox-Migration/
title: "Hyper-V to Proxmox Migration"
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
<p>This project covers a <strong>physical-to-virtual (P2V) migration</strong> moving an existing lab environment — including a running Windows Server failover cluster — off a physical Hyper-V host and onto <strong>Proxmox VE</strong>. The goal was to change hypervisors without losing any of the work already built, and to understand what actually needs to be verified after a migration like this before trusting the environment again.</p>

<p>Environment:</p>
<ul>
  <li>Source: Physical Hyper-V host (FUSION-DT)</li>
  <li>Destination: Cisco UCS host running Proxmox VE</li>
  <li>Migration tooling: Veeam Agent for Windows</li>
</ul>

<h2>🤔 Why Migrate</h2>
<p>The original lab ran on FUSION-DT's D drive — a spinning HDD used for VM storage. Under the I/O load of installing and running the cluster VMs, that drive was regularly getting pegged at 100% utilization, making the environment unreliable to work in. Rather than continue fighting disk performance on aging spinning storage, the cluster VMs were migrated to the Cisco UCS host, which had a proper RAID 10 SSD tier already built for exactly this kind of active VM workload (see the <a href="/portfolio/Cisco-UCS-Setup/">Cisco UCS Lab Server Deployment</a> project).</p>

<h2>🔧 Migration Approach: Two Veeam Tools, Two Reasons</h2>
<p>This was a <strong>P2V (Physical-to-Virtual) migration</strong> — moving workloads off physical hardware (FUSION-DT) into a virtualized environment on the new host. It required two different Veeam tools, each solving a different part of the problem.</p>

<p><strong>Veeam Agent for Windows — why it was necessary:</strong> FUSION-DT was running <strong>Windows 11 Pro</strong>, not Windows Server. Veeam Backup &amp; Replication's Hyper-V integration is built around proper Hyper-V hosts (Windows Server with the Hyper-V role, or a dedicated Hyper-V Server) — it wasn't a supported target for hypervisor-level, host-integrated backup against a Windows 11 Pro machine, even with the client Hyper-V role enabled. Veeam Agent for Windows sidesteps that entirely by performing an image-level backup at the OS level, from inside the guest, regardless of what the underlying host is — which made it the right tool to actually get the VMs backed up off FUSION-DT in the first place.</p>

<p><strong>Veeam Backup &amp; Replication — finishing the migration:</strong> once the Veeam Agent backups existed, <strong>Veeam Backup &amp; Replication's Instant VM Recovery</strong> feature was used to bring those backups up as running VMs directly from the backup repository — and then migrate them into permanent production storage on the Proxmox host, completing the move rather than leaving them running from a temporary recovery state.</p>

<p>The migration process followed a few key stages:</p>
<ul>
  <li><strong>Pre-migration inventory</strong> — documenting exactly what was running on the source Hyper-V host, including the failover cluster nodes, their storage dependencies (the iSCSI shared disk), and any networking configuration that would need to carry over</li>
  <li><strong>Backup</strong> — using Veeam Agent for Windows to capture each VM at the OS level, since FUSION-DT itself wasn't a supported Veeam B&amp;R Hyper-V target</li>
  <li><strong>Instant Recovery &amp; migration to production</strong> — using Veeam B&amp;R's Instant VM Recovery to bring the backed-up VMs online, then migrating them into permanent storage on the Proxmox host to finish the move</li>
  <li><strong>Post-migration validation</strong> — the critical step, and where most of the real work happened</li>
</ul>

<h2>✅ Post-Migration Validation</h2>
<p>Moving VMs between hypervisors isn't the hard part — confirming everything still actually works afterward is. A few things specifically needed re-validation:</p>
<ul>
  <li><strong>Cluster health</strong> — since the migrated environment included a Windows Server failover cluster, cluster validation had to be re-run post-migration to confirm nothing about the underlying virtual hardware change broke cluster communication or storage access</li>
  <li><strong>SCSI-3 Persistent Reservation behavior</strong> — a particular point of attention, since PR validation had already caused problems once during the original cluster build. Re-confirming this behavior specifically after a hypervisor change, rather than assuming it would "just work," avoided any surprises</li>
  <li><strong>Networking</strong> — confirming virtual NICs, VLANs, and any static IP/network configuration carried over correctly on the new virtual hardware layer under Proxmox</li>
</ul>

<h2>📈 Results</h2>
<ul>
  <li>Full lab environment successfully migrated from Hyper-V to Proxmox VE with no data loss</li>
  <li>Failover cluster re-validated and confirmed healthy post-migration</li>
  <li>SCSI-3 Persistent Reservation behavior confirmed intact after the hardware layer change</li>
  <li>Networking configuration carried over correctly with no reconfiguration needed</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>A migration isn't done when the VM boots — it's done when <strong>everything dependent on that VM is re-validated</strong>. For a standalone VM, that might mean confirming the OS boots and services start. For something like a cluster, it means re-testing the specific mechanisms (like SCSI-3 PR) that could be sensitive to underlying hardware changes.</li>
  <li>Documenting the source environment before migrating matters — knowing exactly what dependencies existed (shared storage, specific network config) made it possible to know what to check afterward, rather than discovering gaps reactively.</li>
  <li>Choosing the right migration tool depends on what the source actually is — a client OS like Windows 11 Pro isn't a supported Veeam B&amp;R Hyper-V target, so Veeam Agent was the right tool for the backup step, with Veeam B&amp;R's Instant VM Recovery picking up from there to finish the move. Understanding the boundary between what each Veeam product actually supports avoided wasted time trying to force B&amp;R to do something it wasn't designed for on this particular host.</li>

</ul>


</div>
