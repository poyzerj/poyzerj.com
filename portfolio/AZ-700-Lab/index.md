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
<p>This was the hands-on project completed after earning the <strong>Microsoft Certified: Azure Network Engineer Associate (AZ-700)</strong> certification — a full hub-spoke Azure network built from scratch with hybrid connectivity back to a home lab environment, rather than a purely isolated cloud exercise.</p>

<p>Environment:</p>
<ul>
  <li>3 Azure VNets: <strong>JPHubVNet</strong> (hub), <strong>JPSpoke1VNet</strong>, and <strong>JPSpoke2VNet</strong> (spokes)</li>
  <li>Virtual Network Gateway hosted in JPHubVNet</li>
  <li>Site-to-Site VPN connecting JPHubVNet to an on-prem FortiGate</li>
  <li>Point-to-Site OpenVPN with Entra ID authentication, also terminating on JPHubVNet's gateway</li>
</ul>

<h2>🔧 Objectives</h2>
<ul>
  <li>Design and deploy a hub-spoke VNet architecture with proper peering and gateway transit</li>
  <li>Establish a Site-to-Site VPN tunnel between Azure and an on-prem FortiGate</li>
  <li>Configure Point-to-Site VPN access using Entra ID authentication</li>
  <li>Enable transitive routing between the two spoke VNets, which peering alone doesn't provide</li>
  <li>Validate routing and connectivity across the full hybrid topology</li>
</ul>

<h2>🔀 Hub-Spoke Topology and Gateway Transit</h2>
<p>JPHubVNet held the Virtual Network Gateway and served as the hub. Peering was configured between JPSpoke1VNet and JPHubVNet, and separately between JPSpoke2VNet and JPHubVNet, with traffic allowed across each peering connection. <strong>Gateway transit</strong> was enabled on both peerings, allowing both spoke VNets to use JPHubVNet's VPN Gateway — which is what let both the Site-to-Site tunnel (to the on-prem FortiGate) and the Point-to-Site VPN reach resources in either spoke, not just the hub.</p>

<h2>🔁 Making the Peerings Transitive</h2>
<p>VNet peering by itself isn't transitive — JPSpoke1VNet and JPSpoke2VNet being peered to JPHubVNet doesn't automatically let them talk to <em>each other</em>. To solve that, a route table was created for each spoke VNet, with a <strong>User-Defined Route (UDR)</strong> pointing to the other spoke's subnet, using the <strong>Virtual Network Gateway as the next hop</strong>. That forced spoke-to-spoke traffic to route through JPHubVNet's gateway rather than having no path at all.</p>

<p><strong>Route Propagation</strong> also had to be enabled on each spoke's route table, so that routes learned from on-prem via the Site-to-Site VPN would actually propagate down into the spoke VNets — without it, the spokes would have no route back to the on-prem FortiGate's network even with the UDR and gateway transit in place.</p>

<h2>🏠 On-Prem Side</h2>
<p>The on-prem side of the Site-to-Site VPN was a <strong>FortiGate firewall</strong>, terminating the IPsec tunnel back to JPHubVNet's Virtual Network Gateway.</p>

<h2>📈 Results</h2>
<ul>
  <li>Stable Site-to-Site VPN tunnel established between JPHubVNet and the on-prem FortiGate</li>
  <li>Point-to-Site VPN access working with Entra ID authentication</li>
  <li>Gateway transit enabled on both spoke peerings, letting both spokes reach on-prem resources through JPHubVNet</li>
  <li>Spoke-to-spoke routing working via UDRs pointing to the Virtual Network Gateway, with Route Propagation enabled to bring in on-prem routes</li>
  <li>Full hub-spoke connectivity validated end-to-end</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>VNet peering is not transitive by default — two spokes peered to the same hub still need explicit UDRs (with the gateway as next hop) to route traffic between each other</li>
  <li>Gateway transit is easy to overlook in hub-spoke designs — without it, spoke VNets have no path to on-prem even if the hub's VPN is fully functional</li>
  <li>Route Propagation has to be explicitly enabled on spoke route tables for on-prem routes learned to actually reach the spokes — a UDR alone doesn't bring in dynamically learned routes</li>
  <li>Hybrid connectivity labs are significantly more valuable than isolated cloud-only exercises, since they surface real routing interactions (like transitive routing across a hub-spoke design) that a pure Azure-to-Azure lab never would</li>
</ul>

</div>
