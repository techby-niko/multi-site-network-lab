# Network Test Results

## v1.0

### Interfaces
- All WAN interfaces are up/up: ✅
- R-HQ is DCE on both serial links: ✅
- All LAN interfaces and subinterfaces are up/up: ✅

### Local connectivity
- PC-HQ1 can reach its default gateway: ✅
- SRV-HQ can reach its default gateway: ✅
- PC-BR1 can reach its default gateway: ✅
- PC-BR2 can reach its default gateway: ✅
- R-HQ can reach R-BR1 over the WAN link: ✅
- R-HQ can reach R-BR2 over the WAN link: ✅

### OSPF
- R-HQ has two OSPF neighbors in FULL state: ✅
- R-BR1 has one OSPF neighbor (R-HQ) in FULL state: ✅
- R-BR2 has one OSPF neighbor (R-HQ) in FULL state: ✅
- All routers have OSPF routes to all remote networks: ✅
- LAN interfaces are passive: ✅

### End-to-end connectivity
- PC-BR1 can reach SRV-HQ: ❌ (see ## Notes)
- PC-BR2 can reach SRV-HQ: ✅
- PC-BR1 can reach PC-BR2: ✅
- PC-HQ1 can reach PC-BR1 and PC-BR2: ✅
- Traffic from BR1 to BR2 is routed through R-HQ (tracert): ✅

### Failover
- Shutting down Se0/1/1 on R-HQ removes R-BR2 as neighbor and
  its routes from the other routers: ✅
- Bringing Se0/1/1 back up restores the neighbor and routes: ✅

## Notes

### Server network not reachable from branches
**Problem:** PC-BR1 could not reach SRV-HQ. Tracert stopped at R-BR1.

**Cause:** R-BR1 had no route to 10.10.20.0/24. The OSPF configuration on R-HQ
contained `network 10.20.10.0 0.0.0.255 area 0` instead of `10.10.20.0`, so the
server network was never advertised.

**Fix:** Removed the incorrect network statement and added
`network 10.10.20.0 0.0.0.255 area 0` on R-HQ.

**Result:** Retested, PC-BR1 and PC-BR2 can reach SRV-HQ.