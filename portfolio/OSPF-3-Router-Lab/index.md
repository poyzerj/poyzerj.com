---
permalink: /projects/OSPF-3-Router-Lab/
title: "OSPF - 3 Router Lab"
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
<p>This lab demonstrates <strong>OSPF adjacency formation, neighbor discovery, and route exchange</strong> using a 3-router straight-line topology. All routers are configured in <strong>Area 0</strong>, and each router uses its <strong>loopback interface as its OSPF router ID</strong>.</p>

<p>Routers:</p>
<ul>
  <li>R1 – Router ID: 1.1.1.1</li>
  <li>R2 – Router ID: 2.2.2.2</li>
  <li>R3 – Router ID: 3.3.3.3</li>
</ul>

<h2>📡 Topology</h2>

<img width="620" height="136" alt="image" src="https://github.com/user-attachments/assets/0947ee3d-023e-473e-bd39-abbff11156af" />

<h2>🔧 Objectives</h2>
<ul>
  <li>Configure IP addresses on all routers</li>
  <li>Enable OSPF on all interfaces</li>
  <li>Advertise loopbacks and connected networks</li>
  <li>Verify OSPF neighbor adjacencies</li>
  <li>Validate that all loopback networks are reachable across the topology</li>
</ul>

<h2>🧪 Commands Used</h2>
<pre><code>show ip ospf neighbor
show ip route ospf
show ip protocols
show ip ospf interface
ping &lt;loopback-address&gt;
</code></pre>

<h2>📈 Results</h2>
<ul>
  <li>Full OSPF adjacencies formed between all routers</li>
  <li>Loopback networks (1.1.1.1, 2.2.2.2, 3.3.3.3) are reachable across all routers</li>
  <li>Routing table shows OSPF-learned routes for each loopback</li>
  <li>Convergence confirmed with ping tests</li>
</ul>

<h2>📁 Configuration Files</h2>
<ul>
  <li><a href="https://raw.githubusercontent.com/poyzerj/network-portfolio/refs/heads/main/projects/OSPF-3-Router-Lab/R1.txt">R1.txt</a> – OSPF and IP configuration</li>
  <li><a href="https://raw.githubusercontent.com/poyzerj/network-portfolio/refs/heads/main/projects/OSPF-3-Router-Lab/R2.txt">R2.txt</a> – OSPF and IP configuration</li>
  <li><a href="https://raw.githubusercontent.com/poyzerj/network-portfolio/refs/heads/main/projects/OSPF-3-Router-Lab/R3.txt">R3.txt</a> – OSPF and IP configuration</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Loopbacks are useful as stable <strong>OSPF router IDs</strong></li>
  <li>OSPF adjacency forms only on interfaces with matching hello/dead timers and area configuration</li>
  <li>Network statements determine which interfaces participate in OSPF</li>
  <li>Straight-line topologies illustrate convergence and route propagation clearly</li>
</ul>

</div>
