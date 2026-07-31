---
permalink: /portfolio/FortiGate-Audit-Troubleshoot/
title: "FortiGate Audit & Troubleshoot Tool"
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
<p>A Python tool, built against the FortiGate REST API, that audits a FortiGate's configuration for common issues and runs a structured troubleshooting sweep across multiple layers — interfaces, routing, VPN, system resources, policy/security, and local connectivity — rather than checking any one thing in isolation.</p>

<p><strong>Repo:</strong> <a href="https://github.com/poyzerj/NetworkAutomation/tree/main/fortigate_audit">github.com/poyzerj/NetworkAutomation/fortigate_audit</a></p>

<h2>🔧 Two Modes: Audit and Troubleshoot</h2>
<p>The tool is split into two complementary pieces:</p>
<ul>
  <li><strong>Audit mode</strong> — pulls interfaces, firewall policies, and VPN tunnel status from the FortiGate and runs a set of configuration checks against them (e.g., flagging administratively down interfaces, policies using an overly broad "ALL" service, VPN tunnels not in an "up" state)</li>
  <li><strong>Troubleshoot mode</strong> — a broader diagnostic sweep, layered by <code>Physical/Interfaces</code>, <code>Routing</code>, <code>VPN</code>, <code>System/Resources</code>, <code>Policy/Security</code>, <code>Local Connectivity</code>, and <code>DNS</code>, combining FortiGate-side API checks with local-machine diagnostics (ping, traceroute, DNS resolution) run from wherever the script executes</li>
</ul>

<h2>🧩 Design: Small, Composable Checks</h2>
<p>Rather than one large monolithic audit function, each check is a small, standalone function that takes a shared context object and returns a structured result — a name, a layer, a pass/fail state, and a list of findings. New checks are added by writing the function and registering it in a list; nothing else in the tool needs to change. Remote checks (which need a live FortiGate connection) and local checks (which run entirely on the local machine) are kept in separate registries, so either set can be run independently — useful for isolating whether a problem is on the FortiGate side or the local network path.</p>

<h2>🔍 Troubleshooting Sweep Detail</h2>
<p>The troubleshoot sweep includes checks like:</p>
<ul>
  <li>Interface link status and error/drop counters (RX/TX errors, RX/TX drops), skipping interfaces that are intentionally administratively disabled</li>
  <li>Routing table validation — confirming a default route actually exists</li>
  <li>VPN tunnel Phase 1 (IKE SA) and Phase 2 (per-selector) status</li>
  <li>CPU and memory utilization against configurable warning thresholds</li>
  <li>Local connectivity — default gateway detection and ping, external reachability, DNS resolution — run independently of the FortiGate itself, to help distinguish a FortiGate-side problem from a local network issue</li>
</ul>

<p>The CLI supports running the full sweep, a single named check in isolation (<code>--check vpn</code>), or listing all available checks — built with an eye toward a possible future interactive "diagnose this specific symptom" mode.</p>

<h2>📈 Result</h2>
<p>A reusable diagnostic tool that turns "go check the FortiGate" into a structured, repeatable sweep — useful both for a quick daily health check (audit mode) and for a deeper investigation when something's actually wrong (troubleshoot mode), without needing to remember which fifteen things to manually check every time.</p>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Separating "this data point is a genuine problem" from "this data point is informational" (via an <code>info_only</code> flag) and "the check itself failed to run" (via an <code>error</code> flag) makes the tool's output far more trustworthy than a simple pass/fail — a failed API call and a real configuration problem are very different things and shouldn't look the same in a report</li>
  <li>Building small, composable checks around a shared context object made it trivial to add the hairpin-policy check later without touching anything else in the tool — a direct payoff of the initial design choice</li>
  <li>Running local diagnostics (ping, traceroute, DNS) alongside FortiGate-side API checks helps separate "the firewall is misconfigured" from "the problem isn't the firewall at all" — a distinction that matters a lot when triaging a real issue under time pressure</li>
</ul>


</div>
