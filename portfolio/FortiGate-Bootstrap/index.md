---
permalink: /portfolio/FortiGate-Bootstrap/
title: "FortiGate Bootstrap Script"
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
<p>A Python automation script that bootstraps a FortiGate from its factory-default (or current) state over SSH — setting the admin password, configuring WAN and LAN interfaces, enabling the LAN DHCP server, and optionally provisioning a REST API admin account and token.</p>

<p><strong>Repo:</strong> <a href="https://github.com/poyzerj/NetworkAutomation/tree/main/fortigate_bootstrap">github.com/poyzerj/NetworkAutomation/fortigate_bootstrap</a></p>

<h2>🔧 What It Does</h2>
<ol>
  <p>
  <li>Checks reachability, then SSHes in as <code>admin</code> with a blank password, handling the forced password-change prompt</li>
  <li>If the LAN interface is bundled into a hardware switch, breaks it out into an independent interface first</li>
  <li>Pushes WAN configuration (static IP, gateway, DNS)</li>
  <li>Pushes LAN configuration (static IP, DHCP server) — <strong>last</strong>, since this is the point where the session is expected to drop</li>
  <li>Optionally reconnects at the new LAN address to create a REST API admin account and print a one-time token</li>
  </p>
</ol>

<h2>🖥️ Two Templates: Hardware vs. Virtual</h2>
<p>The script ships with two config templates, since FortiGate's interface behavior differs meaningfully between platforms:</p>
<ul>
  <li><strong>Physical FortiGate template</strong> — desktop models with a built-in LAN switch bundle their physical LAN ports into a single virtual-switch interface, which has to be explicitly broken out before it can take its own static IP</li>
  <li><strong>EVE-NG / virtual lab template</strong> — virtual FortiGate images expose <code>port1</code>/<code>port2</code> as independent interfaces from the start, so no switch breakout step is needed</li>
</ul>

<h2>⚠️ Designed Around an Expected Session Drop</h2>
<p>One of the trickier design problems here: the script reconfigures the LAN interface's IP address <em>while connected to the FortiGate through that same interface</em>. That means the SSH session dropping partway through a run isn't a bug — it's expected behavior, and the script logs it as such rather than treating it as a failure. If a REST API account is requested, the script reconnects at the <strong>new</strong> LAN address afterward, which doubles as confirmation that the LAN reconfiguration actually took effect.</p>

<h2>🔒 Credential & Token Handling</h2>
<p>The admin password lives only in the gitignored config file. If a REST API account is created, the generated token is printed once to the console and deliberately never written to the log file, since a long-lived API token in a log would be a real credential exposure risk.</p>

<h2>📈 Result</h2>
<p>A repeatable way to take a FortiGate from factory defaults (or any known starting state) to a working WAN/LAN configuration with DHCP running, in a single scripted run — with the interface-breakout logic adapting to whether the target is physical hardware or a virtual lab image.</p>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Virtual-switch behavior on FortiGate hardware varies noticeably across models and firmware — the breakout logic is documented as a best-effort default, with an explicit note to verify <code>show system virtual-switch</code> output against the config before running on unfamiliar hardware</li>
  <li>A script that reconfigures its own connection path needs to treat the resulting disconnect as an expected state, not an error — logging it correctly instead of surfacing a false failure was a deliberate design choice</li>
  <li>Sequencing matters: pushing WAN config first (which doesn't touch the active session) before LAN config (which does) minimizes the risk window and keeps the drop predictable and expected, rather than happening at a random point mid-run</li>
</ul>


</div>
