---
permalink: /portfolio/Cisco-UCS-Setup/
title: "Cisco UCS Lab Server Deployment"
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
<p>This project covers deploying a <strong>Cisco UCS C220 M3</strong> server as the core host for an ongoing home lab environment — from bare-metal remote management setup through storage layout, hypervisor installation, and network integration with an existing FortiGate/FortiSwitch environment.</p>

<p>Environment:</p>
<ul>
  <li>Cisco UCS C220 M3 rack server</li>
  <li>CIMC (Cisco Integrated Management Controller) for remote/out-of-band management</li>
  <li>Proxmox VE as the hypervisor</li>
  <li>FortiGate + FortiSwitch (FortiLink) for network integration</li>
</ul>

<h2>🔧 Objectives</h2>
<ul>
  <li>Configure CIMC for remote management of the physical host</li>
  <li>Design and build a tiered storage layout across available drive bays</li>
  <li>Install and configure Proxmox VE as the hypervisor</li>
  <li>Integrate the host into the existing VLAN trunk architecture via FortiLink</li>
</ul>

<h2>🖥️ CIMC Configuration</h2>
<p>The first step was configuring the Cisco Integrated Management Controller (CIMC), which provides out-of-band remote access to the server — power control, virtual KVM, and hardware health monitoring — independent of the host OS. This is what makes it possible to manage the server (including reinstalling the OS or troubleshooting boot issues) without needing physical access.</p>

<h2>🗄️ Storage Layout</h2>
<p>With 8 available drive bays, storage was laid out in three tiers based on workload:</p>
<ul>
  <li><strong>RAID 1</strong> (2x smaller HDD) — OS/boot volume</li>
  <li><strong>RAID 10</strong> (4x SSD) — active VM storage, prioritizing IOPS and rebuild resilience for latency-sensitive workloads</li>
  <li><strong>RAID 1</strong> (2x HDD) — cold storage and backups</li>
</ul>

<h2>🌐 Network Integration</h2>
<p>The UCS host was integrated into the existing lab network using a <strong>VLAN trunk architecture</strong> through FortiSwitch (FortiLink), keeping the host's network configuration consistent with the rest of the lab environment rather than treating it as an isolated island. This allowed VMs running on the host to participate in the same VLAN segmentation as the rest of the lab.</p>

<h2>💽 Hypervisor Installation</h2>
<p>Proxmox VE was installed on top of the configured storage layout, chosen for its flexibility with nested virtualization and its ability to run mixed Windows/Linux workloads side by side — a requirement for later projects like the <a href="/portfolio/Failover-Cluster-Lab/">Failover Cluster Lab</a>, which needed both Windows Server VMs and a Linux-based iSCSI target running on the same host.</p>

<h2>📈 Results</h2>
<ul>
  <li>CIMC configured and confirmed for reliable remote management</li>
  <li>Three-tier storage layout deployed across all 8 drive bays</li>
  <li>Proxmox VE installed and operational as the lab's primary hypervisor</li>
  <li>Host fully integrated into the existing VLAN trunk architecture with no network isolation issues</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Configuring out-of-band management (CIMC) first, before anything else, made every later step easier to troubleshoot — never had to worry about losing access to the host</li>
  <li>Planning storage tiers around actual workload needs (boot vs. active VM storage vs. cold storage) up front avoided having to redesign the layout later</li>
  <li>Integrating a new host into existing network architecture (rather than isolating it) meant later lab projects didn't require any additional network rework to get started</li>
</ul>

</div>
