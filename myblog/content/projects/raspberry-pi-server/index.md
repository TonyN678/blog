+++
date = '2024-12-17T15:01:15+11:00'
title = 'RaspberryPi Home Server'
toc = true
+++
# Setting Up a Raspberry Pi Home Server with Docker

A Raspberry Pi can be transformed into a powerful and lightweight home server with the right configuration. In this guide, I will walk you through my setup process, starting from installing the Raspberry Pi OS Lite to configuring Docker containers and setting up essential services for remote file access, media streaming, and more.

---

## Step 1: Install 64-bit Raspberry Pi OS Lite

To maximise performance and compatibility, I opted for the 64-bit version of Raspberry Pi OS Lite with no GUI. Here are the steps I followed:

1. **Download the OS**: Visit the official Raspberry Pi website and download the latest [64-bit Raspberry Pi OS Lite](https://www.raspberrypi.com/software/).
2. **Flash the SD Card**: Flash the OS onto the sd card using tool like Raspberry Pi Imager.
3. **Enable SSH**: Set up password for user and enable SSH before starting the flashing proccess.
4. **Boot the Pi**: Insert the SD card into the Raspberry Pi and power it on. Connect via SSH.

---

## Step 2: Install Docker

Docker is the backbone of this setup, allowing me to run and manage services efficiently in isolated containers, also uses far lesser computer resources than on a VM or BareMetal.
- [pimylifeup's guide](https://pimylifeup.com/raspberry-pi-docker/) 

1. **Update the System**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. **Install Docker**:
   ```bash
   curl -sSL https://get.docker.com | sh
   ```
3. **Add User to Docker Group**:
   ```bash
   sudo usermod -aG docker $USER
   ```
   Log out and back in to apply group changes.

---

## Step 3: Install Portainer

[Portainer](https://docs.portainer.io/start/install-ce/server/docker/linux) provides an intuitive web interface for managing Docker containers.

1. **Pull the Portainer Image**:
   ```bash
   docker volume create portainer_data
   docker run -d -p 8000:8000 -p 9443:9443 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
   ```
2. **Access the Interface**: Open a browser and navigate to `https://<Raspberry_Pi_IP>:9443` to set up Portainer.

---
## Step 4: Mount an External Hard Drive
This guide explains how to mount an external hard drive to your Raspberry Pi, set ownership and permissions, and make these changes persist across reboots.

### Identify the Drive
1. Plug in your external hard drive.
2. Identify the drive using `lsblk`:
   ```bash
   lsblk
   ```
   Look for your drive (e.g., `/dev/sda1`).

3. Get the UUID of the partition:
   ```bash
   sudo blkid /dev/sda1
   ```
   Copy the UUID of your drive (e.g., `UUID="1234-5678"`).

### Create a Mount Point
1. Create a directory to mount the drive:
   ```bash
   sudo mkdir /mnt/extdrive
   ```

2. (Optional) Adjust permissions for the directory:
   ```bash
   sudo chmod 755 /mnt/data
   ```

### Add the Drive to `/etc/fstab`
1. Open `/etc/fstab` for editing:
   ```bash
   sudo nano /etc/fstab
   ```

2. Add an entry for the drive:
   ```
   UUID=1234-5678 /mnt/data ext4 defaults 0 2
   ```
   - Replace `1234-5678` with your drive's UUID.
   - Adjust the filesystem type (`ext4`, `ntfs`, etc.) as necessary.

3. Save and exit the editor.

4. Test the configuration:
   ```bash
   sudo mount -a
   ```
   If there are no errors, your drive is now mounted.

### Set Ownership and Permissions
1. Change ownership of the mount point to a specific user (e.g., `tien`):
   ```bash
   sudo chown -R tien:tien /mnt/data
   ```

2. (Optional) Adjust permissions if multiple users need access:
   ```bash
   sudo chmod -R 775 /mnt/data
   ```

### Automate Ownership Changes (if Needed)
The `chown` command does not persist across reboots. To automate this:

#### Option A: Use `/etc/rc.local`
1. Open or create the `rc.local` file:
   ```bash
   sudo nano /etc/rc.local
   ```

2. Add the `chown` command before `exit 0`:
   ```bash
   chown -R tien:tien /mnt/data
   ```

3. Save the file and make it executable:
   ```bash
   sudo chmod +x /etc/rc.local
   ```

#### Option B: Use a `systemd` Service
1. Create a new `systemd` service:
   ```bash
   sudo nano /etc/systemd/system/fix-permissions.service
   ```

2. Add the following content:
   ```ini
   [Unit]
   Description=Fix ownership of /mnt/data
   After=local-fs.target

   [Service]
   Type=oneshot
   ExecStart=/bin/chown -R tien:tien /mnt/data

   [Install]
   WantedBy=multi-user.target
   ```

3. Save and enable the service:
   ```bash
   sudo systemctl enable fix-permissions.service
   ```


### Verify the Setup
1. Reboot your Raspberry Pi:
   ```bash
   sudo reboot
   ```

2. After reboot, check the ownership and permissions of the mount point:
   ```bash
   ls -ld /mnt/data
   ```

You should see the correct ownership (e.g., `tien:tien`).

By following these steps, your external hard drive will be properly mounted and accessible, with ownership and permissions configured to persist across reboots.

---
## Step 5: Install FileBrowser

[FileBrowser](https://filebrowser.org/installation) is a lightweight file management solution, perfect for 4GB RAM Raspberry Pi 4. We could also create tunnel to FileBrowser with CloudFlares to have a NAS/cloud functions.     
 I chose FileBrowser because Samba is too minimal and don't have web GUI, NextCloud is over-featured and heavy.

1. **Run FileBrowser in Docker**:
```
docker run -v /path/to/root:/srv -v /path/to/filebrowser.db:/database/filebrowser.db -e PUID=$(id -u) -e PGID=$(id -g) -p 8080:80 filebrowser/filebrowser:s6
```
  > **_NOTE:_**  The "/path/to/root" is storage space(/mnt/extdrive), "/path/to/filebrowser" is database. Only modify the path to the left of the colon ':'.

2. **Access FileBrowser**: Navigate to `http://<Raspberry_Pi_IP>:8080` and log in with the default credentials.

---

## Step 6: Buy a Domain Name

I purchased the domain name `tienng.xyz` from [gen.xyz](https://gen.xyz). After purchasing, I configured DNS settings to point to my Raspberry Pi using Cloudflare.

---

## Step 7: Set Up Cloudflare Tunnel

Since I could not enable port forwarding on my school's network, I used Cloudflare Tunnel to access my Raspberry Pi services remotely.

1. **Register an Account on Cloudflare**:
2. **Verify Domain Name**: Add a domain, choose Free plan, input given Cloudflare's nameservers to my domain's setting on [gen.xyz](https://gen.xyz), you will be taken to Quick Start Guide and can just choose default options. 
Remember to press "check Nameservers again" and wait for confirmation in email.
3. **Create a Tunnel**: After successfully verifying your domain name, press your domain name, in Access tab press "Launch Zero Trust", on the left sidebar find Networks tab and choose Tunnels,
create a new tunnel and follow the intructions on the page.
4. **Access Services**: I mapped `cloud.tienng.xyz` to FileBrowser, enabling secure external access to my files.

**--> BUT at this point**, we have our tunnel up and running but literally ANYONE can get into our network, there may be cases where you
DO want to open up a tunnel connection to the whole wide world (such as if you’re running a public web server), but in this case,
we’re just trying to access some stuff on our own local LAN – we absolutely do not want the whole wide world to be able to 
connect in – we only want to be able to access it ourselves!

5. **Add Application for Authentication**: 
    - https://www.crosstalksolutions.com/cloudflare-tunnel-easy-setup/
    - https://www.youtube.com/watch?v=ey4u7OUAF3c&list=WL&index=14 (NetworkChuck Remote Network)
    - https://www.youtube.com/watch?v=65FdHRs0axE&list=WL&index=9&pp=gAQBiAQB (Restrict Cloudflare Application)
    - https://www.youtube.com/watch?v=wdmbAo02ktQ&list=WL&index=8&pp=gAQBiAQB (Restrict to Google&Github Auth)
    - https://www.youtube.com/watch?v=lnw616HiINY&list=WL&index=5&pp=gAQBiAQB (SSH Remote Connection) 
---


## Step 8: Set Up Immich and Navidrome

Immich and Navidrome are excellent solutions for managing photos and music streaming, respectively.

1. **Install Immich**:
   ```bash
   docker run -d --name=immich -p 3000:3000 -v /mnt/extdrive/immich:/data ghcr.io/immich-app/server
   ```
2. **Install Navidrome**:
   ```bash
   docker run -d --name=navidrome -p 4533:4533 -v /mnt/extdrive/music:/data navidrome/navidrome:latest
   ```
3. **Access Services**:
   - Immich: `http://<Raspberry_Pi_IP>:3000`
   - Navidrome: `http://<Raspberry_Pi_IP>:4533`

---

# Conclusion

This setup transformed my Raspberry Pi into a versatile home server capable of file management, media streaming, and photo management—all accessible remotely through a secure Cloudflare Tunnel. With Docker, adding or modifying services is a breeze, making this setup highly customisable and scalable.




