# Deploying a Static Website

## Docker

Instead of installing `nginx` directly onto the bare metal. I will be using Docker containers to contain it.

Before I can run any container, I have to install Docker.

Since, Raspberry Pi OS is Debian (Bookworm 12), I followed [Docker's official documentation](https://docs.docker.com/engine/install/debian/) and I ran the following commands:

### Uninstall Old Versions

``` bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### Install Docker Using the `apt` Repository

``` bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  # `change $(. /etc/os-release && echo "$VERSION_CODENAME")` to `bookworm` for derivative distros
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

``` bash
# Install the latest version available
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify that the installation is successful by running the `hello-world` image
sudo docker run hello-world
```

### Post-Installation

Now, all Docker commands require root-level privileges, so this means that I will have to type `sudo` every single time.
This is very tiresome, and I know who can access this device so there is nothing much to worry about if I just gave the default user permissions.
I ran the following commands:

``` bash
# Create the `docker` group
sudo groupadd docker

# Add the current user to the `docker` group
sudo usermod -aG docker $USER

sudo reboot
```

After adding the `docker` group to `$USER`, I will need to reboot the machine for the group membership to be re-evaluated.

After this, I logged back in and ran the following command to verify that I could run Docker commands without `sudo`:

``` bash
docker run hello-world
```

Finally, I want my containers to spin up after every boot/reboot of the RPi. This means that the Docker daemon service has to start on boot with `systemd`.
To do this, I ran the following commands:

``` bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

## nginx

### Deploying the Docker container

For all of my Docker containers, I used the `docker-compose.yml` file format instead of using the `docker run` commands because I feel that it is easier and more readable.

``` yaml
version: '3'

services:
  nginx:
    container_name: static-website
    image: nginx:bookworm
    volumes:
      - ./templates:/etc/nginx/templates
      - ./html:/usr/share/nginx/html
      - ./conf:/etc/nginx/conf
    ports:
      - "<rpi-ip>:8888:80"
    environment:
      - NGINX_HOST=rpi5.local
      - NGINX_PORT=80
```

Docker compose files specify their options using the format `<machine-options>:<container-options>`. This means that `8888:80` is using the RPi's port `8888` that binds to the container's port `80`, and I will be able to access the website via `http://<rpi-ip>:8888`.

Finally, to deploy the container, I ran the command `docker compose up -d`. Make sure that you only do this while in same directory as the `docker-compose.yml`. For me, I stored everything at `$HOME/docker-containers/nginx`.

Here is my directory structure for `$HOME/docker-containers/nginx`:

``` tree
nginx
W:.
+---conf
|   +---nginx.conf
+---html
|   +---<my-static-website-files>
\---templates
```

## Samba

I did not create (read: code up) the website on the RPi itself. Instead I created it on my computer, and I transferred it over to the RPi via the Samba Share I created.

### Installation

Update the `apt` package index:

``` bash
sudo apt update
```

Install Samba through `apt`:

``` bash
sudo apt samba
```

Check if Samba has been installed:

``` bash
samba --version
```

### Post-Installaion

Finally, to create the share, I edit `smb.conf`:

``` bash
sudo nano /etc/samba/smb.conf
```

At the bottom of `smb.conf`, I added my share in the following format:

``` conf
[NAME-OF-SHARE]
        comment = Description of Share
        path = /path/to/share
        writeable = yes
        read only = no
        browsable = yes
        public = yes
        create mask = 0777
        directory mask = 0777
```

Now, for Samba to reload the changes in the `smb.conf`, I ran the command:

``` bash
sudo service smbd restart
```

Next, I added a password to the share:

``` bash
sudo smbdpasswd -a $USER
```

Last but not least, I went back to my computer (that runs Windows), I opened File Explorer, and under the `Computer` tab, I clicked `Map network drive`, and entered `\\<rpi-ip>\<NAME-OF-SHARE>`.

[prev](./00-Setup.md)---[next](./02-Deploying-a-VPN.md)
