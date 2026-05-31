# Lab Project 04 — Network Configuration & Troubleshooting

**Date:** 2026-05-22
**Environment:** Ubuntu 26.04 LTS | VirtualBox | Host: Windows 10 Pro
**Objective:** Diagnose and troubleshoot a Linux network configuration issue
by simulating a broken gateway, restoring connectivity, and flushing DNS cache.

---

## Scenario

A user reports their Linux workstation has no internet access. As the helpdesk
tech you need to run a full network diagnostic, identify the fault, restore
the correct configuration, and flush the DNS cache.

---

## Steps Performed

### 1. Checked baseline network configuration
```bash
ip a
ip route show
cat /etc/resolv.conf
```
- `ip a` showed adapter enp0s3 with IP 10.0.2.15/24 — confirmed network
  adapter was recognized
- `ip route show` confirmed default gateway was 10.0.2.2
- `cat /etc/resolv.conf` confirmed DNS nameserver was configured
- Established a clean baseline before making any changes

### 2. Ran full diagnostic chain
```bash
ping -c 4 127.0.0.1
ping -c 4 10.0.2.2
ping -c 4 8.8.8.8
ping -c 4 google.com
```
- All four pings returned 0% packet loss confirming full connectivity
- Loopback confirmed network stack working
- Gateway ping confirmed local routing working
- 8.8.8.8 confirmed internet connectivity without DNS
- google.com confirmed DNS resolving correctly

### 3. Simulated network failure (the break)
```bash
sudo ip route del default
```
- Deleted the default gateway route
- Machine no longer knew where to send internet traffic
- Verified break with `ping -c 4 8.8.8.8` — returned 100% packet loss
- Internet confirmed broken

### 4. Diagnosed the problem
```bash
ip route show
```
- Confirmed default gateway route was missing entirely
- This is the exact diagnostic step a helpdesk tech would run to identify
  a routing problem on a user's machine

### 5. Restored correct gateway (the fix)
```bash
sudo ip route add default via 10.0.2.2
```
- Restored the correct default gateway
- Verified with `ip route show` — default via 10.0.2.2 confirmed

### 6. Verified connectivity restored
```bash
ping -c 4 8.8.8.8
ping -c 4 google.com
```
- Both returned 0% packet loss confirming full connectivity restored
- DNS also confirmed working after fix

### 7. Flushed DNS cache
```bash
sudo resolvectl flush-caches
resolvectl statistics
ping -c 4 google.com
```
- Cleared all stored DNS lookups
- Verified cache was cleared with statistics command
- Confirmed DNS still resolving correctly after flush

---

## Issues Encountered

**Problem:** Command `sudo ip route add default via 10.0.2.100` returned
"default is invalid" error when attempting to set a wrong gateway.
**Outcome:** Skipped this step since deleting the default route had already
successfully broken internet connectivity — achieving the same diagnostic goal.
**What this taught:** A missing gateway is just as common in the real world
as a wrong one. Both result in no internet access and require the same fix.

---

## What I Learned

- `ip a` is the Linux equivalent of ipconfig — always the first command
  to run when investigating network issues
- `ip route show` reveals the routing table — a missing or wrong default
  gateway is one of the most common causes of no internet access
- The diagnostic chain always goes in order — loopback → gateway →
  internet IP → domain name. This isolates exactly where the fault is
- If 8.8.8.8 works but google.com fails the problem is DNS not internet
- If 8.8.8.8 fails the problem is routing or connectivity not DNS
- Flushing DNS cache is a standard first step when websites won't load
  correctly even with a working connection
- Real world troubleshooting sometimes requires adapting when a command
  behaves unexpectedly — the goal is proving the fault exists, not
  following steps rigidly

---

## CompTIA A+ Relevance

- Domain 2.1 — Compare and contrast TCP and IP addressing
- Domain 2.5 — Summarize the properties and purpose of networked host services
- Domain 5.7 — Troubleshoot common wired and wireless network problems
