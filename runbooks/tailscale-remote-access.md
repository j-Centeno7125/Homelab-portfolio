# Runbook: Troubleshooting Remote Access Timeout for Jellyfin via Tailscale

## 📝 Issue Description
Attempting to access the Jellyfin media server remotely via its Tailscale IP address (`100.x.x.x:8096`) resulted in a browser error: `ERR_CONNECTION_TIMED_OUT`. 

## 🔍 Troubleshooting Steps
1. **Verified Local Access:** Confirmed Jellyfin was running and accessible on the local network (`http://192.168.1.199:8096`).
2. **Checked Port Listening:** Used `ss -tulpn | grep 8096` to confirm Jellyfin was listening on all interfaces (`0.0.0.0:8096`).
3. **Tested the Firewall:** Suspected UFW was blocking traffic from the Tailscale interface or Docker network. 
4. **Identified the Root Causes:** 
   - UFW was dropping packets from the internal Docker bridge network when Uptime Kuma tried to monitor the service.
   - The browser was failing to establish a secure connection over the raw Tailscale IP, causing a TLS error (`ERR_SSL_PROTOCOL_ERROR`) and later a DNS error (`DNS_PROBE_FINISHED_NXDOMAIN`) because MagicDNS was not fully configured on the client device.

## 🛠️ Resolution
1. **Firewall Fix:** Added a specific UFW rule to allow traffic from the Docker subnet to port 8096: `sudo ufw allow from 172.16.0.0/12 to any port 8096 proto tcp`.
2. **TLS/Proxy Fix:** Instead of exposing ports directly, used the Tailscale CLI to provision a valid HTTPS certificate and act as a reverse proxy: `sudo tailscale serve --bg http://127.0.0.1:8096`.
3. **MagicDNS Fix:** Enabled MagicDNS in the Tailscale Admin Console and ensured the client device was using Tailscale DNS, allowing access via the secure hostname: `https://littlegurtplanet.tail2a47b9.ts.net`.

## 💡 Lessons Learned
- Learned how to use `ss` to verify port bindings.
- Deepened understanding of how UFW interacts with Docker's iptables rules.
- Gained practical experience with TLS certificates and using Tailscale Serve to securely expose local services without opening router ports.
