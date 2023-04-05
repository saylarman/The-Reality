# The Reality

_Hello, my name is Arsham, I have been testing Reality Tls for a few weeks and I was satisfied with it, of course, I also did other things that I plan to teach you in this tutorial.
If an explanation is needed, I will write for each part, otherwise you can find the result yourself with a simple search._

_**I don't give you any guarantee that your server will not be filtered!**_

### -[ Install BBR ]
 _If you have high users enable this_
```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```
_I use this because it's good_
### -[ Change SSH port ]
_I recommend changing the port, you can change it if you want_

_If you don't have the netstat tool, you can install it_

_For check SSH port_
```bash
apt install net-tools
```
```bash
netstat -tulnp | grep ssh
```
_If you get the result, you can change the **PORT**_

_Now you can use this to change the port if you can't_

_Go to Google or Youtube and see tutorials for this part_

_Block the old **SSH** port_
```bash
ufw status numbered
```
_**ufw delete SSH-OLD-PORT**_
```bash
ufw delete
```
_Note that some providers do not allow changing the port!_

_Usually, by changing the SSH port, many problems such as speed drop, etc. are solved_
```bash
nano /etc/ssh/sshd_config
```
_Now restart **SSH** service_
```bash
systemctl restart sshd
```
_If you are using **UFW**, open a new **SSH** port_
```bash
ufw allow 45678/tcp
```
_The **45678** is a example_
### -[ Install Xray v1.8.0 ]
```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install --version 1.8.0 -u root
```
_Because I added **-u root**, there is no need to change the Xray configuration to change "user"._
### -[ install dlc.dat and iran.dat ]
_This section is for blocking Ads, Iran domians or IPs, Porn Websites and..._
```bash
wget -O /usr/local/share/xray/iran.dat https://github.com/bootmortis/iran-hosted-domains/releases/latest/download/iran.dat
wget -O /usr/local/share/xray/dlc.dat https://github.com/v2fly/domain-list-community/releases/latest/download/dlc.dat
```
_These files are constantly being updated, so delete the old version every few days and download the new version!_

_**If you want files to be downloaded and updated every 24 hours or..., do this**_

_We do this with Python, download the prerequisites_
```bash
apt update -y && apt upgrade -y && apt install python3 && apt install screen -y
```
_In root directory, create a **geo.py**_ 
```bash
nano geo.py
```
_Now add these codes and save_
```bash
import time
import os

p1 = os.path.exists(r'/usr/local/share/xray/iran.dat')
p2 = os.path.exists(r'/usr/local/share/xray/dlc.dat')

while True:
    if p1:
        os.remove("/usr/local/share/xray/iran.dat")
        os.system("wget -O /usr/local/share/xray/iran.dat https://github.com/bootmortis/iran-hosted-domains/releases/latest/download/iran.dat")

    if p2:
        os.remove("/usr/local/share/xray/dlc.dat")
        os.system("wget -O /usr/local/share/xray/dlc.dat https://github.com/v2fly/domain-list-community/releases/latest/download/dlc.dat")
    time.sleep(172800)
```
_**These times are in seconds**_

_86400 = 1 day_

_172800 = 2 day_

_259200 = 3 day_

_In order for the Python file to have access and be able to work 24 hours a day, use the following commands_
```bash
chmod +x geo.py && screen -c geo.py
```
### -[ For get Pub Key and Priv Key ]
_Save the **Pub Key** and **Priv Key** in a text file or save it on your server_
```bash
xray x25519 > key
```
_After that use 'ls' to check the directory_

_You can use **cat** for see **key**_
```bash
cat key
```
_If you want a find a domian for **serverNames** use this_

_**xray tls ping YOUR-DOMAIN**_
```bash
xray tls ping
```
_You can see the results like this_

Pinging with SNI
handshake succeeded
**Allowed domains: [debian.org, ftp.debian.org]**
Tls ping finished

_We need **Allowed domains** domains_

_**Note :**_

_1- Allowed domains may also have Star ( **Like this** > **\*.debian.org** ) which are not supported yet_

_2- Because Reality uses **Tls v1.3** and need **H2** protocol, be sure to check with your browser's Dev Option that the desired site uses Tls **1.3** and **H2**_

_For check **Tls version** protocol, Go to domian and open **Dev Tools** and go to **Security Tab**, and in **Connection** you can check the Protocol of **Tls**_

_For check **H2** protocol, Go to domian and open **Dev Tools** and go to **Network tab**, you can see a **Status** click right and enable **Protocol** if you can see **H2** it's true_
### -[ Change config.json of Xray ]
```bash
nano /usr/local/etc/xray/config.json
```
_First remove the **{}** in the config.json file, Now you can add the codes_

_**Only Vless Reality TCP, gRPC, H2**_
```bash
{
    "inbounds": [
        {
            "listen": "0.0.0.0",
            "port": 443,
            "protocol": "vless",
            "settings": {
                "clients": [
                {
                    "id": "EX", # Your UUID
                    "flow": "" # If you want use TCP, add xtls-rprx-vision else no need to change
                }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "EX", # You can change this to h2, grpc, tcp, I Recommend grpc
                "security": "reality",
                "realitySettings": {
                    "show": true,
                    "dest": "EX", # Example : ftp.debian.org:443 or etc...
                    "xver": 0, # I suggest that you use the same site for dest from the site you use for serverNames
                    "serverNames": [
                    "EX" # Example : ftp.debian.org
                    ],
                    "privateKey": "EX", # Your Private Key
                    "shortIds": [
                    "" # I don't use Short ID
                    ]
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                    ]
            }
        }
    ],
    "outbounds": [
    {
        "protocol": "freedom",
        "tag": "direct"
    }, {
        "protocol": "blackhole",
        "tag": "blocked"
    }
    ],
    "api": {
        "tag": "api",
        "services": [
            "HandlerService",
            "LoggerService",
            "StatsService"
            ]
    },
    "policy": {
        "levels": {
            "0": {
                "statsUserUplink": true,
                "statsUserDownlink": true
            }
        },
        "system": {
            "statsInboundUplink": true,
            "statsInboundDownlink": true,
            "statsOutboundUplink": true,
            "statsOutboundDownlink": true
        }
    },
    "log": {
        "loglevel": "info",
        "access": "/var/log/xray/access.log",
        "error": "/var/log/xray/error.log"
    },
    "dns": {
        "servers": [
            "https+local://cloudflare-dns.com/dns-query",
            "1.0.0.1",
            "1.1.1.1",
            "8.8.8.8",
            "8.8.4.4",
            "localhost"
            ]
    },
    "routing": {
        "domainStrategy": "IPIfNonMatch",
        "rules": [
        {
            "inboundTag": [
                "api"
                ],
            "outboundTag": "api",
            "type": "field"
        }, {
            "domain": [
                "instagram.com",
                "www.instagram.com",
                "mediafire.com",
                "www.mediafire.com"
                ],
            "outboundTag": "proxy",
            "type": "field"
        }, {
            "domain": [
                "regexp:.*\\.ir$",
                "ext:iran.dat:ir",
                "ext:iran.dat:ads",
                "ext:iran.dat:other",
                "geosite:category-ir",
                "geosite:category-ir-gov",
                "geosite:category-ir-news",
                "geosite:category-ir-bank",
                "geosite:category-ir-tech",
                "geosite:category-ir-travel",
                "geosite:category-ir-scholar",
                "geosite:category-ir-shopping",
                "geosite:category-ir-insurance"
                ],
            "type": "field",
            "outboundTag": "blocked"
        }, {
            "type": "field",
            "ip": [
                "geoip:ir"
                ],
            "outboundTag": "blocked"
        }, {
            "protocol": [
                "bittorrent"
                ],
            "type": "field",
            "outboundTag": "blocked"
        }
        ]
    }
}
```
_**If no need to Proxy block, you can remove**_
```bash
           {
            "domain": [
                "instagram.com",
                "www.instagram.com",
                "mediafire.com",
                "www.mediafire.com"
                ],
            "outboundTag": "proxy",
            "type": "field"
        },
```
_**Only Trojan Reality gRPC, H2**_
```bash
{
    "inbounds": [
      {
          "port": 443,
          "protocol": "trojan",
          "settings": {
              "clients": [
              {
                  "password": "EX" # Your UUID
              }
              ]
          },
          "streamSettings": {
              "network": "EX", # You can change this to h2 or grpc
              "security": "reality",
              "realitySettings": {
                  "show": true,
                  "dest": "EX", # Example : ftp.debian.org:443 or etc...
                  "xver": 0, # I suggest that you use the same site for dest from the site you use for serverNames
                  "serverNames": [
                  "EX" # Example : ftp.debian.org
                  ],
                  "privateKey": "EX", # Your Private Key
                  "shortIds": [
                  "" # I don't use Short ID
                  ]
              }
          },
          "sniffing": {
              "enabled": true,
              "destOverride": [
                  "http",
                  "tls"
                  ]
          }
      }
    ],
    "outbounds": [
    {
        "protocol": "freedom",
        "tag": "direct"
    }, {
        "protocol": "blackhole",
        "tag": "blocked"
    }
    ],
    "api": {
        "tag": "api",
        "services": [
            "HandlerService",
            "LoggerService",
            "StatsService"
            ]
    },
    "policy": {
        "levels": {
            "0": {
                "statsUserUplink": true,
                "statsUserDownlink": true
            }
        },
        "system": {
            "statsInboundUplink": true,
            "statsInboundDownlink": true,
            "statsOutboundUplink": true,
            "statsOutboundDownlink": true
        }
    },
    "log": {
        "loglevel": "info",
        "access": "/var/log/xray/access.log",
        "error": "/var/log/xray/error.log"
    },
    "dns": {
        "servers": [
            "https+local://cloudflare-dns.com/dns-query",
            "1.0.0.1",
            "1.1.1.1",
            "8.8.8.8",
            "8.8.4.4",
            "localhost"
            ]
    },
    "routing": {
        "domainStrategy": "IPIfNonMatch",
        "rules": [
        {
            "inboundTag": [
                "api"
                ],
            "outboundTag": "api",
            "type": "field"
        }, {
            "domain": [
                "instagram.com",
                "www.instagram.com",
                "mediafire.com",
                "www.mediafire.com"
                ],
            "outboundTag": "proxy",
            "type": "field"
        }, {
            "domain": [
                "regexp:.*\\.ir$",
                "ext:iran.dat:ir",
                "ext:iran.dat:ads",
                "ext:iran.dat:other",
                "geosite:category-ir",
                "geosite:category-ir-gov",
                "geosite:category-ir-news",
                "geosite:category-ir-bank",
                "geosite:category-ir-tech",
                "geosite:category-ir-travel",
                "geosite:category-ir-scholar",
                "geosite:category-ir-shopping",
                "geosite:category-ir-insurance"
                ],
            "type": "field",
            "outboundTag": "blocked"
        }, {
            "type": "field",
            "ip": [
                "geoip:ir"
                ],
            "outboundTag": "blocked"
        }, {
            "protocol": [
                "bittorrent"
                ],
            "type": "field",
            "outboundTag": "blocked"
        }
        ]
    }
}
```
_**Note Trojan Reality supported only Andtoid**_

_**Because other people have explained it, I will not explain it very completely ( You can refer to the end of the page for links )**_

[_**You can also see more Configuration, click here**_](https://github.com/lxhao61/integrated-examples)

_**If you want to block Speed test websites or Whoer websites, you can add these codes to Rules**_
```bash
                "speed.cloudflare.com",
                "nordvpn.com",
                "testmyspeed.com",
                "speedcheck.org",
                "gocompare.com",
                "netspotapp.com",
                "virginmedia.com",
                "broadbandspeedtest.org.uk",
                "thinkbroadband.com",
                "broadbandspeedchecker.co.uk",
                "measurementlab.net",
                "mxtoolbox.com",
                "tunnelbear.com",
                "top10vpn.com",
                "ip.me",
                "which.co.uk",
                "whatismyip.net",
                "ipcost.com",
                "myip.com",
                "whatsmyip.com",
                "dnsleak.com",
                "whatsmyip.org",
                "iplocation.net",
                "whatismyip.com",
                "whoer.net",
                "whatismypublicip.com",
                "ipaddress.my",
                "showmyip.com",
                "www.expressvpn.com",
                "perfect-privacy.com",
                "surfshark.com",
                "browserleaks.com",
                "dnsleaktest.org",
                "www.dnsleaktest.com",
                "whatismyipaddress.com",
                "fast.com",
                "speedtest.net",
```
_**I have written the most used ones, it is not very complete**_

_If want block **Porns** and this_
```bash
                "geosite:category-porn",
```
_If want block **Ads** and this_
```bash
                "geosite:category-ads-all",
                "geosite:category-ads",
                "geosite:google-ads"
```
_It does not block all advertisements, it is possible that a series of programs may encounter problems, to solve this problem, define those programs in the Proxy section._

_Now restart Xray and check status for this working **normally** or **not**_
```bash
systemctl restart xray.service
systemctl status xray.service
```
### -[ Install Nginx ]
_**To be able to bring up a simple page**_

_Please allow **HTTP** port in ufw before install_
```bash
ufw allow http
```
_Now can install_
```bash
apt install nginx -y
```
_**If you want to get a certificate for your page, use these codes**_

_Please allow **HTTPS** port in ufw before install_
```bash
ufw allow https
```
_Now can install_
```bash
apt install nginx -y
apt install certbot python3-certbot-nginx -y
certbot --nginx -d
```
_certbot --nginx -d YOUR-DOMAIN_

_Now go to this Directory_
```bash
cd /var/www/html/
```
_Remove the **index.html** file_
```bash
rm index.html
```
_Create a **index.html** file_
```bash
nano index.html
```
_Now add these codes_

```bash
<html>
    <head>
        <link rel="icon" type="image/x-icon" href="http://YOUR-IP-OR-DOMAIN/kozu.gif">
        <meta name="viewport" content="user-scalable=no">
        <meta charset="UTF-8">
        <title>=)</title>
        <style>
body {
        margin-top: 60px;
       -webkit-user-select: none;
       -ms-user-select: none;
        user-select: none;
        overflow-x: hidden;
        background: #FDFAFA;
        font-weight: 350;
        font-family: -apple-system, BlinkMacSystemFont, opensans, Optima, 'Microsoft Yahei', sans-serif;
        line-height: 0;
}

.kozu {
        padding: 40px 0;
}
.kozu p {
        color: #000;
        font-style: italic;
        text-decoration:none;
        letter-spacing: 1px;
        cursor: default;
        text-align:center;
        font-size:32px;
}
img {
        pointer-events: none;
        margin: auto;
        display: block;
}
.isBold { font-weight: bold;
}
</style>
    </head>
    <body><img loading="lazy" src="http://YOUR-IP-OR-DOMAIN/kozu.gif" alt="Kozu" class="center">
        <div class="kozu">
            <p>This IP belongs to <span class="isBold">ARSHAM.6IX</span></p>
        </div>
    </body>
</html>
```
_Now go to your **Nginx configuration** for **Redirect** all to **index.html**_
```bash
nano /etc/nginx/sites-enabled/default
```
_You can see **try_files $uri $uri/ =404;**, change **=404;** to **/index.html;** of **HTTPS**_
```bash
/index.html;
```
_If you use certificate you need to change **=404;** to **/index.html;** of **HTTPS** and **HTTP**_

_Now restart **Nginx**_
```bash
systemctl restart nginx
```
### -[ Other ]
_If your users cannot access some sites ( **if you use IPv6** ), enter this command._
```bash
sysctl -w net.ipv6.conf.all.disable_ipv6=1 sysctl -w net.ipv6.conf.default.disable_ipv6=1 sysctl -w net.ipv6.conf.lo.disable_ipv6=1
```
_Restart Nginx_
```bash
systemctl restart nginx
```
### -[ Recommendations ]
_I recommend reading or viewing these pages_
  - [_**How to find the site for REALITY ( Persian )**_](https://telegra.ph/%D9%86%D8%AD%D9%88%D9%87-%D9%BE%DB%8C%D8%AF%D8%A7-%DA%A9%D8%B1%D8%AF%D9%86-%D8%B3%D8%A7%DB%8C%D8%AA-%D8%A8%D8%B1%D8%A7%DB%8C-REALITY-03-11)
  - [_**Teaching the use of REALITY ( Persian )**_](https://telegra.ph/%D8%A2%D9%85%D9%88%D8%B2%D8%B4-%D8%A7%D8%B3%D8%AA%D9%81%D8%A7%D8%AF%D9%87-%D8%A7%D8%B2-REALITY-03-10)
  - [_**Project X Documents Of Configs**_](https://xtls.github.io/config/)
  - [_**Domain List Community**_](https://github.com/v2ray/domain-list-community)
  - [_**Official Xray Core Page**_](https://github.com/XTLS/Xray-core)
  - [_**Iran Hosted Domains**_](https://github.com/bootmortis/iran-hosted-domains)
  - [_**Integrated Examples**_](https://github.com/lxhao61/integrated-examples/)
  - [_**REALITY In English**_](https://cscot.pages.dev/2023/03/02/Xray-REALITY-tutorial)
  - [_**Chika0801**_](https://github.com/chika0801/Xray-examples)
  - [_**Project V**_](https://www.v2ray.com/en/)
  - [_**Dev分享**_](https://idev.dev/)
  - [_**Mr.xiao**_](https://www.losem.tk/)

_Thanks to the friends for helped me in the web field_

_Written by Arsham.6ix._
