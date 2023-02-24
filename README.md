# Homelab-services
A compilation of some ubuntu apps for my personal server

## 1. DNS Server
### 1.1. ADGuard home
Since the port 53 is used by ubuntu, it has to be removed before setting up Adguard:
```
sudo mkdir -p /etc/systemd/resolved.conf.d
nano /etc/systemd/resolved.conf.d/adguardhome.conf
```
Insert the next text:
```
[Resolve]
DNS=127.0.0.1
DNSStubListener=no
```
Activate and restart another resolv.conf file:
```
sudo mv /etc/resolv.conf /etc/resolv.conf.backup
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
sudo systemctl reload-or-restart systemd-resolved
```
Install and open the ports 80 and 3000 (if is a VPS).
```
curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh -s -- -v
ufw allow 80,3000/tcp
```
### 1.2. Pihole
Tried on debian 11
```
curl -sSL https://install.pi-hole.net | bash
```
To solve the error: `FTL failed to start due to failed to create listening socket for port 53: Address already in use`
```
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
curl -sSL https://install.pi-hole.net | bash
```
In settings, run restart DNS
## 2. OpenVPN 
Install the server
```
apt install ca-certificates wget net-tools gnupg
wget -qO - https://as-repository.openvpn.net/as-repo-public.gpg | apt-key add -
echo "deb http://as-repository.openvpn.net/as/debian focal main">/etc/apt/sources.list.d/openvpn-as-repo.list
apt update
apt install openvpn-as -y
```
Log into your server: `https://localhost:943/admin`, then set a password for the admin `openvpn`:
```
passwd openvpn
```
## 3. PiVPN
```
curl -L https://install.pivpn.io | bash
```
## 4. Nextcloud
Easily install Nextcloud using snap:
```
sudo snap install nextcloud
```
Edit your custom ports:
```
sudo snap set nextcloud ports.http=81 ports.https=444
```
After creating an admin account and installing nextcloud, If you are using a proxy, add `'overwriteprotocol' => 'https',` to: `/var/snap/nextcloud/current/nextcloud/config/config.php`
![Screenshot](https://user-images.githubusercontent.com/74340724/220689797-41100e6f-81e6-481f-8cae-428352996a03.png)

## 4. Jellyfin
### a. Install requirements:
```
sudo rm /etc/apt/sources.list.d/jellyfin.list
sudo apt install curl gnupg
sudo add-apt-repository universe
```
### b. Add the Jellyfin repository:
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/jellyfin.gpg
```
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/jellyfin.sources
Types: deb
URIs: https://repo.jellyfin.org/$( awk -F'=' '/^ID=/{ print $NF }' /etc/os-release )
Suites: $( awk -F'=' '/^VERSION_CODENAME=/{ print $NF }' /etc/os-release )
Components: main
Architectures: $( dpkg --print-architecture )
Signed-By: /etc/apt/keyrings/jellyfin.gpg
EOF
```
### c. Install Jellyfin:
```
sudo apt update
sudo apt install jellyfin
```
### d. Add storage permisions:
Supposing that your media files are in `/media/ubuntu/external/`:
```
sudo chown -R jellyfin /media/ubuntu/external
```
### e. Install plugins:
After the first setup, in Dashboard/Plugins install:
* Fanart
* OpenSubtitles
* Reports
After that, restart Jellyfin:
```
sudo /etc/init.d/jellyfin restart
```
