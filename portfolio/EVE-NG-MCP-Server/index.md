---
permalink: /portfolio/EVE-NG-MCP-Server/
title: "EVE-NG MCP Server"
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
.lab-content ol {
  font-size: 17px;
  line-height: 1.6;
  margin-bottom: 15px;
  padding-left: 20px;
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

<h2>Overview</h2>
<p>I wanted to be able to tell Claude Desktop "build me a 4 router OSPF lab" and have it actually show up in EVE-NG, cabled, powered on and configured. There's an open source MCP server for EVE-NG that gets you part of the way there, so I forked it, fixed what was broken in my setup and kept adding to it until it covered how I actually lab for the CCNP.</p>

<p>Now I can build a topology, push configs, run show commands across every router at once, watch a debug, save configs back into the lab and label the canvas with IPs, all from a chat window.</p>

<p><strong>Repo:</strong> <a href="https://github.com/poyzerj/eveng-mcp">github.com/poyzerj/eveng-mcp</a></p>

<p><strong>Credit:</strong> This is a fork of <a href="https://github.com/moimran/eveng-mcp">moimran/eveng-mcp</a> by Mohammed Imran. He built the original server and the EVE-NG API integration. My changes are on top of his work.</p>

<h2>My Setup</h2>
<ul>
  <li>EVE-NG Community running as a VM on Proxmox, on a Cisco UCS server in my home lab</li>
  <li>Claude Desktop on my Windows PC, running the MCP server locally over stdio (standard input/output). That just means Claude Desktop starts the server as a background program on my PC and they pass messages back and forth directly. Nothing listens on the network, so nothing else can connect to it.</li>
  <li>IOSv routers, IOSvL2 switches and VPCS for end hosts</li>
</ul>

<h2>What I Fixed</h2>
<p>When I first got it running, a few things broke right away:</p>
<ul>
  <li><strong>I kept getting kicked out of the web UI.</strong> EVE-NG only allows one login per user, so the server and my browser were fighting over the admin session. I made a separate <code>claude</code> user for the server and added automatic re-login when the session expires (EVE-NG returns HTTP 412).</li>
  <li><strong>Creating links failed.</strong> The tools were passing node IDs where the API wanted names. I added lookups so you can use either one, and interface names like <code>Gi0/1</code> or <code>GigabitEthernet0/1</code> both work.</li>
  <li><strong>Every node landed on top of each other.</strong> They were all being placed at 50,50, which EVE-NG reads as pixels, not percent. New nodes now get spread out, and there's an arrange tool to lay out a whole lab.</li>
  <li><strong>The topology view crashed</strong> because the code expected a dictionary and got a list.</li>
</ul>

<h2>What I Added</h2>

<h3>Console access</h3>
<p>The original could build a lab but couldn't talk to the devices. EVE-NG gives every node a console port. It's the same connection SecureCRT opens when I click a node in the web UI, and it only exists inside my lab network. The port number is predictable:</p>
<pre><code>def console_port(node_id: int, pod: int = 0) -&gt; int:
    return 32768 + 128 * int(pod) + int(node_id)</code></pre>
<p>I wrote a small console driver that handles the initial config dialog, waits for the prompt, enters and exits config mode on its own and stops at the first rejected line. Each device gets a lock so two jobs can't type into the same console at once. That actually happened on my SW1 before I added it, and the config was a mess.</p>

<h3>Asking before answering [confirm]</h3>
<p>Early on the driver was auto-answering prompts like <code>[confirm]</code>, which is fine until it's a reload or an erase. Now any device question gets cancelled with Ctrl+C and sent back to me. It only gets answered if I say yes and the tool is re-run with <code>confirm=true</code>.</p>
<pre><code>QUESTIONS = [
    (re.compile(r"\[confirm\]\s*$", re.I), "\r"),
    (re.compile(r"\[yes/no\]:?\s*$", re.I), "yes\r"),
    (re.compile(r"\[y/n\]:?\s*$", re.I), "y\r"),
    (re.compile(r"Destination filename \[[^\]]*\]\?\s*$"), "\r"),
]</code></pre>

<h3>Debug capture</h3>
<p>You can run a debug and have it collect output for up to 40 seconds. <code>undebug all</code> always runs afterward, even if something fails, so I don't leave a router stuck spamming the console. I tested this by debugging OSPF adjacencies on R1, shutting an interface and watching the neighbor drop and come back.</p>

<h3>Start and stop as my user</h3>
<p>The API starts nodes under whoever is logged in, so nodes started by the <code>claude</code> user were in a different POD from the ones I see in the browser. To power nodes as admin, the server SSHes into EVE-NG with a key and runs EVE-NG's own wrapper. It's the only command it's allowed to run, and the arguments are checked first:</p>
<pre><code>def build_command(action: str, pod: int, node_id: int, lab_path: str, user: str) -&gt; str:
    if action not in ACTIONS:
        raise PowerError(f"action must be one of {ACTIONS}")
    cmd = (f"{WRAPPER} -a {action} -T {int(pod)} -D {int(node_id)} "
           f"-F {shlex.quote(lab_file(lab_path))}")
    return cmd if user == "root" else "sudo -n " + cmd</code></pre>
<p>The SSH host key gets pinned the first time it connects, and the setup script swaps the password for an ed25519 key.</p>

<h3>Saving configs back into the lab</h3>
<p>I wanted a "save my progress" button. My first try used EVE-NG's export script, but it has a hard 15 second timeout and my R1 config was too long to make it. I switched to pulling <code>show running-config</code> over the console and uploading it through the API as the node's startup config:</p>
<pre><code>self.api.upload_node_config(lab_path, str(node_id), config)
self.api.enable_node_config(lab_path, str(node_id))
stored = self.api.get_node_config_by_id(lab_path, int(node_id))</code></pre>
<p>It reads the config back to make sure it stuck. There's also a backup tool that saves configs to timestamped folders on my PC, and a reset tool that wipes and reboots nodes from their saved config. Reset asks for permission first.</p>

<h3>IP labels on the canvas</h3>
<p>This was the fun one. The server reads each running device's interfaces and puts text labels on the EVE-NG canvas: interface IPs next to each link, and loopbacks and SVIs under each device. The placement logic avoids link lines, nodes, EVE's own port tags and other labels. I dropped trunk labels after trying them out because they cluttered things up. Now access VLANs only show on ports that face a PC.</p>

<h2>Testing</h2>
<p>I didn't want to test every change against my real lab, so the repo has 33 offline tests. They use a fake EVE-NG API, fake IOS consoles and a fake SSH server. They cover the console driver, the confirm handling, power commands, config saves and label placement. Run them with:</p>
<pre><code>uv run --extra dev pytest fork_tests -o addopts=""</code></pre>

<h2>Result</h2>
<p>To test the whole thing, I built an STP/RSTP lab from scratch out of a CCNP switching workbook. That meant four switches, ten links and a VPCS, with nodes created in the right order so the MAC addresses (and the root bridge election) matched the workbook. It pushed the base configs, configured the PC and labeled everything. The root bridge and root ports came out exactly like the book.</p>

<h2>Lessons Learned</h2>
<ul>
  <li>EVE-NG Community only attaches links when a node boots. If you cable a running node, nothing happens until you restart it, so cable first and power on second.</li>
  <li>IOSvL2 trunks need <code>switchport trunk encapsulation dot1q</code> before <code>switchport mode trunk</code>, just like older Catalyst switches.</li>
  <li>IOSvL2 can crash on first boot because of the PnP agent. Once it has a saved startup config, that stops.</li>
  <li>Console output is messy. Leftover output, syslog messages in the middle of a command and prompts that show up before the logs finish all broke my early versions. Most of the driver work was making it wait for the right thing.</li>
  <li>Automating a lab is only useful if it's safe. Asking before <code>[confirm]</code>, locking each console and locking down SSH mattered as much as the features did.</li>
</ul>

</div>
