# Network Design

## 1. Project Overview

This project simulates a company with a headquarters (HQ) and two branch
offices (BR1 and BR2) using Cisco Packet Tracer.

The objective is to connect all sites over WAN links and use dynamic routing
so that every site can reach every other site without static routes.

## 2. Business Requirements

The company requires:

* A separate network for each site.
* Separate user and server networks at HQ.
* Connectivity between all sites.
* Dynamic routing that adapts automatically if a WAN link fails.
* A scalable IP plan that makes the location of an address easy to identify.
* Documented configuration and test results.

## 3. Network Topology

* HQ: router R-HQ, switch SW-HQ, PC-HQ1 and server SRV-HQ.
* Branch 1: router R-BR1, switch SW-BR1 and PC-BR1.
* Branch 2: router R-BR2, switch SW-BR2 and PC-BR2.
* Hub-and-spoke WAN: each branch connects to HQ over a serial link.

### Physical connections

| Device | Port    | Connected to | Port    | Type             |
|--------|---------|--------------|---------|------------------|
| R-HQ   | Gi0/0   | SW-HQ        | Gi0/1   | 802.1Q trunk     |
| R-HQ   | Se0/1/0 | R-BR1        | Se0/1/0 | Serial WAN (DCE) |
| R-HQ   | Se0/1/1 | R-BR2        | Se0/1/0 | Serial WAN (DCE) |
| SW-HQ  | Fa0/1   | PC-HQ1       | Fa0     | Access VLAN 10   |
| SW-HQ  | Fa0/2   | SRV-HQ       | Fa0     | Access VLAN 20   |
| R-BR1  | Gi0/0   | SW-BR1       | Gi0/1   | Access VLAN 10   |
| SW-BR1 | Fa0/1   | PC-BR1       | Fa0     | Access VLAN 10   |
| R-BR2  | Gi0/0   | SW-BR2       | Gi0/1   | Access VLAN 10   |
| SW-BR2 | Fa0/1   | PC-BR2       | Fa0     | Access VLAN 10   |

R-HQ is the DCE side of both serial links and provides the clock rate (= 64,000 bps).

See `diagrams/network-topology.png`.

## 4. IP Addressing Plan

The second octet identifies the site and the third octet matches the VLAN ID:

* 10.**10**.x.x = HQ
* 10.**20**.x.x = Branch 1
* 10.**30**.x.x = Branch 2
* 10.**0**.0.x = WAN links

### LAN networks

| Site | VLAN | Name    | Network        | Gateway    |
|------|------|---------|----------------|------------|
| HQ   | 10   | USERS   | 10.10.10.0/24  | 10.10.10.1 |
| HQ   | 20   | SERVERS | 10.10.20.0/24  | 10.10.20.1 |
| BR1  | 10   | USERS   | 10.20.10.0/24  | 10.20.10.1 |
| BR2  | 10   | USERS   | 10.30.10.0/24  | 10.30.10.1 |

The same VLAN ID is reused at every site for the same purpose. VLANs do not
extend across the routed WAN, so VLAN 10 at each site is a separate broadcast
domain with its own subnet.

### WAN networks

| Link     | Network     | R-HQ     | Branch   |
|----------|-------------|----------|----------|
| HQ - BR1 | 10.0.0.0/30 | 10.0.0.1 | 10.0.0.2 |
| HQ - BR2 | 10.0.0.4/30 | 10.0.0.5 | 10.0.0.6 |

A /30 provides exactly two usable addresses, one for each router.

### Hosts

| Device | IP Address     | Gateway    | Assignment |
|--------|----------------|------------|------------|
| PC-HQ1 | 10.10.10.10/24 | 10.10.10.1 | Static     |
| SRV-HQ | 10.10.20.10/24 | 10.10.20.1 | Static     |
| PC-BR1 | 10.20.10.10/24 | 10.20.10.1 | Static     |
| PC-BR2 | 10.30.10.10/24 | 10.30.10.1 | Static     |

Clients use static addresses in v1.0. Centralized DHCP is planned for v2.0.

## 5. Routing Design

All routers run OSPF process 1 in a single area (area 0).

| Router | Router ID | Networks advertised                                    |
|--------|-----------|--------------------------------------------------------|
| R-HQ   | 1.1.1.1   | 10.10.10.0/24, 10.10.20.0/24, 10.0.0.0/30, 10.0.0.4/30 |
| R-BR1  | 2.2.2.2   | 10.20.10.0/24, 10.0.0.0/30                             |
| R-BR2  | 3.3.3.3   | 10.30.10.0/24, 10.0.0.4/30                             |

* Router IDs are set manually so they are predictable and easy to identify.
* LAN interfaces are passive: their networks are advertised, but no OSPF
  hellos are sent toward end devices.

## 6. Security Considerations

v1.0 includes only basic device configuration (`enable secret`, login banner).
Full device hardening is planned for a later version.

## 7. Testing

All tests and results are documented in `docs/test-results.md`.

## 8. Project Status

Status: v1.0 implemented

Next step: centralized DHCP with DHCP relay, DNS, NTP and Syslog (v2.0).