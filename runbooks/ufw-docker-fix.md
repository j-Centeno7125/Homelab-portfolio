# Runbook: Resolving Docker-to-Host Firewall Conflicts

## 📝 Issue Description
Uptime Kuma (running in a Docker container) was reporting the Jellyfin media server as "Down," despite the server being accessible from a web browser on the local network.

## 🔍 Troubleshooting Steps
1. Verified Jellyfin was running using `docker ps`.
2. Used `curl -I http://127.0.0.1:8096` on the host to confirm Jellyfin was responding locally.
3. Identified that the request was timing out between the Docker container and the host.
4. Suspected UFW was dropping packets originating from the Docker bridge network (172.16.0.0/12).

## 🛠️ Resolution
UFW was actively blocking traffic from the internal Docker network to the host's port 8096. I implemented a specific firewall rule to allow traffic from the Docker subnet to the specific port.

**Command used:**
`sudo ufw allow from 172.16.0.0/12 to any port 8096 proto tcp`

## 💡 Lessons Learned
Learned how Docker manipulates `iptables` and how UFW interacts with Docker bridge networks. Validated the importance of the Principle of Least Privilege by only opening the specific port required, rather than disabling the firewall entirely.
