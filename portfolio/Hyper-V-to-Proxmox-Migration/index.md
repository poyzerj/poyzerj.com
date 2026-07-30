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
  <li>Destination: Proxmox VE</li>
  <li>Migration tooling: Veeam</li>
</ul>

<h2>🤔 Why Migrate</h2>
<p>The original lab was built on a physical Hyper-V host. Moving to Proxmox VE was driven by wanting more flexibility for future lab builds — nested virtualization, easier snapshotting, and a platform better suited to running mixed Windows/Linux lab environments (like the Linux-based iSCSI targets used elsewhere in the lab) side by side.</p>

<h2>🔧 Migration Approach</h2>
<p>This was effectively a <strong>P2V migration</strong> — moving a workload running on physical hardware into a virtualized environment on the new host. <strong>Veeam</strong> was used as the migration tool, chosen for its ability to handle the transfer cleanly without requiring a full manual rebuild of every VM from scratch.</p>

<p>The migration process followed a few key stages:</p>
<ul>
  <li><strong>Pre-migration inventory</strong> — documenting exactly what was running on the source Hyper-V host, including the failover cluster nodes, their storage dependencies (the iSCSI shared disk), and any networking configuration that would need to carry over</li>
  <li><strong>Migration execution</strong> — using Veeam to move the VMs from the Hyper-V source to the Proxmox destination</li>
  <li><strong>Post-migration validation</strong> — the critical step, and where most of the real work happened</li>
</ul>

<h2>✅ Post-Migration Validation</h2>
<p>Moving VMs between hypervisors isn't the hard part — confirming everything still actually works afterward is. A few things specifically needed re-validation:</p>
<ul>
  <li><strong>Cluster health</strong> — since the migrated environment included a Windows Server failover cluster (see the <a href="/portfolio/Failover-Cluster-Lab/">Failover Cluster Lab</a> project), cluster validation had to be re-run post-migration to confirm nothing about the underlying virtual hardware change broke cluster communication or storage access</li>
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
  <li>Choosing the right migration tool saves significant time — using Veeam instead of manually rebuilding every VM meant the migration itself was fast, letting the real time investment go into validation, which is exactly where it should go.</li>
</ul>

<p>This project builds directly on the <a href="/portfolio/Failover-Cluster-Lab/">Failover Cluster Lab</a> — the cluster covered there is the same one migrated in this project.</p>

</div>
