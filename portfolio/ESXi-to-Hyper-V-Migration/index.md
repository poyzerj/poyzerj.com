---
permalink: /portfolio/ESXi-to-Hyper-V-Migration/
title: "ESXi to Hyper-V Migration"
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
<p>This project covers a <strong>V2V (Virtual-to-Virtual) migration</strong>, moving VM workloads from a VMware ESXi host to a Hyper-V environment using <strong>Veeam Backup &amp; Replication</strong>. The goal was to build hands-on comfort with cross-hypervisor migration tooling and validate that a clean migration workflow could move workloads without manual rebuilds.</p>
<p>Environment:</p>
<ul>
  <li>Source: VMware ESXi host</li>
  <li>Destination: Hyper-V host</li>
  <li>Migration tooling: Veeam Backup &amp; Replication, running on a dedicated VM (VeeamServer)</li>
  <li>Shared backup storage: <code>\\\\192.168.20.100\VeeamBackups</code></li>
</ul>

<h2>🖥️ Why a Dedicated VeeamServer VM</h2>
<p>Veeam Backup &amp; Replication requires a proper Windows Server OS to install on — it wasn't an option to install it directly on FUSION-DT, since that machine runs Windows 11 Pro rather than Windows Server. The workaround was standing up a dedicated VM, <strong>VeeamServer</strong>, running Windows Server specifically to host the Veeam Backup &amp; Replication installation, rather than trying to force an install onto an unsupported OS.</p>

<h2>🔧 Objectives</h2>
<ul>
  <li>Migrate VM workloads from ESXi to Hyper-V without a manual rebuild</li>
  <li>Validate VM functionality post-migration</li>
  <li>Build repeatable familiarity with Veeam Backup &amp; Replication as a cross-hypervisor migration tool</li>
</ul>

<h2>🚀 Migration Approach</h2>
<p>With Veeam Backup &amp; Replication installed on VeeamServer, the migration followed a few key steps:</p>
<ul>
  <li><strong>Connect shared backup storage</strong> — pointed Veeam at <code>\\\\192.168.20.100\VeeamBackups</code> as the backup repository, authenticating with the <code>jpadmin</code> local administrator account</li>
  <li><strong>Add the ESXi host</strong> — connected Veeam to the source VMware ESXi host so it could see and back up the target VMs</li>
  <li><strong>Add the Hyper-V host</strong> — connected Veeam to the destination Hyper-V host as the migration target</li>
  <li><strong>Full backup</strong> — ran a full backup of the source VMs from ESXi to the shared backup repository</li>
  <li><strong>Instant VM Recovery</strong> — used Veeam's Instant VM Recovery to bring the backed-up VMs online directly from the backup repository</li>
  <li><strong>Migrate to production</strong> — migrated the recovered VMs into permanent production storage on the Hyper-V host, completing the move</li>
</ul>

<p>Since this was a V2V migration (VM to VM, rather than physical hardware to VM), the process was more straightforward than a P2V migration, without needing to account for physical hardware drivers or boot configuration differences.</p>

<h2>📈 Results</h2>
<ul>
  <li>VMs migrated successfully from ESXi to Hyper-V with no data loss</li>
  <li>Migration completed smoothly using Veeam Backup &amp; Replication, with no significant issues encountered</li>
  <li>Post-migration validation confirmed VMs booted correctly and functioned as expected on the new hypervisor</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Veeam Backup &amp; Replication's OS requirements matter — a proper Windows Server install is required, which meant standing up a dedicated VeeamServer VM rather than trying to run it on a Windows 11 Pro machine</li>
  <li>V2V migrations are generally more predictable than P2V migrations, since the source is already virtualized — fewer hardware-specific variables to account for</li>
  <li>Instant VM Recovery followed by a migration to production storage is a reliable pattern for minimizing downtime — the VM is usable almost immediately from the backup repository, with the final storage move happening in the background</li>
  <li>Even when a migration goes smoothly, post-migration validation is still worth doing as a matter of process — confirming VM functionality rather than just confirming the migration job completed</li>
</ul>


</div>
