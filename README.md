# hostapd_AP
A concise guide to setting up Access Point (AP) on Raspberry Pi 5 and other Linux systems using hostapd.

**1. Install Required Packages**

sudo apt update

sudo apt install -y hostapd dnsmasq

sudo systemctl unmask hostapd

sudo systemctl enable hostapd dnsmasq

**2. Prevent NetworkManager Interference**

sudo mkdir -p /etc/NetworkManager/conf.d

sudo tee /etc/NetworkManager/conf.d/unmanaged-wlan0.conf >/dev/null <<'EOF'
[keyfile]
unmanaged-devices=interface-name:wlan0
EOF

sudo systemctl restart NetworkManager_

**3. Assign Static IP for wlan0**  --> currently assigned with 10.42.1.1_

_sudo tee /etc/systemd/system/wlan0-static.service >/dev/null <<'EOF'
[Unit]
Description=Set static IP on wlan0 for AP
After=network.target
Wants=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ip link set wlan0 up
ExecStart=/usr/sbin/ip addr flush dev wlan0
ExecStart=/usr/sbin/ip addr add **10.42.1.1/24** dev wlan0
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload

sudo systemctl enable --now wlan0-static.service

**4. Configure hostapd** --->  a format file attached for a 5Ghz AP

**5. Configure dnsmasq for DHCP**

sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak

sudo tee /etc/dnsmasq.d/ap.conf >/dev/null <<'EOF'
interface=wlan0
bind-interfaces
dhcp-range=10.42.1.100,10.42.1.200,255.255.255.0,12h
dhcp-option=option:router,10.42.1.1
dhcp-option=option:dns-server,10.42.1.1
domain=ap.lan
EOF

sudo systemctl enable dnsmasq

6. **Restart all services** _--> maintain the order_

sudo systemctl restart wlan0-static.service

sudo systemctl restart dnsmasq

sudo systemctl restart hostapd



