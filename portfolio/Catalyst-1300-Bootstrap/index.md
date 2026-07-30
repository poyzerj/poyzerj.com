---
permalink: /portfolio/Catalyst-1300-Bootstrap/
title: "Catalyst 1300 Bootstrap Script"
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
<p>A Python automation script that bootstraps a factory-default Cisco Catalyst 1300 switch over SSH — handling the forced first-login password change, pushing a baseline VLAN/trunk config, and replacing the factory <code>cisco</code> account with two dedicated, randomly-generated-password accounts.</p>

<p><strong>Repo:</strong> <a href="https://github.com/poyzerj/NetworkAutomation/tree/main/catalyst1300_bootstrap">github.com/poyzerj/NetworkAutomation/catalyst1300_bootstrap</a></p>

<h2>🔧 What It Does</h2>
<ul>
  <li>Connects via SSH using the factory default <code>cisco</code>/<code>cisco</code> credentials</li>
  <li>Drives the forced first-login password-change dialog automatically</li>
  <li>Creates VLANs 10, 20, and 30</li>
  <li>Configures <code>gi0/48</code> as a trunk allowing those VLANs</li>
  <li>Creates two priv15 accounts — an admin account and a dedicated monitoring account — each with a randomly generated password</li>
  <li>Removes the factory <code>cisco</code> account once both new accounts are confirmed working</li>
  <li>Saves the running config to startup config</li>
</ul>

<h2>🚧 The Prompt-Detection Problem</h2>
<p>The Catalyst 1300 / CBS family runs an "S300-style" CLI that looks like standard IOS but isn't quite. Netmiko's <code>cisco_s300</code> driver is a reasonable starting point, but prompt auto-detection can fail — especially on first login, when the switch forces an interactive old/new/confirm password dialog before it ever presents a normal CLI prompt.</p>

<p>The fix: <code>connect.py</code> tries the <code>cisco_s300</code> driver first, then falls back to a raw, manually-driven session to walk through that forced password-change dialog step by step if the automatic driver can't lock onto a prompt.</p>

<h2>🔒 Credential Handling</h2>
<p>Passwords for the new admin and monitoring accounts are generated fresh on every run — they're never read from or written to the config file. After a run, they're printed once to the console and written to a gitignored local credentials log, meant to be moved into a password manager and deleted. Every config command sent and the switch's response is also logged, with generated passwords redacted from that log specifically (only the one-time credentials file has them in full).</p>

<h2>📈 Result</h2>
<p>A repeatable, safe way to bring a brand-new Catalyst 1300 from factory defaults to a baseline secure state — VLANs configured, trunk in place, and no factory default credentials left active — without needing to manually walk through the CLI's quirky first-login flow every time.</p>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Not every "IOS-like" CLI is actually IOS — the S300 family's prompt behavior during first login required a manual fallback path rather than trusting the standard Netmiko driver in every case</li>
  <li>Generating credentials at runtime rather than storing them in a config file removes an entire class of accidental credential leakage (e.g., committing a config file with a real password in it)</li>
  <li>Only removing the factory account after confirming the replacement accounts work avoids a failure mode where a script bug could lock out all access to the switch</li>
</ul>

</div>
