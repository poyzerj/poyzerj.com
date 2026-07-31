---
permalink: /portfolio/RDS-Lab/
title: "RDS Lab (Planned)"
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

<span class="status-badge">📋 PLANNED</span>

<h2>📌 Overview</h2>
<p>This project is planned as a full <strong>Remote Desktop Services (RDS)</strong> deployment lab, covering a complete five-server topology with modern profile management. The design phase is complete; build-out is next on the lab roadmap, following the <a href="/portfolio/AVD-Lab">AVD Lab</a>, which was prioritized first.</p>

<h2>🗺️ Planned Topology</h2>
<ul>
  <li>Domain Controller (DC)</li>
  <li>RD Gateway (RDGW)</li>
  <li>RD Connection Broker (RDCB)</li>
  <li>2x RD Session Hosts (RDSH)</li>
</ul>

<h2>🔧 Planned Objectives</h2>
<ul>
  <li>Deploy a full five-role RDS environment: DC, Gateway, Broker, and two Session Hosts</li>
  <li>Configure FSLogix for user profile management across session hosts</li>
  <li>Integrate OneDrive Known Folder Move (KFM) for folder redirection, so user documents/desktop sync through OneDrive rather than a traditional file-share redirect</li>
  <li>Validate RD Gateway external access and session load balancing across both session hosts</li>
</ul>

<h2>🤔 Why This Design</h2>
<p>FSLogix plus OneDrive-based folder redirection reflects how RDS environments are increasingly deployed in practice — combining traditional on-prem RDS infrastructure with cloud-connected profile and file storage, rather than a fully legacy on-prem design. This also mirrors patterns used in Azure Virtual Desktop deployments, making the two labs complementary rather than redundant.</p>

<h2>📝 Why It's on the Roadmap</h2>
<p>AVD was prioritized first, ahead of this RDS build, due to a limited-time Azure trial.</p>


</div>
