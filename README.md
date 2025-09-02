# Self-Hosted Network Stack (Pi-hole + Unbound + WireGuard + Portainer + FreeDNS)

A self-hosted homelab stack with **Pi-hole**, **Unbound**, **WireGuard VPN**, **Portainer**, and **FreeDNS (ddclient)** — all managed via Docker Compose.

This setup provides:
- **Portainer** — Web-based Docker management UI  
- **Unbound** — Recursive DNS resolver for privacy  
- **Pi-hole** — Network-wide ad-blocking DNS server  
- **WireGuard (wg-easy)** — Secure VPN with easy client management  
- **FreeDNS (ddclient)** — Dynamic DNS updater for external access  

The stack is designed to run on a single server (e.g., Raspberry Pi, VPS) with an isolated Docker network and `.env` support for sensitive data.

---

## 🚀 Setup Instructions

### 1.0 Install prerequisites
Make sure **Docker** and **Docker Compose** are installed properly on your server.  
- [Docker installation guide](https://docs.docker.com/engine/install/)  
- [Docker Compose plugin installation](https://docs.docker.com/compose/install/linux/)  

---

### 1.1 Fix IP address for the server on which the stack is running

---

### 1.2 Set up FreeDNS
- Go to [afraid.org FreeDNS](https://freedns.afraid.org/) and create your **Dynamic DNS hostname**.  
- Configure it to point to your **public IP address**.  
- Allow it on your **router** (FritzBox has DNS Rebind Protection. The host must be added as exception).  

---

### 1.3 Open router ports
Log into your router and **forward the necessary ports** to your server:
Following ports must be added for the local IP of the server on which the stack is running.
- `51820/UDP` → WireGuard VPN
- `80/TCP` → Pi-hole web UI (if remote access needed, otherwise optional)  
- `9000/TCP` → Portainer web UI (if remote access needed, otherwise optional)  

---

### 1.4 Modify the `.env` file
Before starting the stack, you must edit the `.env_to_be_modified` file in the repository to set your passwords, IP addresses, and other sensitive data. Then it must renamed as `.env`   

---

### 2. Start the stack
Clone this repository, then run:
``` bash
git clone https://github.com/andreaperin/dns-vpn-stack.git
cd dns-vpn-stack
docker compose up -d
```

---

## 3. Access the services

After the stack is running, you can log in to the different services:

- **Portainer** → `http://<LOCAL_IP>:9000`  
  - The first time you log in, you must **create a username and password**, which will then be stored securely for future access.

- **WireGuard (wg-easy)** → `http://<LOCAL_IP>:51821`  
  - Username and password are set in the `.env` file (`WG_PASSWORD`).

- **Pi-hole** → `http://<LOCAL_IP>:8080/admin`  
  - Login user is `admin` and the password is set in the `.env` file (`PIHOLE_WEBPASSWORD`).

> 🔎 Replace `<LOCAL_IP>` with the IP addresses configured in your `.env` file or Docker Compose network.

---

## 4. Configure Pi-hole with Unbound

After everything is set up, configure Pi-hole to use Unbound as its upstream DNS resolver:

### Access Pi-hole's Admin Interface
Navigate to `http://<LOCAL_IP>:8080/admin` and log in using the credentials set in the `.env` file.

### Disable Default Upstream DNS Servers
- Go to **Settings** > **DNS**.  
- Under **Upstream DNS Servers**, uncheck all default options (e.g., Google, OpenDNS).

### Set Custom DNS Server
- Enable **Custom 1 (IPv4)**.  
- Enter `10.2.0.200#53` as the address.  

### Enable Respond only on interface eth0
- Go to **Settings** > **DNS**.
- Under **Interface settings** enable **Respond only on interface eth0**

### Add new Blocklist:

1. **Access the Adlists Configuration**:  
   - Click on **Group Management** in the left-hand menu.  
   - Select **Lists**.

2. **Add a New Blocklist**:  
   - In the **Address** field, paste the URL of the desired blocklist from [Firebog](https://firebog.net/) or another trusted source.  
   - Optionally, provide a **Name** for the list.  
   - Click **Add** to save.

4. **Update Gravity**:  
   - Navigate to **Tools** > **Update Gravity**.  
   - Click **Update** to refresh Pi-hole's blocklist database with the new entries.

> 🔄 After updating Gravity, Pi-hole will process the new blocklist, and you'll see an increase in the number of blocked domains.

> ⚠️ Note: Adding too many blocklists can lead to performance issues or over-blocking. It's advisable to start with a few trusted lists and monitor the impact.

---

### Enable Local DNS server on router:

- FritzBox works as a local DNS server so unbound will not work unless you enable the local DNS server
- Go to router page: **Network** > **Network Settings** > **IPv4 Settings** > Enable local DNS with `<LOCAL_IP>` of the machine you are running the stack in.

