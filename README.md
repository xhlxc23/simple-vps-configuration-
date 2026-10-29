# introduction
Using a domain name to enable external network access via a VPS.

_v.domin is not a true domin name, please replace it any time you see the word in the article_
# tools and website
- [vultr](https://www.vultr-cn.com/)
- [v2rayN](https://www.v2rayg.com/zh-CN/download.html)
# preparation
1. Launch a server on Vultr(Debian 12,Los Angeles,$5). Here your will get a new IP address;
2. Go to your domin name provider(Cloudflare) and ponit the A record for your `v.domin` to the new IP address; make sure your choice is DNS only;
# main operations
## step one
1. On your local computer's cmd or powershell, use _ssh_ to conect to the server so that you can regulate it. Paste the following code (which is only responsible for installing dependencies, enabling BBR, opening basic ports);
  ```bash
cat > /root/setup.sh << 'EOF'
#!/bin/bash
apt-get update -y >/dev/null 2>&1
apt-get install -y curl socat ufw lsof >/dev/null 2>&1

# BBR
ufw allow 22,80,443/tcp >/dev/null 2>&1
ufw --force enable >/dev/null 2>&1
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p >/dev/null 2>&1

# 3x-ui
echo -e "1\nn\nn\n4\n" | bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh) >/dev/null 2>&1
EOF
  ```
  ```bash
chmod +x /root/setup.sh && bash /root/setup.sh
  ```
## step two
_reset the user and disable 2FA_

1. In SSH session,type `x-ui`;
2. Enter _7 Reset..._
3. When prompted for confirmation, enter `y`.
4. Set the username: type `admin` and press Enter.
5. Set the password: type `admin123` and press Enter.
6. You will be asked whether to disable two-factor authentication: enter `y` and press Enter (this step turns off the annoying 2FA).
7. You will be asked whether to restart the panel: just press Enter (the default option is `y`).

_After the restart completes, the terminal will display your panel’s port. You can view the full login URL by entering `x-ui settings`. Also, you can pass it for the step 3 has the operation with the same effect._
## step three
_Use the x-ui menu to request a certificate._

Press Enter again to enter the x-ui menu:
1. Enter 20 (SSL Certificate Management).
2. Enter 1 (Get SSL - Domain).
3. Enter your domain: `v.domin`, then press Enter.
4. When prompted for a port, just press Enter (it will use port 80 by default for verification).
5. Wait a few seconds; you’ll see “Cert success” in green on the screen. It will then ask if you want to modify the reload command—simply enter 0 and press Enter to exit.
   (At this point, the certificate has been successfully generated in the default directory, and the panel has automatically recognized it.)
6. Simply paste the entire code snippet below into your SSH terminal and press Enter; it will display the control panel URL along with the public and private key addresses.
```bash
IP=$(curl -s ifconfig.me)
PORT=$(x-ui settings | grep '^port:' | awk '{print $2}')
PATH_URL=$(x-ui settings | grep '^webBasePath:' | awk '{print $2}')

mkdir -p /root/cert/v.domin
cp /root/.acme.sh/v.domin_ecc/fullchain.cer /root/cert/v.domin/fullchain.pem 2>/dev/null
cp /root/.acme.sh/v.domin_ecc/v.domin.xyz.key /root/cert/v.domin/privkey.pem 2>/dev/null

echo -e "\n======================================================="
echo "control panel: http://$IP:$PORT$PATH_URL"
echo "-------------------------------------------------------"
echo "public key: /root/cert/v.xzhfirst.xyz/fullchain.pem"
echo "private key: /root/cert/v.xzhfirst.xyz/privkey.pem"
echo "======================================================="
```
## step four
_Create a node on the panel_

1. Open your panel’s address in a browser (http://IP:port/path) and log in with admin / admin123.
2. Click “Inbound List” on the left -> click “Add Inbound.”
3. Fill in the following:
   · Remark: Enter anything (e.g., node)
   · Protocol: vless
   · Port: 443
   · Transport: tcp
   · Security: tls
4. In the TLS settings:
   · SNI: Enter v.domin
   · Certificate: public and private key address
5. Click “Create” in the bottom right corner.
## step five
_Add a client and export_

1. In the list, find the node you just created and click the “0” in the “Clients” column.
2. Click “Add Client,” enter any email address (e.g., 1), and click OK (the number will change to 1).
3. Click the “Export Link” or QR code icon on the right side of the node.
4. Copy the link or scan the QR code, import it into your v2rayN or mobile client, and enable the proxy to enjoy unrestricted access.
