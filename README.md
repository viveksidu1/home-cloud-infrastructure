# Home Cloud Infrastructure (Self-Hosted)
This project outlines the setup of a personal, self-hosted server designed to act as a private alternative to mainstream cloud storage and streaming services.

## What is the Purpose?

PRIVACY: No large companies will have access to your data, ensuring full privacy and security.


CLOUD ALTERNATIVE: Acts as a replacement for Google Drive, Google Photos, Microsoft OneDrive, Apple iCloud, and Dropbox.


AUTOMATIC BACKUPS: Automatically syncs phone and computer data (photos and videos) and frees up local storage.


MEDIA STREAMING: Serves as a Netflix or Amazon Prime alternative to stream your movie collection on any device.


GLOBAL ACCESS: View your data from anywhere on the internet using any device or browser.

## Hardware Prerequisites
To run this server, you will need the following hardware in your home or office:


INTERNET CONNECTION: High-speed connection required.


NETWORKING: Router or switch with a free LAN port (1Gbps preferred).


SERVER HARDWARE: A spare computer (desktop or laptop) to convert into a server.


INPUT/OUTPUT: Monitor, keyboard, and mouse for the initial installation.


STORAGE MEDIA: * 64GB SanDisk USB PenDrive for the installer ISO.

64GB HP USB PenDrive for the OS installation storage.


PRIMARY PC: For remote setup and management.

### Reference Build Configuration
For this project, a PC was built with components from Amazon India for approximately Rs. 20,500/-: | Component | Specification | | :--- | :--- | | CPU | Intel Pentium Gold G6400 (2C/4T @ 4.0 GHz)  | | Motherboard | Gigabyte H410M  | | RAM | 16GB (8GB x 2) DDR4 2666 RAM  | | Storage 1 | 500GB NVME SSD x1  | | Storage 2 | SATA 2.5inch 500GB HDD x1  | | Storage 3 | 256GB SSD x2  |
+4

## Software Prerequisites
The setup uses legal, mostly open-source, and free software:
+1


OS: TrueNAS SCALE 


STORAGE/CLOUD: NextCloud 


PHOTOS: PhotoPrism 


MEDIA: Jellyfin 


MANAGEMENT: File Browser 


REMOTE ACCESS: CloudFlare Tunnel 


ADMINISTRATIVE: One credit/debit card and one TLD domain name.

## Installation Steps
### 1. TrueNAS USB Installation Disk Creation
Download the TrueNAS SCALE ISO to your primary PC.

Use Rufus to create the installation USB.


Important: You must force Rufus to use "DD Image mode" because TrueNAS SCALE (based on Debian) hates the default partition modification of ISO mode.

![Image: SanDisk USB Pen Drive used for installation] 

![Image: Rufus settings showing DD Image mode selection] 

### 2. Installing TrueNAS SCALE
Connect the SanDisk installer USB, HP storage USB, monitor, and keyboard to the server PC.

In the BIOS, set the installer USB as the first boot device and turn off Secure Boot.

Select Start TrueNAS SCALE Installation from the boot menu.

![Image: TrueNAS SCALE GRUB boot screen] 

Select the HP USB Flash Drive as the destination disk for the OS installation and keep other disks unselected.

![Image: Installer screen for choosing destination media] 

![Image: HP 64GB USB Flash Drive used for OS installation] 

After installation, reboot and remove the installer USB.

The system will display a console screen with the Web GUI IP Address. Note this down.

![Image: TrueNAS console setup showing web interface URL] 

## Post-Installation Setup
### 3. Web GUI and Static IP
Access the server via a web browser using the IP (e.g., http://192.168.0.108).

Login with the credentials set during installation.

![Image: TrueNAS SCALE Web Login page] 

Navigate to Network -> Interfaces.

Edit your interface (e.g., eno1), disable DHCP, and manually add your current IP to make it static.

![Image: Network interface settings in TrueNAS] 

![Image: Editing interface to set static IP] 

### 4. Storage Pool Configuration
Go to Storage and click Create Pool.
+1

![Image: Storage dashboard showing the Create Pool button] 

Verify available disks.

![Image: Disks list showing available NVMe and SATA drives] 


Single SSD Pool (Stripe): Name the pool (e.g., DYAZO_SSD_240GB) and select Stripe layout.
+2

![Image: Naming the pool in the Creation Wizard] 

![Image: Selecting Stripe layout] 

Use Manual Disk Selection to add the disk to a VDEV.

![Image: Manual disk selection button] 

![Image: Dragging disks into VDEVs in manual selection] 

![Image: Finalizing the Stripe VDEV] 


Mirror Pool (Dual Disk): Repeat the process but select Mirror layout and add two identical disks (e.g., 500GB HDD + 500GB SSD) to ensure data redundancy.
+1

![Image: Selecting Mirror layout] 

![Image: Mirror VDEV with two disks for redundancy] 

Our Final Storage Pool Table: | Pool Name | Type | Details | | :--- | :--- | :--- | | DYAZO_SSD_240GB | Stripe | 1x 256GB SSD  | | NVME_240GB | Stripe | 1x 256GB SSD  | | MIRROR_480GB | Mirror | 1x 500GB HDD & 1x 500GB SSD  |

![Image: Disk table showing all pools assigned] 

### 5. System Config

Localization: Set your Time Zone (e.g., Kolkata) in System Settings -> General -> Localization.


Swap: Set the storage pool to your fastest drive and configure the swap size to at least half of your total RAM in System Settings -> Advanced -> Storage.

![Image: Storage settings showing pool selection and swap size] 

## Application Deployment
### Application Storage Pool
Go to Apps and click Settings.

Choose your fastest SSD pool (e.g., NVME_240GB) as the default for application storage.

![Image: Selecting the default pool for Apps] 

### 6. File Browser App
Required because TrueNAS SCALE lacks a native file explorer.

During installation, add your storage pools as "Additional Storage Locations" using Host Path types.
+1

Default login is admin / admin.

![Image: File Browser App discovery page] 

![Image: Adding NVME_240GB host path to File Browser] 

![Image: Adding MIRROR_480GB host path to File Browser] 

![Image: Adding DYAZO_SSD_240GB host path to File Browser] 

![Image: Apps page showing File Browser running] 

![Image: File Browser login screen] 

### 7. Dataset Creation for Apps
Navigate to Datasets.

Select your Mirror pool and click Add Dataset.

Name it Apps and set the preset to Apps.

![Image: Datasets menu with Add Dataset highlighted] 

![Image: Creating the Apps dataset] 

Edit Permissions to give the apps user Full Control.

![Image: Permissions section for a dataset] 

![Image: ACL Editor setting Full Control for the apps user] 

Create sub-datasets for specific data (e.g., NC_Data, NC_DB) to ensure important user data stays on the Mirror drive.
+2

![Image: Final dataset hierarchy showing Apps, NC_Data, and NC_DB] 

![Image: Removing extra apps user permissions in ACL Editor] 

### 8. Nextcloud Deployment
Install from the Apps Catalogue.

Set Host Paths for "Nextcloud User Data Storage" and "Nextcloud Postgres Data Storage" to the datasets created on the Mirror pool.
+1

![Image: Mapping Nextcloud user data to the Host Path] 

![Image: Mapping Nextcloud Postgres data to the Host Path] 

![Image: Apps page showing Nextcloud deploying] 

![Image: Nextcloud login portal] 

### 9. Jellyfin Deployment
Configure Jellyfin Config Storage to a Host Path on the Mirror pool.

Add Additional Storage with a Host Path to your media folder (e.g., JellyFin_Data) and set the mount path to /media.

![Image: Jellyfin configuration storage mapping] 

![Image: Jellyfin media library additional storage mapping] 

Complete the initial setup wizard to create an admin account and add your media library.
+2

![Image: Jellyfin welcome screen] 

![Image: Jellyfin user account creation] 

![Image: Jellyfin library setup screen] 

![Image: Metadata language selection in Jellyfin] 

![Image: Enabling remote access in Jellyfin] 

![Image: Jellyfin dashboard with Add Media Library option] 

![Image: Selecting content type and library name] 

![Image: Selecting the /media/Movies folder path] 

### 10. PhotoPrism
Install using default settings on the fast NVMe pool.

In Settings -> Services, add your Nextcloud account to import photos automatically.

![Image: PhotoPrism gallery interface] 

## Remote Access with Cloudflare Tunnel
### Domain and Cloudflare Setup
Purchase a cheap TLD domain (e.g., .online or .pw).
+1

Add the site to a free Cloudflare account.
+1

![Image: Cloudflare login page] 

![Image: Adding a new site to Cloudflare dashboard] 

![Image: Entering the domain name] 

![Image: Selecting the Free Cloudflare plan] 

Delete any existing DNS records and update your domain registrar's Name Servers to those provided by Cloudflare.
+1

![Image: Deleting old DNS records in Cloudflare] 

![Image: Cloudflare assigned Nameservers] 

![Image: Updating Nameservers at the domain registrar] 

![Image: Domain showing as active in Cloudflare] 

### Creating the Tunnel
In Zero Trust, navigate to Network -> Tunnel and add a new Cloudflared tunnel.

![Image: Cloudflare Zero Trust Tunnels page] 

![Image: Selecting the Cloudflared tunnel type] 

Name the tunnel and copy the secret token.

![Image: Naming the tunnel] 

![Image: Connector installation showing the token command] 

Install the Cloudflare Tunnel app on TrueNAS SCALE and paste the token.

![Image: Apps dashboard with Cloudflared running] 

Add Public Hostnames in Cloudflare to map subdomains to your local App ports.
+1

Example: nextcloud.lazytourer.online -> http://192.168.0.108:9001.
+1

### 11. Nextcloud External Access Configuration
Open System Settings -> Shell and run sudo su.

![Image: TrueNAS Shell with root access] 

Edit config.php using Nano: nano /mnt/[Your_Pool]/ix-applications/releases/nextcloud/volumes/ix_volumes/html/config/config.php.

Add your subdomain to the 'trusted_domains' array.
+1

![Image: Editing config.php to add trusted domains] 

## Mobile App Configuration
Download the Nextcloud app from the Play Store or App Store.

Login via your server IP or subdomain.

Enable Auto Upload in settings and configure your camera folders.
+1

![Image: Nextcloud mobile app settings] 

![Image: Enabling Auto upload in mobile app] 

![Image: Configuring folder for auto upload] 

![Image: Setting remote folder path to /Photos/] 

![Image: Uploads progress screen in Nextcloud mobile app] 

## Conclusion
Your self-hosted home cloud is now fully operational, accessible globally, and ready for private backups and streaming. Ongoing costs are limited to the monthly electricity (approx. Rs. 250–300) and your annual domain renewal
