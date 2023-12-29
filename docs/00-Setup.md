# Setup

[Official Documentation on the Raspberry Pi 5](https://www.raspberrypi.com/documentation/computers/raspberry-pi-5.html)

## Installing an Operating System

I used `Raspberry Pi OS Lite (64-bit)`, and the [`Raspberry Pi Imager`](https://www.raspberrypi.com/software/) to install it onto the SD Card.

Next, I continued to apply "OS customisation settings". Here, we can preconfigure certain settings to let the RPi run headless immediately. The settings that I configured are:

1. WiFi Credentials / Wireless LAN -- my local network
2. Device `hostname` -- `rpi5`
3. `username` and `password` for the admin account -- encrypt the password with `openssl passwd -6` if using `userconf.txt`
4. Locale settings -- `Asia/Singapore` & keyboard layout: `us`
5. Enable SSH -- most important setting for a headless config. temporarily allow password authentication, and slowly change to public-key authentication incase anything screws up

Moving on, I started writing (via `Raspberry Pi Imager`) these configurations and install the OS onto the SD card.

Finally, I plugged the SD card into the RPi, booted it up, and it worked immediately as a headless unit.

Once I ssh'ed into the RPi and booted into the terminal, I ran `sudo apt-get update && sudo apt-get upgrade -y` and waited for it to finish doing its thing.

[next](./01-Deploying-a-Static-Website.md)
