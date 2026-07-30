---
permalink: /portfolio/NSE7-OSPF-over-IPsec/
title: "NSE7 - OSPF over IPsec VPN"
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
<p>This was the hands-on "Do-It" project completed as part of earning the <strong>Fortinet NSE 7 Enterprise Firewall Administrator</strong> certification — building a Site-to-Site IPsec VPN tunnel between two FortiGates, then running <strong>OSPF as a dynamic routing protocol over the tunnel</strong> rather than relying on static routes.</p>

<p>Environment:</p>
<ul>
  <li>2x FortiGate VMs, hosted on a Hyper-V server</li>
  <li>Site-to-Site IPsec VPN tunnel between the two FortiGates</li>
  <li>OSPF running over the IPsec tunnel interface</li>
</ul>

<h2>🔧 Objectives</h2>
<ul>
  <li>Establish a Site-to-Site IPsec VPN tunnel between two virtual FortiGate instances</li>
  <li>Configure OSPF to run dynamically over the IPsec tunnel interface</li>
  <li>Form OSPF adjacency across the tunnel and verify route exchange</li>
  <li>Validate that routes learned via OSPF are correctly installed and traffic routes as expected across the tunnel</li>
</ul>

<h2>🔒 Building the IPsec Tunnel</h2>
<p>The first phase was establishing a stable Site-to-Site IPsec tunnel between the two FortiGate VMs — configuring Phase 1 (IKE) parameters for the tunnel negotiation itself, and Phase 2 parameters defining the traffic selectors and encryption settings for the actual data traversing the tunnel.</p>

<h2>🔄 Running OSPF Over the Tunnel</h2>
<p>Rather than relying on static routes to reach networks on the far side of the tunnel — the more common, simpler approach — OSPF was configured to run directly over the IPsec tunnel interface. This meant treating the tunnel interface like any other OSPF-enabled interface, forming a neighbor adjacency across it, and letting routes propagate dynamically rather than being manually maintained.</p>

<p>This is a more advanced and more resilient design than static routing: if the topology on either side changes (new subnets added, routes removed), OSPF adapts automatically rather than requiring manual route updates on both FortiGates every time something changes.</p>

<h2>📈 Results</h2>
<ul>
  <li>Stable Site-to-Site IPsec tunnel established between both FortiGate VMs</li>
  <li>OSPF neighbor adjacency formed successfully across the tunnel interface</li>
  <li>Routes learned dynamically via OSPF and correctly installed in the routing table on both sides</li>
  <li>Validated end-to-end connectivity and traffic routing across the tunnel using OSPF-learned routes</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Running a dynamic routing protocol over a VPN tunnel requires the tunnel interface to be treated the same as a physical interface for routing purposes — adjacency formation depends on the tunnel being stable first</li>
  <li>OSPF over IPsec is significantly more scalable than static routing for environments where the network topology on either end is expected to change over time</li>
  <li>Getting Phase 1/Phase 2 IPsec parameters correct is a prerequisite for everything else — a flapping or unstable tunnel will prevent OSPF adjacency from ever forming reliably, so tunnel stability has to be confirmed before troubleshooting routing</li>
</ul>

<h2>🔗 Related Projects</h2>
<ul>
  <li><a href="/portfolio/AZ-700-Lab">AZ-700 Do-It: Azure Hub-Spoke Network with Hybrid Connectivity</a></li>
</ul>

</div>
