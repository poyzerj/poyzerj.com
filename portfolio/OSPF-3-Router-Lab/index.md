---
permalink: /portfolio/OSPF-3-Router-Lab/
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
  cursor: pointer;
}

.lab-content a:hover {
  text-decoration: underline;
}

.lab-content strong {
  font-weight: bold;
}

/* Modal Styles */
.modal {
  display: none;
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background-color: rgba(0,0,0,0);
}

.modal-content {
  background-color: #ffffff;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 0;
  border: none;
  outline: none;
  width: 90%;
  max-width: 900px;
  max-height: 80vh;
  border-radius: 0;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
  display: flex;
  flex-direction: column;
}

.modal-header {
  padding: 16px 20px;
  background-color: #ffffff;
  border-bottom: none;
  border-radius: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}

.modal-header h3 {
  margin: 0;
  font-size: 19px;
  font-weight: bold;
}

.close {
  color: #aaa;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
  line-height: 20px;
}

.close:hover,
.close:focus {
  color: #000;
}

.modal-body {
  padding: 0;
  overflow-y: auto;
  flex: 1;
  background-color: #ffffff;
}

.modal-body::-webkit-scrollbar {
  width: 12px;
}

.modal-body::-webkit-scrollbar-track {
  background: #ffffff;
}

.modal-body::-webkit-scrollbar-thumb {
  background: #cccccc;
  border-radius: 6px;
}

.modal-body::-webkit-scrollbar-thumb:hover {
  background: #999999;
}

.modal-body > div {
  padding: 20px;
}

.modal-body pre {
  background-color: #ffffff;
  padding: 0;
  border-radius: 0;
  overflow-x: auto;
  margin: 0;
  font-size: 14px;
  line-height: 1.5;
  text-align: left;
}

.modal-body code {
  font-family: 'Courier New', monospace;
  font-size: 14px;
  text-align: left;
  display: block;
}

.loading {
  text-align: center;
  padding: 40px;
  color: #666;
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
  <li><a onclick="openConfigModal('R1', 'https://raw.githubusercontent.com/poyzerj/poyzerj.com/refs/heads/main/portfolio/OSPF-3-Router-Lab/R1.txt')">R1.txt</a> – OSPF and IP configuration</li>
  <li><a onclick="openConfigModal('R2', 'https://raw.githubusercontent.com/poyzerj/poyzerj.com/refs/heads/main/portfolio/OSPF-3-Router-Lab/R2.txt')">R2.txt</a> – OSPF and IP configuration</li>
  <li><a onclick="openConfigModal('R3', 'https://raw.githubusercontent.com/poyzerj/poyzerj.com/refs/heads/main/portfolio/OSPF-3-Router-Lab/R3.txt')">R3.txt</a> – OSPF and IP configuration</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Loopbacks are useful as stable <strong>OSPF router IDs</strong></li>
  <li>OSPF adjacency forms only on interfaces with matching hello/dead timers and area configuration</li>
  <li>Network statements determine which interfaces participate in OSPF</li>
  <li>Straight-line topologies illustrate convergence and route propagation clearly</li>
</ul>

</div>

<!-- Modal -->
<div id="configModal" class="modal">
  <div class="modal-content">
    <div class="modal-header">
      <h3 id="modalTitle">Configuration File</h3>
      <span class="close" onclick="closeConfigModal()">&times;</span>
    </div>
    <div class="modal-body">
      <div id="modalContent" class="loading">Loading...</div>
    </div>
  </div>
</div>

<script>
function openConfigModal(routerName, url) {
  const modal = document.getElementById('configModal');
  const modalTitle = document.getElementById('modalTitle');
  const modalContent = document.getElementById('modalContent');
  
  modalTitle.textContent = routerName + '.txt - Configuration';
  modalContent.innerHTML = '<div class="loading">Loading configuration...</div>';
  modal.style.display = 'block';
  
  fetch(url)
    .then(response => response.text())
    .then(data => {
      modalContent.innerHTML = '<div><pre><code>' + escapeHtml(data) + '</code></pre></div>';
    })
    .catch(error => {
      modalContent.innerHTML = '<div class="loading">Error loading configuration file.</div>';
      console.error('Error:', error);
    });
}

function closeConfigModal() {
  document.getElementById('configModal').style.display = 'none';
}

function escapeHtml(text) {
  const map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#039;'
  };
  return text.replace(/[&<>"']/g, m => map[m]);
}

// Close modal when clicking outside of it
window.onclick = function(event) {
  const modal = document.getElementById('configModal');
  if (event.target == modal) {
    closeConfigModal();
  }
}

// Close modal with Escape key
document.addEventListener('keydown', function(event) {
  if (event.key === 'Escape') {
    closeConfigModal();
  }
});
</script>
