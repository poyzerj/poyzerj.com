---
permalink: /portfolio/AZ-500-Lab/
title: "AZ-500 - Securing an Azure Hub-Spoke Network"
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
  cursor: zoom-in;
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

.lab-content em {
  font-size: 15px;
  color: #666;
}

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
  padding: 40px;
  align-items: center;
  justify-content: center;
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
  max-width: 1100px;
  max-height: 90vh;
  border-radius: 8px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.2);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.modal-header {
  padding: 16px 20px;
  background-color: #ffffff;
  border-bottom: none;
  border-radius: 8px 8px 0 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}

.modal-header h3 {
  margin: 0;
  font-size: 17px;
  font-weight: bold;
}

.modal .close {
  color: #aaa;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
  line-height: 20px;
}

.modal .close:hover,
.modal .close:focus {
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

.modal-body img {
  width: 100%;
  height: auto;
  display: block;
  margin: 0;
  cursor: default;
}
</style>

<div class="lab-content">

<h2>📌 Overview</h2>
<p>This project applies the <strong>Secure Networking</strong> domain of the AZ-500 (Azure Security Engineer Associate) exam — at 20–25%, the largest single domain on the exam — to a real environment rather than a standalone lab. The hub-spoke topology itself isn't new: it's the same VNets, VPN Gateway, and hybrid connectivity built for the <a href="/portfolio/AZ-700-Lab">AZ-700 project</a>, renamed and reused. What's new is everything layered on top of it: NSGs and ASGs, a centralized Azure Firewall, user-defined routes, and a Network Watcher validation pass that ended up being the most instructive part of the whole build.</p>

<p>Environment:</p>
<ul>
  <li>Hub VNet (<strong>JPVNetHub</strong>) with a Virtual Network Gateway and a centralized Azure Firewall</li>
  <li>Two spoke VNets (<strong>JPVNetSpoke1</strong>, <strong>JPVNetSpoke2</strong>) peered to the hub, each running one workload VM</li>
  <li>Site-to-Site VPN to an on-prem FortiGate, Point-to-Site VPN authenticated through Microsoft Entra ID</li>
  <li>Three VMs (<strong>JPAZVM11</strong> hub, <strong>JPAZVM12</strong> Spoke1, <strong>JPAZVM13</strong> Spoke2), each behind its own NSG and Application Security Group</li>
</ul>

<h2>🔧 Objectives</h2>
<ul>
  <li>Apply defense-in-depth network controls — NSGs/ASGs, centralized firewall, UDRs — to an existing topology instead of a disposable lab</li>
  <li>Get hands-on with the specific AZ-500 Secure Networking skills: segmentation, egress control, VPN connectivity, and Network Watcher diagnostics</li>
  <li>Validate every control live rather than assuming configuration equals enforcement</li>
  <li>Produce a portfolio-ready case study, including the troubleshooting that didn't go as expected</li>
</ul>

<h2>🔒 Segmentation: NSGs & ASGs</h2>
<p>The first surprise of this project showed up before any new resources were even deployed: peering two VNets together does <strong>not</strong> segment them. Azure's platform-default <code>AllowVnetInBound</code> rule (priority 65000) quietly allows everything inside the peered address space, and it can't be deleted — only out-prioritized. Phase 1 added an explicit <code>Deny-VirtualNetwork-Inbound</code> rule (priority 4000) and <code>Deny-Internet-Inbound</code> (4010) to all three NSGs, then layered narrow allows above them for exactly what's needed: RDP and ICMP from on-prem, plus RDP from the hub as a jump-host path.</p>

<img src="/portfolio/AZ-500-Lab/06-nsg-deny-rules-spoke1.png" alt="JPNSGSpoke1 inbound security rules showing Deny-VirtualNetwork-Inbound and Deny-Internet-Inbound ranked above the platform default AllowVnetInBound" onclick="openImageModal('/portfolio/AZ-500-Lab/06-nsg-deny-rules-spoke1.png', 'JPNSGSpoke1 — Inbound Security Rules')" />
<p><em>JPNSGSpoke1's full rule set, platform defaults included — the explicit denies at 4000/4010 sit well above AllowVnetInBound at 65000. (Click to enlarge.)</em></p>

<p>Application Security Groups (<code>JPASGHub</code>, <code>JPASGSpoke1</code>, <code>JPASGSpoke2</code>) sit between the NSG rules and the VMs so rules reference a role instead of a hardcoded IP. With one VM per role today that's not saving much, but it means the rules don't need to change if a role ever grows past one VM.</p>

<img src="/portfolio/AZ-500-Lab/07-asg-membership-spoke1.png" alt="JPASGSpoke1 Application Security Group overview showing it tied to JPAZVM12-nic (10.1.0.4), attached to JPAZVM12" onclick="openImageModal('/portfolio/AZ-500-Lab/07-asg-membership-spoke1.png', 'JPASGSpoke1 — Membership')" />
<p><em>JPASGSpoke1's membership — tied to JPAZVM12's NIC, the same object referenced as a destination in the NSG rules above. (Click to enlarge.)</em></p>

<h2>🔥 Centralized Firewall & Egress Control</h2>
<p>Segmentation handles north-south trust at the NSG layer, but spoke-to-spoke traffic over VNet peering has no inspection point in the path at all by default — peering just connects, it doesn't route through anything. Phase 2 deployed an Azure Firewall (Standard SKU) into the hub and built a policy with a network rule permitting HTTPS egress from both spokes and an application rule allowing Windows Update by FQDN tag, with Threat Intelligence in Alert mode to start.</p>

<img src="/portfolio/AZ-500-Lab/08-firewall-rule-collections.png" alt="JPAZFW01-Policy rule collections showing rcg-spoke-egress with net-allow-required (network, priority 100) and app-allow-updates (application, priority 200)" onclick="openImageModal('/portfolio/AZ-500-Lab/08-firewall-rule-collections.png', 'Firewall Policy — Rule Collections')" />
<p><em>The policy's two rule collections: baseline HTTPS egress, and Windows Update by FQDN tag. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/09-firewall-threat-intelligence.png" alt="JPAZFW01-Policy Threat Intelligence setting showing Alert Only mode" onclick="openImageModal('/portfolio/AZ-500-Lab/09-firewall-threat-intelligence.png', 'Firewall Policy — Threat Intelligence')" />
<p><em>Threat Intelligence set to Alert Only — validate against real lab traffic before tightening to Alert and Deny. (Click to enlarge.)</em></p>

<h2>⚠️ The UDR Precedence Gotcha</h2>
<p>This phase had its own gotcha, and it's the one worth remembering: Azure resolves routing by longest-prefix match, and a user-defined route only beats a system route at <strong>equal or greater</strong> specificity. The automatic peering route between the two spokes and the explicit UDR pointing spoke-to-spoke traffic at the firewall were originally both <code>/16</code> prefixes — so adding only a <code>0.0.0.0/0 → Firewall</code> default route wasn't enough. Without an explicit <code>/16 → Firewall</code> route on Spoke1's route table (and the mirror on Spoke2's), the automatic peering route kept winning and spoke-to-spoke traffic silently bypassed the firewall entirely. Nothing errored. Nothing warned about it. The only way to catch it was to check Next Hop and see <code>VirtualNetwork</code> instead of <code>VirtualAppliance</code> — which is exactly what Phase 4 is for.</p>

<p>Those routes have since been tightened to the spokes' actual, resized address spaces (<code>/23</code> instead of <code>/16</code>), so they now win on specificity alone rather than relying on the UDR-over-system tiebreak — same underlying lesson either way.</p>

<img src="/portfolio/AZ-500-Lab/10-udr-route-table-spoke1.png" alt="RT-Spoke1 routes showing to-Internet (0.0.0.0/0) and to-Spoke2 (10.2.0.0/23), both Virtual Appliance to 10.0.2.4" onclick="openImageModal('/portfolio/AZ-500-Lab/10-udr-route-table-spoke1.png', 'RT-Spoke1 — Routes')" />
<p><em>RT-Spoke1: both routes pointing at the firewall's private IP. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/11-udr-route-table-spoke2.png" alt="RT-Spoke2 routes showing to-Internet (0.0.0.0/0) and to-Spoke1 (10.1.0.0/23), both Virtual Appliance to 10.0.2.4" onclick="openImageModal('/portfolio/AZ-500-Lab/11-udr-route-table-spoke2.png', 'RT-Spoke2 — Routes')" />
<p><em>RT-Spoke2, mirrored. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/12-udr-effective-routes-after.png" alt="Effective routes on JPAZVM12-nic showing the User route to 10.2.0.0/23 Active via Virtual appliance, and the Default 0.0.0.0/0 Internet route marked Invalid" onclick="openImageModal('/portfolio/AZ-500-Lab/12-udr-effective-routes-after.png', 'JPAZVM12-nic — Effective Routes')" />
<p><em>The UDR actually winning in practice — and a nice side effect: the platform's own 0.0.0.0/0 route shows as Invalid, superseded by the custom one. (Click to enlarge.)</em></p>

<h2>🔐 Secure VPN Connectivity</h2>
<p>No new build here — the Site-to-Site (custom IPsec/IKE policy, AES256/SHA256/DH Group 14/PFS2048) and Point-to-Site (Entra ID authentication, OpenVPN) connections already existed from the AZ-700 project. This phase was about reviewing what was already there and mapping it explicitly to this domain's requirements: a custom cipher suite instead of accepting Azure's default proposal, and identity-based P2S auth instead of a shared certificate.</p>

<img src="/portfolio/AZ-500-Lab/13-vpn-s2s-connected.png" alt="JPVPNGW connections showing JPAZ-to-JPHome, Site-to-site (IPsec), status Connected" onclick="openImageModal('/portfolio/AZ-500-Lab/13-vpn-s2s-connected.png', 'VPN Gateway — S2S Connection')" />
<p><em>The Site-to-Site connection to the on-prem FortiGate, Connected. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/14-vpn-s2s-ipsec-policy.png" alt="JPAZ-to-JPHome connection configuration showing custom IPsec/IKE policy: AES256 encryption, SHA256 integrity, DHGroup14, PFS2048" onclick="openImageModal('/portfolio/AZ-500-Lab/14-vpn-s2s-ipsec-policy.png', 'S2S Connection — Custom IPsec/IKE Policy')" />
<p><em>The custom cipher suite instead of Azure's default proposal. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/15-vpn-p2s-config.png" alt="JPVPNGW Point-to-site configuration showing address pool 172.16.20.0/24, OpenVPN (SSL) tunnel type, Microsoft Entra ID authentication" onclick="openImageModal('/portfolio/AZ-500-Lab/15-vpn-p2s-config.png', 'VPN Gateway — P2S Configuration')" />
<p><em>Point-to-Site: Entra ID authentication instead of a shared certificate. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/16-vpn-p2s-entra-app.png" alt="Azure VPN app registration in Microsoft Entra ID showing status Activated" onclick="openImageModal('/portfolio/AZ-500-Lab/16-vpn-p2s-entra-app.png', 'Entra ID — Azure VPN App Registration')" />
<p><em>The Entra ID side of P2S auth — the Azure VPN Client app registration, active. (Click to enlarge.)</em></p>

<h2>🔍 Visibility & Validation</h2>
<p>This phase deploys nothing — it's a validation pass against the earlier phases, run through five Network Watcher tools: Effective Security Rules, Next Hop, IP Flow Verify, NSG Diagnostics, and Connection Troubleshoot.</p>

<p>Flow logs were evaluated for this phase first, and dropped. Both NSG Flow Logs and Virtual Network Flow Logs produce deeply nested JSON with no simple allow/deny field — flow state gets folded into a single value instead — and critically, they never capture the IP Flow Verify or NSG Diagnostics tests, since neither of those tools sends a real packet; they're rule-evaluation simulations. Network Watcher's other diagnostics validate the same enforcement more directly and read far better live, so flow logging was left out of the build rather than bolted on as a sixth method.</p>

<img src="/portfolio/AZ-500-Lab/21-effective-security-rules-setup.png" alt="Network Watcher Effective Security Rules tool targeting JPAZVM12, network interface JPAZVM12-nic, showing Associated NSGs: JPNSGSpoke1" onclick="openImageModal('/portfolio/AZ-500-Lab/21-effective-security-rules-setup.png', 'Effective Security Rules — Target')" />
<p><em>Targeting JPAZVM12-nic — the tool confirms its associated NSG before showing the combined rule set. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/17-effective-security-rules.png" alt="Effective security rules on JPAZVM12-nic showing the combined inbound and outbound rule set, with Deny-VirtualNetwork-Inbound and Deny-Internet-Inbound ranked above AllowVnetInBound" onclick="openImageModal('/portfolio/AZ-500-Lab/17-effective-security-rules.png', 'JPAZVM12-nic — Effective Security Rules')" />
<p><em>The combined rule set as actually applied on the NIC — not just what the NSG's own definition says. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/22-ip-flow-verify-setup.png" alt="IP Flow Verify configured for JPAZVM12, inbound TCP, local port 3389, remote IP 192.168.20.10" onclick="openImageModal('/portfolio/AZ-500-Lab/22-ip-flow-verify-setup.png', 'IP Flow Verify — Test Configuration')" />
<p><em>Simulating inbound RDP from an on-prem address against JPAZVM12. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/18-ip-flow-verify-allow.png" alt="IP Flow Verify result showing Access allowed, matched by Allow-RDP-From-Home on JPNSGSpoke1" onclick="openImageModal('/portfolio/AZ-500-Lab/18-ip-flow-verify-allow.png', 'IP Flow Verify — Access Allowed')" />
<p><em>Inbound RDP from an on-prem source: Allowed, matched by Allow-RDP-From-Home. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/23-nsg-diagnostics-setup.png" alt="NSG Diagnostics configured for JPAZVM12, outbound traffic from 10.1.0.4 to 10.2.0.4 on port 3389" onclick="openImageModal('/portfolio/AZ-500-Lab/23-nsg-diagnostics-setup.png', 'NSG Diagnostics — Test Configuration')" />
<p><em>Tracing outbound Spoke1-to-Spoke2 RDP traffic through every NSG in the path. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/19-nsg-diagnostics-allowed.png" alt="NSG Diagnostics result showing Traffic status Allowed across every NSG evaluated for outbound Spoke1 to Spoke2 traffic" onclick="openImageModal('/portfolio/AZ-500-Lab/19-nsg-diagnostics-allowed.png', 'NSG Diagnostics — Allowed')" />
<p><em>NSG Diagnostics names every NSG in the path and the specific rule that matched — useful when a result needs explaining, not just confirming. (Click to enlarge.)</em></p>

<img src="/portfolio/AZ-500-Lab/20-next-hop-virtual-appliance.png" alt="Next Hop result from JPAZVM12 to 10.2.0.4 showing Next hop type VirtualAppliance, IP address 10.0.2.4" onclick="openImageModal('/portfolio/AZ-500-Lab/20-next-hop-virtual-appliance.png', 'Next Hop — Virtual Appliance')" />
<p><em>Spoke-to-spoke traffic routing through the firewall, confirmed directly. (Click to enlarge.)</em></p>

<h2>🚧 The Three-Layer RDP Mystery</h2>
<p>The last test in the validation plan was Connection Troubleshoot — real traffic, not a simulation — from <strong>JPAZVM12</strong> (Spoke1) to <strong>JPAZVM13</strong> (Spoke2) on port 3389. By design this was expected to come back <strong>Unreachable</strong>, since Spoke2's NSG only allows RDP from the hub subnet and on-prem, not from Spoke1 directly. It did:</p>

<img src="/portfolio/AZ-500-Lab/01-connection-troubleshoot-unreachable.png" alt="Connection Troubleshoot showing Unreachable, 316 probes sent, 316 failed" onclick="openImageModal('/portfolio/AZ-500-Lab/01-connection-troubleshoot-unreachable.png', 'Connection Troubleshoot — Unreachable')" />
<p><em>Connection Troubleshoot: Unreachable, 316/316 probes failed — the expected result. (Click to enlarge.)</em></p>

<p>That was the expected result — right up until I modified <code>Allow-RDP-From-Hub</code> to add JPAZVM12's IP as an explicit source, intending to test opening that one host through. The NSG rule now clearly allowed it. Connection Troubleshoot still said Unreachable, 316/316 failed.</p>

<p><strong>Troubleshooting approach:</strong> work outward from the NSG, since that's the layer I'd just changed. First, confirm outbound wasn't the problem — the Spoke1 NSG's outbound rules were untouched, just the platform defaults:</p>

<img src="/portfolio/AZ-500-Lab/02-nsg-outbound-rules-default.png" alt="NSG outbound rules showing only the default AllowVnetOutBound, AllowInternetOutBound, and DenyAllOutBound rules" onclick="openImageModal('/portfolio/AZ-500-Lab/02-nsg-outbound-rules-default.png', 'JPNSGSpoke1 — Outbound Security Rules')" />
<p><em>Spoke1's outbound rules, unmodified — the default AllowVnetOutBound already covers this. (Click to enlarge.)</em></p>

<p>The <code>VirtualNetwork</code> service tag includes peered VNets, so outbound from Spoke1 to Spoke2 was never the issue. With NSG outbound on Spoke1 allowed, NSG inbound on Spoke2 allowed (after my edit), and still 100% probe failure, there was exactly one checkpoint left on the path: Azure Firewall. UDRs route all Spoke1↔Spoke2 traffic through the firewall, and its policy only had a network rule for TCP 443 — nothing for 3389. <strong>Azure Firewall default-denies anything with no matching rule</strong>, so it was silently dropping every SYN regardless of what either NSG said.</p>

<p>Adding a network rule fixed it — permitting TCP 3389 from Spoke1 to Spoke2:</p>

<img src="/portfolio/AZ-500-Lab/03-firewall-rdp-rule-added.png" alt="Firewall policy rule collection showing the new net-allow-rdp rule, permitting TCP 3389 from 10.1.0.0/24 to 10.2.0.0/24" onclick="openImageModal('/portfolio/AZ-500-Lab/03-firewall-rdp-rule-added.png', 'Firewall Policy — net-allow-rdp Rule Added')" />
<p><em>The missing firewall rule, added — TCP 3389, Spoke1 to Spoke2, Allow. (Click to enlarge.)</em></p>

<p>Re-running Connection Troubleshoot: Reachable, 316/316 probes passed.</p>

<img src="/portfolio/AZ-500-Lab/04-connectivity-reachable.png" alt="Connection Troubleshoot now showing Reachable, 316 probes sent, 0 failed" onclick="openImageModal('/portfolio/AZ-500-Lab/04-connectivity-reachable.png', 'Connection Troubleshoot — Reachable')" />
<p><em>Reachable — once both the NSG and the firewall agreed. (Click to enlarge.)</em></p>

<p>Except the full diagnostic (Connectivity + NSG diagnostic + Next Hop + Port Scanner, all in one pass) told a more complete story — three checks passed clean, and a fourth came back stuck:</p>

<img src="/portfolio/AZ-500-Lab/05-full-diagnostic-port-timeout.png" alt="Full diagnostic results: Connectivity Reachable, Outbound and Inbound NSG diagnostic Allow, Next hop Success via Virtual Appliance, and Destination port accessible showing Timeout" onclick="openImageModal('/portfolio/AZ-500-Lab/05-full-diagnostic-port-timeout.png', 'Full Diagnostic — Destination Port Accessible: Timeout')" />
<p><em>Everything upstream passes — but "Destination port accessible" times out. (Click to enlarge.)</em></p>

<p>"Destination port accessible" is a different question than "can a packet get there" — it's asking whether something is actually listening on 3389 from the destination VM's own perspective. A timeout there, with everything upstream green, points past Azure entirely and into the guest OS. Sure enough: these VMs had never been RDP'd into, so Windows Defender Firewall on JPAZVM13 was still blocking RDP at its default, out-of-the-box state — a layer Network Watcher can diagnose the symptom of but can't reach to fix.</p>

<p>That's three independent checkpoints in one troubleshooting pass — NSG, Azure Firewall, and guest OS firewall — each one caught something real, each one needed a different tool and a different fix. I scripted the last one using <code>Invoke-AzVMRunCommand</code> so it can flip <code>fDenyTSConnections</code> and enable the Windows Firewall's Remote Desktop rule group over the control plane, without needing RDP to already work to get in and fix it:</p>

<pre><code>Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" `
  -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"</code></pre>

<h2>📈 Results</h2>
<ul>
  <li>Earlier phases confirmed enforcing via live diagnostics, not just reviewed configuration — Effective Security Rules, Next Hop, IP Flow Verify, and NSG Diagnostics all returned expected results</li>
  <li>Traced a real "it should work but doesn't" failure through three independent enforcement layers (NSG → Azure Firewall → guest OS) to its actual root cause at each step, rather than guessing</li>
  <li>Flow logs evaluated and deliberately left out of the final build, in favor of Network Watcher's diagnostic tools, which cover the same ground and read far better live</li>
  <li>VPN connectivity from the AZ-700 build reviewed and explicitly mapped to AZ-500's Secure Networking requirements, with no changes needed</li>
  <li>A reusable Run Command script added to the project for enabling RDP on fresh VMs ahead of a live demo, without needing the network path open first</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Passing one layer doesn't mean the request passed every layer — an NSG allow, a UDR pointing at the firewall, and the firewall's own rule set are each an independent decision point. "I allowed it" only answers one of those three questions at a time</li>
  <li>Azure Firewall fails closed and silently — with no matching rule, it just drops the packet, no NSG-style "Deny" result to point at. The only visible symptom from the client side is a timeout indistinguishable from a dozen other causes, which is why a tool that sends real traffic is worth running even after the simulated tools say everything's fine</li>
  <li>The UDR precedence issue is the kind of bug that doesn't announce itself — two routes at the same prefix length, one silently winning over the other, with no error on either side. Worth validating routing explicitly instead of assuming a UDR took effect just because it exists</li>
  <li>Raw flow log JSON looks like ground truth and isn't, for this use case — it's a legitimate record of <em>some</em> traffic, but the specific tools used to validate enforcement here never touch the wire, so the tests this project cared about never would have shown up in it regardless of how well-parsed the output was</li>
</ul>

</div>

<!-- Image Modal -->
<div id="imageModal" class="modal" style="display: none;">
  <div class="modal-content">
    <div class="modal-header">
      <h3 id="imageModalTitle">Screenshot</h3>
      <span class="close" onclick="closeImageModal()">&times;</span>
    </div>
    <div class="modal-body">
      <img id="imageModalImg" src="" alt="" />
    </div>
  </div>
</div>

<script>
function openImageModal(src, title) {
  const modal = document.getElementById('imageModal');
  const modalTitle = document.getElementById('imageModalTitle');
  const modalImg = document.getElementById('imageModalImg');

  modalTitle.textContent = title;
  modalImg.src = src;
  modalImg.alt = title;
  modal.style.display = 'flex';
}

function closeImageModal() {
  document.getElementById('imageModal').style.display = 'none';
}

// Close modal when clicking outside of it
window.addEventListener('click', function(event) {
  const modal = document.getElementById('imageModal');
  if (event.target == modal) {
    closeImageModal();
  }
});

// Close modal with Escape key
document.addEventListener('keydown', function(event) {
  if (event.key === 'Escape') {
    closeImageModal();
  }
});
</script>
