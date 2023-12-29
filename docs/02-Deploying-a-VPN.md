# Deploying a VPN

## Wireguard

I followed [Wolfgang's Wireguard VPN Guide](https://notthebe.ee/blog/set-up-your-own-vpn-on-raspberry-pi/#template) to set up my VPN.

### Dynamic DNS

Before I installed Wireguard, I needed to make sure that I had some way to always get my public ip. This is because I do not have a static public ip, and I will need a Dynamic DNS provider to aid the tracking/changes. The provider I chose was [`freedns.afraid.org`](https://freedns.afraid.org/). Now, I moved onto doing things on the RPi.

#### `ddclient`

##### Installation of `ddclient`

``` bash
sudo apt install ddclient
```

##### Post-Installation of `ddclient`

Now, to configure `ddclient` to listen to `freedns.afraid.org`, I ran this command:

``` bash
sudo nano /etc/ddclient.conf
```

I deleted everything in the config file, and replaced it with this:

``` conf
daemon=5m
timeout=10
syslog=no # log update msgs to syslog
#mail=root # mail all msgs to root
#mail-failure=root # mail failed update msgs to root
pid=/var/run/ddclient.pid # record PID in file.
ssl=yes # use ssl-support. Works with ssl-library

# use=if, if=eth0
use=web, web=dynamicdns.park-your-domain.com/getip
server=freedns.afraid.org
protocol=freedns
login=FREEDNS LOGIN
password=FREEDNS PASSWORD
freedns.domain
```

Now I edited another config file, and set everything to `false` except `run_daemon`. In this case, `run_daemon` was not written in file but I added `run_daemon=true` at the bottom of the file.

``` bash
sudo nano /etc/default/ddclient
```

Now that everything has been configured, I restarted the `ddclient` service for the changes to take place:

``` bash
sudo systemctl restart ddclient
```

Just to checked that it worked, the ip-address that is on `freedns.afraid.org` should have changed from `0.0.0.0` to the actual public ip-address.

Finally, I enabled the service to run when the RPi boots:

``` bash
sudo systemctl enable ddclient
```

### Port-Forwarding

I opened my router's settings and port-forwarded the port that Wireguard will need. `51820` is Wireguard's default port.

``` router
Device: Raspberry Pi's hostname or IP
Protocol: UDP
Port range: 51820-51820
Outgoing port: 51820
Permit Internet access: yes
```

### Installation of Wireguard

I ran the following command in the terminal; the first part will download the bash script and the second part will run the bash script ([Github](https://github.com/Nyr/wireguard-install)):

``` bash
wget https://git.io/wireguard -O wireguard-install.sh && sudo bash wireguard-install.sh
```

During the installation process, it asked for my public IPV4 address, this is where I provided my `freedns.domain` that I created earlier.

### Post-Installation of Wireguard

The script prompted me to add my first user, and I added my phone. On my phone, I downloaded the official Wireguard app, and added a tunnel via the QR Code that was generated by the script.

To add my laptop into the client list, I ran the script again:

``` bash
sudo bash wireguard-install.sh
```

Since my laptop cannot scan a QR Code, I had to copy over the config file that was generated from the script while in root mode; because the files are stored in root:

``` bash
# Enter root mode
sudo su

cp /root/*.conf path/to/samba/share
```

[prev](./01-Deploying-a-Static-Website.md)