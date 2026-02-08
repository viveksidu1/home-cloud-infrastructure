# ☁️ Home Cloud Infrastructure (Self-Hosted)

This project outlines the setup of a personal, self-hosted server designed to act as a private alternative to mainstream cloud storage and streaming services.

---

## 🎯 What is the Purpose?

* **PRIVACY:** No large companies will have access to your data, ensuring full privacy and security.
* **CLOUD ALTERNATIVE:** Acts as a replacement for Google Drive, Google Photos, Microsoft OneDrive, Apple iCloud, and Dropbox.
* **AUTOMATIC BACKUPS:** Automatically syncs phone and computer data (photos and videos) and frees up local storage.
* **MEDIA STREAMING:** Serves as a Netflix or Amazon Prime alternative to stream your movie collection on any device.
* **GLOBAL ACCESS:** View your data from anywhere on the internet using any device or browser.

![diagram](images/diagram.png)

---

## 🛠️ Hardware Prerequisites

To run this server, you will need the following hardware:

* **INTERNET:** High-speed connection required.
* **NETWORKING:** Router/switch with a free LAN port (1Gbps preferred).
* **SERVER:** A spare desktop or laptop to convert into a server.
* **I/O:** Monitor, keyboard, and mouse for the initial installation.
* **STORAGE MEDIA:** * 64GB SanDisk USB PenDrive (Installer ISO).
    * 64GB HP USB PenDrive (OS Installation Storage).
* **PRIMARY PC:** For remote setup and management.

### 💰 Reference Build Configuration
*Total Cost: Approx. ₹20,500/-*

| Component | Specification |
| :--- | :--- |
| **CPU** | Intel Pentium Gold G6400 (2C/4T @ 4.0 GHz) |
| **Motherboard** | Gigabyte H410M |
| **RAM** | 16GB (8GB x 2) DDR4 2666 RAM |
| **Storage 1** | 500GB NVME SSD x1 |
| **Storage 2** | SATA 2.5inch 500GB HDD x1 |
| **Storage 3** | 256GB SSD x2 |

---

## 💻 Software Prerequisites

The setup uses legal, mostly open-source, and free software:
**OS:** TrueNAS SCALE | **Cloud:** NextCloud | **Photos:** PhotoPrism | **Media:** Jellyfin | **Access:** Cloudflare Tunnel

---

## 🚀 Installation Steps

### 1. TrueNAS USB Installation Disk
1. Download the TrueNAS SCALE ISO.
2. Use Rufus to create the installation USB.
> **Important:** You must force Rufus to use **"DD Image mode"** because TrueNAS SCALE (based on Debian) hates the default ISO mode.

![Sandisk](images/SanDisk.jpeg)
![Rufus](images/rufus1.jpeg)
![rufus](images/rufus2.jpeg)

### 2. Installing TrueNAS SCALE
1. Connect the installer USB and HP storage USB to the server.
2. In BIOS, set the installer USB as the first boot device and **turn off Secure Boot**.
3. Select **Start TrueNAS SCALE Installation**.

![install](images/Install.jpeg)

4. Select the **HP USB Flash Drive** as the destination disk.

![boot](images/boot.jpeg)
![hp](images/hp.jpeg)

5. After installation, reboot and remove the installer USB. Note the **Web GUI IP Address**.

![web](images/web.jpeg)

---

## ⚙️ Post-Installation Setup

### 3. Web GUI and Static IP
Access the server via a web browser (e.g., `http://192.168.0.108`).

![truenas1](images/truenas1.png) 
![truenas2](images/truenas2.png)

Navigate to **Network -> Interfaces**. Disable DHCP and manually add your IP to make it static.

![ip1](images/ip1.png)
![ip2](images/ip2.png)
![ip3](images/ip3.png)

### 4. Storage Pool Configuration
Go to **Storage** and click **Create Pool**.

![pool1](images/pool1.png)
![pool2](images/pool2.png)

**Stripe Pool (Fast Storage):-**
Name the pool (e.g., DYAZO_SSD_240GB) and select Stripe layout. Use Manual Disk Selection to add the disk to a VDEV.

![vdev1](images/vdev1.png)
![vdev2](images/vdev2.png) 
![vdev3](images/vdev3.png)
![vdev4](images/vdev4.png)

**Mirror Pool (Redundancy):-**
Repeat the process but select Mirror layout and add two identical disks (e.g., 500GB HDD + 500GB SSD) to ensure data redundancy.

![mirror1](images/mirror1.png)
![mirror2](images/mirror2.png) 

#### Final Storage Layout:
![list](images/list.png)

### 5. System Config
Set **Time Zone** (Kolkata) and **Swap** settings on your fastest drive.

Localization:- Set your Time Zone (e.g., Kolkata) in System Settings -> General -> Localization.

Swap:- Set the storage pool to your fastest drive and configure the swap size to at least half of your total RAM in System Settings -> Advanced -> Storage.

![sys](images/sys.png) 

---

## 📦 Application Deployment

### Application Storage Pool
![app-pool](images/app-pool.png) 

### 6. File Browser App
Required because TrueNAS SCALE lacks a native file explorer.

During installation, add your storage pools as "Additional Storage Locations" using Host Path types.

Default login is admin / admin

![fb1](images/fb1.png)
![fb2](images/fb2.png)
![fb3](images/fb3.png)
![fb4](images/fb4.png)
![fb5](images/fb5.png)
![fb6](images/fb6.png)

### 7. Dataset Creation for Apps
Ensure user data stays on the **Mirror** drive for safety.

Navigate to Datasets.

Select your Mirror pool and click Add Dataset. Name it Apps and set the preset to Apps.

Edit Permissions to give the apps user Full Control.

Create sub-datasets (e.g., NC_Data, NC_DB) on the Mirror drive for redundancy.

![dataset1](images/dataset1.png)
![dataset2](images/dataset2.jpg)
![dataset3](images/dataset3.jpg)
![dataset4](images/dataset4.jpg)
![dataset5](images/dataset5.jpg)
![dataset6](images/dataset6.jpg)

### 8. Nextcloud Deployment
Install from the Apps Catalogue.

Set Host Paths for "Nextcloud User Data Storage" and "Nextcloud Postgres Data Storage" to the datasets created on the Mirror pool.

![nc1](images/nc1.jpg)
![nc1](images/nc1.jpg)
![nc1](images/nc1.jpg)
![nc1](images/nc1.jpg)

### 9. Jellyfin Deployment
Configure Jellyfin Config Storage to a Host Path on the Mirror pool.

Add Additional Storage with a Host Path to your media folder (e.g., JellyFin_Data) and set the mount path to /media.

![jf1](images/jf1.jpg)
![jf2](images/jf2.jpg)
![jf3](images/jf3.jpg)
![jf4](images/jf4.jpg)
![jf5](images/jf5.jpg)
![jf6](images/jf6.jpg)
![jf7](images/jf7.jpg)
![jf8](images/jf8.jpg)
![jf9](images/jf9.jpg)
![jf10](images/jf10.jpg)

### 10. PhotoPrism
Install using default settings on the fast NVMe pool.

In Settings -> Services, add your Nextcloud account to import photos automatically.

![prism](images/prism.png)

---

## 🌐 Remote Access (Cloudflare Tunnel)

### Domain Setup
Purchase a cheap TLD domain.

Add the site to a free Cloudflare account.

Update your domain registrar's Name Servers to Cloudflare's.

![cf1](images/cf1.jpg)
![cf2](images/cf2.jpg)
![cf3](images/cf3.jpg)
![cf4](images/cf4.jpg)
![cf5](images/cf5.jpg)
![cf6](images/cf6.jpg)
![cf7](images/cf7.jpg)
![cf8](images/cf8.jpg)

### Creating the Tunnel
In Zero Trust, create a new Cloudflared tunnel.

Install the Cloudflare Tunnel app on TrueNAS SCALE and paste the token.

Add Public Hostnames (e.g., nextcloud.yourdomain.online -> http://192.168.0.108:9001)

![cf9](images/cf9.jpg)
![cf10](images/cf10.jpg)
![cf11](images/cf11.jpg)
![cf12](images/cf12.jpg)
![tun1](images/tun1.jpg)

### 11. Nextcloud External Access Configuration
Run `sudo su` in the shell and edit `config.php` to add your trusted domain.

Open System Settings -> Shell and run sudo su.

Edit config.php: nano /mnt/[Pool]/ix-applications/releases/nextcloud/volumes/ix_volumes/html/config/config.php.

Add your subdomain to the 'trusted_domains' array.

![root1](images/root1.jpg)
![root2](images/root2.jpg)

---

## 📱 Mobile App Configuration
Download the Nextcloud app.

Login via your subdomain.

Enable Auto Upload in settings for camera folders.

![mobile1](images/mobile1.jpg)
![mobile2](images/mobile2.jpg)
![mobile3](images/mobile3.jpg)
![mobile4](images/mobile4.jpg)
![mobile5](images/mobile5.jpg)

---

## 🏁 Conclusion
Your self-hosted home cloud is now fully operational. 
**Ongoing Costs:** ~₹250–300/month (Electricity) + Annual Domain Renewal.






