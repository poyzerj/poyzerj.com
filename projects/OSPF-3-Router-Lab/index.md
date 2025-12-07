---
permalink: /OSPF-2-Router-Lab/
title: "OSPF - 3 Router Lab"
layout: single
author_profile: true
---

## 📌 Overview
This lab demonstrates **OSPF adjacency formation, neighbor discovery, and route exchange** using a 3-router straight-line topology. All routers are configured in **Area 0**, and each router uses its **loopback interface as its OSPF router ID**.

Routers:  
- R1 – Router ID: 1.1.1.1  
- R2 – Router ID: 2.2.2.2  
- R3 – Router ID: 3.3.3.3  

---

## 📡 Topology

<img width="620" height="136" alt="image" src="https://github.com/user-attachments/assets/0947ee3d-023e-473e-bd39-abbff11156af" />

---

## 🔧 Objectives
- Configure IP addresses on all routers  
- Enable OSPF on all interfaces  
- Advertise loopbacks and connected networks  
- Verify OSPF neighbor adjacencies  
- Validate that all loopback networks are reachable across the topology  

---

## 🧪 Commands Used
```
show ip ospf neighbor
show ip route ospf
show ip protocols
show ip ospf interface
ping <loopback-address>
```
---

## 📈 Results
- Full OSPF adjacencies formed between all routers  
- Loopback networks (1.1.1.1, 2.2.2.2, 3.3.3.3) are reachable across all routers  
- Routing table shows OSPF-learned routes for each loopback  
- Convergence confirmed with ping tests  

---

## 📁 Configuration Files
- [R1.txt](https://raw.githubusercontent.com/poyzerj/network-portfolio/refs/heads/main/OSPF-3-Router-Lab/R1.txt) – OSPF and IP configuration
- [R2.txt](https://raw.githubusercontent.com/poyzerj/network-portfolio/refs/heads/main/OSPF-3-Router-Lab/R2.txt) – OSPF and IP configuration  
- [R3.txt](https://raw.githubusercontent.com/poyzerj/network-portfolio/refs/heads/main/OSPF-3-Router-Lab/R3.txt) – OSPF and IP configuration  

---

## 📝 Notes / Lessons Learned
- Loopbacks are useful as stable **OSPF router IDs**  
- OSPF adjacency forms only on interfaces with matching hello/dead timers and area configuration  
- Network statements determine which interfaces participate in OSPF  
- Straight-line topologies illustrate convergence and route propagation clearly
