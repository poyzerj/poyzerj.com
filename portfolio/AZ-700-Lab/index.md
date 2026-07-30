---
permalink: /portfolio/AZ-700-Lab/
title: "AZ-700 - Azure Hub-Spoke Network with Hybrid Connectivity"
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
<p>This was the hands-on project completed as part of earning the <strong>Microsoft Certified: Azure Network Engineer Associate (AZ-700)</strong> certification — a full hub-spoke Azure network built from scratch with hybrid connectivity back to a home lab environment, rather than a purely isolated cloud exercise.</p>

<p>Environment:</p>
<ul>
  <li>Azure hub-spoke virtual network topology (Central US region)</li>
  <li>Site-to-Site VPN connecting Azure to a home FortiGate</li>
  <li>Point-to-Site OpenVPN with Entra ID authentication</li>
  <li>Private Endpoint and Private DNS integration</li>
</ul>

<h2>🔧 Objectives</h2>
<ul>
  <li>Design and deploy a hub-spoke VNet architecture with proper peering and gateway transit</li>
  <li>Establish a Site-to-Site VPN tunnel between Azure and an on-prem FortiGate</li>
  <li>Configure Point-to-Site VPN access using Entra ID authentication</li>
  <li>Implement Private Endpoint and Private DNS for secure, name-resolvable access to Azure PaaS resources</li>
  <li>Validate routing and connectivity across the full hybrid topology</li>
</ul>

<h2>🌉 Site-to-Site VPN: FortiGate Phase 2 Selector Gaps</h2>
<p>Getting the Site-to-Site VPN stable between Azure's VPN Gateway and the home FortiGate surfaced a recurring issue with <strong>Phase 2 selectors</strong> — the IPsec proxy IDs that define which traffic is allowed to traverse the tunnel. Mismatched or incomplete selectors on the FortiGate side caused intermittent tunnel instability, since Azure and FortiGate need matching, complete traffic selectors on both ends of the negotiation to keep the tunnel consistently up.</p>

<h2>🔀 VNet Peering and Gateway Transit</h2>
<p>In a hub-spoke topology, spoke VNets need to reach on-prem resources through the hub's VPN Gateway — which requires <strong>gateway transit</strong> to be explicitly enabled on the peering connection. Troubleshooting here involved confirming the correct gateway transit flags were set on both sides of the peering relationship, since a spoke VNet without transit enabled has no path back to on-prem through the hub.</p>

<h2>📡 UDR / BGP Propagation Issues</h2>
<p>User-Defined Routes (UDRs) interacting with BGP-propagated routes from the VPN Gateway required careful attention — specifically making sure UDRs didn't unintentionally override routes that should have been learned dynamically via BGP, which could silently break connectivity for spoke subnets depending on how route priority resolved.</p>

<h2>📈 Results</h2>
<ul>
  <li>Stable Site-to-Site VPN tunnel established between Azure and the home FortiGate</li>
  <li>Point-to-Site VPN access working with Entra ID authentication</li>
  <li>Gateway transit correctly enabled, allowing spoke VNets to reach on-prem resources through the hub</li>
  <li>Private DNS resolution working correctly across hybrid environments after resolving the forwarder reordering issue</li>
  <li>Full hub-spoke connectivity validated end-to-end</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>IPsec Phase 2 selectors need to match exactly on both ends — a partial match can cause a tunnel to appear "up" while still dropping specific traffic</li>
  <li>Gateway transit is easy to overlook in hub-spoke designs — without it, spoke VNets have no path to on-prem even if the hub's VPN is fully functional</li>
  <li>UDRs and BGP-learned routes can silently conflict — route priority needs to be understood explicitly, not assumed</li>
</ul>

<h2>🔗 Related Projects</h2>
<ul>
  <li><a href="/portfolio/NSE7-OSPF-over-IPsec">NSE7 - OSPF over IPsec VPN</a></li>
</ul>

</div>
