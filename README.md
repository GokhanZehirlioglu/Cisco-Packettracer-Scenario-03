# Cisco-Packettracer-Scenario-03
L3 Switch-inter-Vlan-Routing-

## Überblick
Dieses Projekt zeigt die Konfiguration eines Cisco **3560-24PS (L3-Switch)** mit zwei VLANs (**Verkauf** und **Verwaltung**) und Inter-VLAN-Routing.  
Zusätzlich wird ein **Cisco 1941 Router** eingebunden, um den PCs den Zugang zum Internet über NAT zu ermöglichen.  

- **Technologien:** VLAN, Inter-VLAN Routing, SVI, Routed Ports, Statische Routen, NAT (PAT)  
- **Keine Verwendung von:** Router-on-a-Stick, Trunk-Ports, DNS  

---

## Netzwerk-Topologie

PC-V1 ---- Gi1/0/1 |
PC-V2 ---- Gi1/0/2 |-- VLAN 10 (Verkauf)
PC-V3 ---- Gi1/0/3 |

PC-W1 ---- Gi1/0/4 |
PC-W2 ---- Gi1/0/5 |-- VLAN 20 (Verwaltung)
PC-W3 ---- Gi1/0/6 |

Router Gi0/0 (10.0.0.2) -- Gi1/0/24 (10.0.0.1) -- 3560-24PS (L3) -- VLAN 10/20

yaml
Code kopieren

---

## IP-Adressierung

| VLAN / Gerät | Interface | IP-Adresse | Subnetz | Gateway |
|--------------|-----------|------------|---------|---------|
| VLAN 10 (Verkauf) | Vlan10 (Switch) | 192.168.10.1 | /24 | – |
| PC-V1 | NIC | 192.168.10.11 | /24 | 192.168.10.1 |
| PC-V2 | NIC | 192.168.10.12 | /24 | 192.168.10.1 |
| PC-V3 | NIC | 192.168.10.13 | /24 | 192.168.10.1 |
| VLAN 20 (Verwaltung) | Vlan20 (Switch) | 192.168.20.1 | /24 | – |
| PC-W1 | NIC | 192.168.20.11 | /24 | 192.168.20.1 |
| PC-W2 | NIC | 192.168.20.12 | /24 | 192.168.20.1 |
| PC-W3 | NIC | 192.168.20.13 | /24 | 192.168.20.1 |
| Switch ↔ Router | Gi1/0/24 (Switch) | 10.0.0.1 | /30 | – |
| Router ↔ Switch | Gi0/0 (Router) | 10.0.0.2 | /30 | – |
| Router extern | Gi0/1 | 200.0.0.2 | /30 | 200.0.0.1 |

---

## Konfigurationsschritte

### Switch (3560-24PS)

! VLANs erstellen
vlan 10
 name VERKAUF
vlan 20
 name VERWALTUNG

! Ports zuordnen
interface range gi1/0/1 - 3
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast

interface range gi1/0/4 - 6
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast

! Gateways (SVI)
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown

! Routing aktivieren
ip routing

! Routed Port zum Router
interface gi1/0/24
 no switchport
 ip address 10.0.0.1 255.255.255.252
 no shutdown

! Default Route zum Router
ip route 0.0.0.0 0.0.0.0 10.0.0.2



### Router (1941)

! Interne Schnittstelle
interface gi0/0
 ip address 10.0.0.2 255.255.255.252
 ip nat inside
 no shutdown

! Externe Schnittstelle
interface gi0/1
 ip address 200.0.0.2 255.255.255.252
 ip nat outside
 no shutdown

! Rückrouten zu VLANs
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1

! NAT-Konfiguration (PAT)
ip access-list standard NAT_INSIDE
 permit 192.168.10.0 0.0.0.255
 permit 192.168.20.0 0.0.0.255

ip nat inside source list NAT_INSIDE interface gi0/1 overload

! Default Route ins Internet
ip route 0.0.0.0 0.0.0.0 200.0.0.1

## Verifikation

#  -----------Switch

show vlan brief

show ip interface brief

show ip route


# -------------Router

show ip interface brief

show ip route

show ip nat translations

show access-lists


# ----------PC-Tests

Ping eigenes Gateway (192.168.10.1 / 192.168.20.1)

Ping VLAN-übergreifend (z. B. PC-V1 → PC-W1)

Ping Router intern (10.0.0.2)

Ping Router extern (200.0.0.1, ISP)


## Häufige Fehler


-ip routing vergessen → keine Kommunikation zwischen VLANs

-Ports nicht zugewiesen → SVI bleibt down

-no shutdown vergessen → Interface bleibt down

-Keine R

-NAT fehlt/falsch → kein Internetzugang


