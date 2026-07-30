---
permalink: /portfolio/AVD-Lab/
title: "Azure Virtual Desktop Lab"
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
<p>This lab covers deploying an <strong>Azure Virtual Desktop (AVD)</strong> environment from scratch — host pool, session hosts, application groups, and FSLogix profile containers — including troubleshooting a RemoteApp publishing issue that wasn't visible from any single setting in the portal.</p>

<p>Environment:</p>
<ul>
  <li>Azure Virtual Desktop host pool (pooled, Windows 11 multi-session)</li>
  <li>FSLogix profile containers for user profile management</li>
  <li>RemoteApp application groups publishing individual apps (Chrome, Word)</li>
  <li>Desktop application group for full desktop sessions</li>
</ul>

<h2>🔧 Objectives</h2>
<ul>
  <li>Stand up a functional AVD host pool with session hosts</li>
  <li>Configure FSLogix for user profile roaming</li>
  <li>Publish both RemoteApps and a full desktop experience</li>
  <li>Validate the app feed for test users end-to-end</li>
</ul>

<h2>⚠️ Licensing Gotcha</h2>
<p>Early in the build, session hosts weren't deploying correctly for <strong>Windows 11 multi-session</strong> — the fix ended up being a licensing checkbox that's easy to miss in the deployment wizard, confirming the host pool's session host image and licensing are actually entitled for multi-session before deployment.</p>

<h2>🚧 The RemoteApp Feed Problem</h2>
<p>Once the host pool and session hosts were up, published RemoteApps (Chrome, Word) weren't showing up in the feed for test users — even though the deployment appeared fully configured. Every visible setting looked correct.</p>

<p><strong>Troubleshooting approach:</strong> rather than guessing, went layer by layer using PowerShell to check the deployment directly:</p>
<pre><code>Get-AzWvdApplication
Get-AzWvdApplicationGroup
Get-AzRoleAssignment
Get-AzWvdWorkspace
</code></pre>

<p>All of it came back clean, which meant the problem wasn't a misconfiguration in any single place — it had to be an interaction between settings.</p>

<p><strong>Root cause:</strong> a user assigned to both a <strong>Desktop application group</strong> and a <strong>RemoteApp application group</strong> on the <em>same host pool</em> caused the broker to suppress the RemoteApp feed entirely. Removing the user's direct assignment wasn't enough on its own — the group-based assignment also had to be cleared, since either path alone still caused the conflict.</p>

<h2>📈 Results</h2>
<ul>
  <li>Session hosts deployed correctly for Windows 11 multi-session after fixing the licensing configuration</li>
  <li>RemoteApp feed worked correctly for test users once both assignment paths (direct and group-based) were cleared</li>
  <li>FSLogix profile containers confirmed working across sessions</li>
</ul>

<h2>📝 Notes / Lessons Learned</h2>
<ul>
  <li>Some AVD issues can't be solved by checking settings individually — they require understanding how <strong>two valid-looking configurations can conflict</strong> with each other at the broker level</li>
  <li>PowerShell is often more reliable than the portal for confirming exactly what's assigned where, especially when the portal UI doesn't surface a conflict clearly</li>
  <li>Licensing/entitlement issues can masquerade as deployment failures — worth checking early rather than assuming it's a networking or image problem</li>
</ul>

</div>
