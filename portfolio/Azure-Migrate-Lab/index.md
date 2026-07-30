---
permalink: /portfolio/Azure-Migrate-Lab/
title: "Azure Migrate Lab (Planned)"
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

.status-badge {
  display: inline-block;
  background-color: #f0ad4e;
  color: #ffffff;
  font-size: 14px;
  font-weight: bold;
  padding: 4px 10px;
  border-radius: 4px;
  margin-bottom: 15px;
}
</style>

<div class="lab-content">

<span class="status-badge">📋 PLANNED — In Design/Planning Stage</span>

<h2>📌 Overview</h2>
<p>This project is planned to build hands-on depth with <strong>Azure Migrate</strong>, Microsoft's native tool for discovering, assessing, and migrating on-prem workloads into Azure. The goal is to move beyond conceptual knowledge (covered during AZ-305 study) into an actual, executed migration lab.</p>

<h2>🔧 Planned Objectives</h2>
<ul>
  <li>Stand up an Azure Migrate project and connect it to an on-prem lab environment for discovery</li>
  <li>Run a discovery and assessment against existing lab VMs to evaluate sizing, cost, and migration readiness</li>
  <li>Execute an actual VM migration into Azure using Azure Migrate's replication and cutover workflow</li>
  <li>Validate the migrated VM's functionality and networking post-migration</li>
</ul>

<h2>🗺️ Planned Approach</h2>
<p>The planned workflow follows Azure Migrate's standard process: deploy the discovery appliance against the source host (Hyper-V or Proxmox), run an assessment to determine sizing and readiness, then use the built-in replication tooling (backed by Azure Site Recovery) to migrate a target VM into Azure. A test migration would be performed before a full cutover, consistent with the migration validation approach used across other lab projects on this site.</p>

<h2>🤔 Why This Project</h2>
<p>Azure Migrate is directly relevant to real project work. Building hands-on familiarity with it, rather than relying solely on conceptual knowledge from certification study, is intended to translate directly into stronger contribution on similar real-world migration projects.</p>

<h2>🔗 Related Projects</h2>
<ul>
  <li><a href="/portfolio/RDS-Lab">RDS Lab (Planned)</a></li>
  <li><a href="/portfolio/ESXi-to-Hyper-V-Migration">ESXi to Hyper-V Migration</a></li>
  <li><a href="/portfolio/Hyper-V-to-Proxmox-Migration">Hyper-V to Proxmox Migration</a></li>
</ul>

</div>
