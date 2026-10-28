# 🔐 Home Network Hardening — UniFi Firewall, Segmentation & Secure Remote Access

Follow-up to my [home-network-rebuild](https://github.com/jchungit/home-network-rebuild) project. Once the new UniFi gateway and VLANs were in place, I worked through a set of hardening changes to shrink the attack surface, control DNS, and make remote access safe.

> 🔒 IP addresses, subnet ranges, VLAN IDs, domains, and device names have been intentionally left out.

---

## 🗺️ Network Segments

| Segment | Purpose | Notes |
|---|---|---|
| **Personal** | Phones, laptops, desktop | DNS forced through Pi-hole |
| **Homelab** | 24/7 server and lab VMs | Fixed addresses; services live here |
| **IoT** | TVs, smart-home devices | DNS forced through Pi-hole |
| **Guest** *(new)* | Visitors' devices | UniFi *Hotspot* zone — internet only, no access to internal networks |

---

## 🛠️ Hardening Changes

### 1. Guest Network Isolation
- Created a separate **Guest Wi-Fi** on its own VLAN using UniFi's **Hotspot zone type**, so visitor devices can reach the internet but nothing inside the house

### 2. Inbound Firewall Rules
- Added a gateway rule to **block inbound FTP (File Transfer Protocol, TCP 21)** from the internet and verified it was enforced
- Noted that the ISP gateway in front can't block individual ports in its UI — so the UniFi gateway (behind it, in passthrough mode) is the real enforcement point
- Only **80/443** are forwarded, and only to the reverse proxy — no admin panels, SSH, or dashboards are exposed

### 3. Region Blocking (Geo-IP)
- Enabled **UniFi CyberSecure Region Blocking** — *Block, Incoming only* — for countries that are common sources of scanning traffic
- Chose a **targeted blocklist** over "allow US only," so outbound browsing and roaming devices aren't broken while unsolicited inbound traffic is cut down
- UniFi firewall logs feed into my Wazuh SIEM for alerting (see [wazuh-soc-dashboard](https://github.com/jchungit/wazuh-soc-dashboard))

### 4. DNS Control
- **Pi-hole** is the *only* DHCP-assigned DNS server on the Personal and IoT networks, so ad/tracker/malware-domain filtering can't be skipped by picking a different resolver
- Upstream DNS is encrypted with **DNS-over-HTTPS (DoH)** via dnscrypt-proxy
- **Tailscale global DNS** points at the same Pi-hole with *Override DNS* on — so my phone and laptop are filtered even when I'm away from home

### 5. Secure Remote Access (Tailscale)
- The 24/7 server is the **Tailscale subnet router** (reach home networks remotely) and **exit node** (route all traffic home when on untrusted Wi-Fi)
- *Accept routes* is turned off on the server machines themselves to avoid routing conflicts
- Lesson learned: leaving the **exit node on while at home** made YouTube and the App Store slow on my phone — traffic was making an unnecessary trip through the tunnel. Exit node is now *off* at home, *on* for public Wi-Fi.

---

## 🧯 Troubleshooting Notes

| Symptom | Root Cause | Fix |
|---|---|---|
| Slow streaming / app downloads on phone at home | Exit node active on the home network | Exit node → None when at home |
| Ad-blocking briefly stopped on some devices | DHCP DNS set to Auto during testing | Restored Pi-hole as the sole DNS server |
| Couldn't block a port at the ISP gateway | ISP gateway UI doesn't support per-port rules | Enforce on the UniFi gateway instead |

---

## 📌 Planned
- [ ] Close 80/443 entirely and make media streaming **Tailscale-only** (after testing the TV on the IoT network)
- [ ] **IPv6** rollout as a learning exercise: DHCPv6 prefix delegation, confirm default-deny inbound IPv6, keep DNS pinned to Pi-hole, leave IoT on IPv4 only
- [ ] UniFi Protect cameras with local recording

---

## 🎓 Concepts Practiced
**Attack-surface reduction** · **Network segmentation** · **Default-deny inbound** · **Geo-IP filtering** · **DNS sinkholing** · **Encrypted DNS** · **Zero-trust style remote access (WireGuard via Tailscale)** · **Change verification**

## 🧠 Skills Demonstrated
`UniFi Network` · `Firewall rules` · `VLANs` · `Pi-hole` · `DNS-over-HTTPS` · `Tailscale` · `Geo-IP blocking` · `Network troubleshooting`
