# OSPF 3-Router Lab

## 📌 Overview
This lab demonstrates **OSPF adjacency formation, neighbor discovery, and route exchange** using a 3-router straight-line topology. All routers are configured in **Area 0**, and each router uses its **loopback interface as its OSPF router ID**.

Routers:  
- R1 – Router ID: 1.1.1.1  
- R2 – Router ID: 2.2.2.2  
- R3 – Router ID: 3.3.3.3  

---

## 📡 Topology

*(Insert topology screenshot here: `topology.png`)*

---

## 🔧 Objectives
- Configure IP addresses on all routers  
- Enable OSPF on all interfaces  
- Advertise loopbacks and connected networks  
- Verify OSPF neighbor adjacencies  
- Validate that all loopback networks are reachable across the topology  

---

## 🧪 Commands Used

show ip ospf neighbor
show ip route ospf
show ip protocols
show ip ospf interface
ping <loopback-address>

---

## 📈 Results
- Full OSPF adjacencies formed between all routers  
- Loopback networks (1.1.1.1, 2.2.2.2, 3.3.3.3) are reachable across all routers  
- Routing table shows OSPF-learned routes for each loopback  
- Convergence confirmed with ping tests  

---

## 📁 Configuration Files
- `R1.txt` – OSPF and IP configuration  
- `R2.txt` – OSPF and IP configuration  
- `R3.txt` – OSPF and IP configuration  

---

## 📝 Notes / Lessons Learned
- Loopbacks are useful as stable **OSPF router IDs**  
- OSPF adjacency forms only on interfaces with matching hello/dead timers and area configuration  
- Network statements determine which interfaces participate in OSPF  
- Straight-line topologies illustrate convergence and route propagation clearly
