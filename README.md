# Home Cloud Infrastructure (Self-Hosted)
This project outlines the setup of a personal, self-hosted server designed to act as a private alternative to mainstream cloud storage and streaming services.

## What is the Purpose?

PRIVACY: No large companies will have access to your data, ensuring full privacy and security.

CLOUD ALTERNATIVE: Acts as a replacement for Google Drive, Google Photos, Microsoft OneDrive, Apple iCloud, and Dropbox.

AUTOMATIC BACKUPS: Automatically syncs phone and computer data (photos and videos) and frees up local storage.

MEDIA STREAMING: Serves as a Netflix or Amazon Prime alternative to stream your movie collection on any device.

GLOBAL ACCESS: View your data from anywhere on the internet using any device or browser.



![diagram](images/diagram.png)



## Hardware Prerequisites
To run this server, you will need the following hardware in your home or office:


INTERNET CONNECTION: High-speed connection required.

NETWORKING: Router or switch with a free LAN port (1Gbps preferred).

SERVER HARDWARE: A spare computer (desktop or laptop) to convert into a server.

INPUT/OUTPUT: Monitor, keyboard, and mouse for the initial installation.

STORAGE MEDIA: * 64GB SanDisk USB PenDrive for the installer ISO.
->64GB HP USB PenDrive for the OS installation storage.

PRIMARY PC: For remote setup and management.


### Reference Build Configuration
For this project, a PC was built with components from Amazon India for approximately Rs. 20,500/-: | Component | Specification | | :--- | :--- | | CPU | Intel Pentium Gold G6400 (2C/4T @ 4.0 GHz)  | | Motherboard | Gigabyte H410M  | | RAM | 16GB (8GB x 2) DDR4 2666 RAM  | | Storage 1 | 500GB NVME SSD x1  | | Storage 2 | SATA 2.5inch 500GB HDD x1  | | Storage 3 | 256GB SSD x2  |

## Software Prerequisites
The setup uses legal, mostly open-source, and free software:


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



![Sandisk](images/SanDisk.jpeg)
![Rufus](images/rufus1.jpeg)
![rufus](images/rufus2.jpeg)



### 2. Installing TrueNAS SCALE
Connect the SanDisk installer USB, HP storage USB, monitor, and keyboard to the server PC.

In the BIOS, set the installer USB as the first boot device and turn off Secure Boot.

Select Start TrueNAS SCALE Installation from the boot menu.

![install](images/Install.jpeg)

Select the HP USB Flash Drive as the destination disk for the OS installation and keep other disks unselected.



![boot](images/boot.jpeg)
![hp](images/hp.jpeg)



After installation, reboot and remove the installer USB.

The system will display a console screen with the Web GUI IP Address. Note this down.

![web](images/web.jpeg)

## Post-Installation Setup
### 3. Web GUI and Static IP
Access the server via a web browser using the IP (e.g., http://192.168.0.108).

Login with the credentials set during installation.



![truenas1](images/truenas1.png) 
![truenas2](images/truenas2.png)



Navigate to Network -> Interfaces.

Edit your interface (e.g., eno1), disable DHCP, and manually add your current IP to make it static.



![ip](images/ip1.png)
![ip](images/ip2.png)
![ip](images/ip3.png)



### 4. Storage Pool Configuration
Go to Storage and click Create Pool.

![pool](images/pool1.png)


Verify available disks.


![pool](images/pool2.png)



Single SSD Pool (Stripe): Name the pool (e.g., DYAZO_SSD_240GB) and select Stripe layout.



![pool](images/pool1.png) 
![pool](images/pool2.png) 



Use Manual Disk Selection to add the disk to a VDEV.



![vdev](images/vdev1.png)
![vdev](images/vdev2.png) 
![vdev](images/vdev3.png)
![vdev](images/vdev4.png)



Mirror Pool (Dual Disk): Repeat the process but select Mirror layout and add two identical disks (e.g., 500GB HDD + 500GB SSD) to ensure data redundancy.



![mirror](images/mirror1.png)
![mirror](images/mirror2.png) 




Our Final Storage Pool Table: | Pool Name | Type | Details | | :--- | :--- | :--- | | DYAZO_SSD_240GB | Stripe | 1x 256GB SSD  | | NVME_240GB | Stripe | 1x 256GB SSD  | | MIRROR_480GB | Mirror | 1x 500GB HDD & 1x 500GB SSD  |

![list](images/list.png)

### 5. System Config

Localization: Set your Time Zone (e.g., Kolkata) in System Settings -> General -> Localization.


Swap: Set the storage pool to your fastest drive and configure the swap size to at least half of your total RAM in System Settings -> Advanced -> Storage.



![sys](images/sys.png) 



## Application Deployment
### Application Storage Pool
Go to Apps and click Settings.

Choose your fastest SSD pool (e.g., NVME_240GB) as the default for application storage.



![app-pool](images/app-pool.png) 



### 6. File Browser App
Required because TrueNAS SCALE lacks a native file explorer.

During installation, add your storage pools as "Additional Storage Locations" using Host Path types.

Default login is admin / admin.

File Browser App discovery page
![fb](images/fb1.png)

Adding NVME_240GB host path to File Browser
![fb](images/fb2.png)

Adding MIRROR_480GB host path to File Browser
![fb](images/fb3.png)

Adding DYAZO_SSD_240GB host path to File Browser
![fb](images/fb4.png)

Apps page showing File Browser running
![fb](images/fb5.png)

File Browser login screen
![fb](images/fb6.png)



### 7. Dataset Creation for Apps
Navigate to Datasets.

Select your Mirror pool and click Add Dataset.

Name it Apps and set the preset to Apps.

Datasets menu with Add Dataset highlighted
![dataset](images/dataset1.png)

Creating the Apps dataset
![dataset](images/dataset2.jpg)



Edit Permissions to give the apps user Full Control.

Permissions section for a dataset
![dataset](images/dataset3.jpg)

ACL Editor setting Full Control for the apps user
![dataset](images/dataset4.jpg)



Create sub-datasets for specific data (e.g., NC_Data, NC_DB) to ensure important user data stays on the Mirror drive.

Final dataset hierarchy showing Apps, NC_Data, and NC_DB
![dataset](images/dataset5.jpg)

Removing extra apps user permissions in ACL Editor
![dataset](images/dataset6.jpg)



### 8. Nextcloud Deployment
Install from the Apps Catalogue.

Set Host Paths for "Nextcloud User Data Storage" and "Nextcloud Postgres Data Storage" to the datasets created on the Mirror pool.

Mapping Nextcloud user data to the Host Path
![nc](images/nc1.jpg)

Mapping Nextcloud Postgres data to the Host Path
![nc](images/nc1.jpg)

Apps page showing Nextcloud deploying
![nc](images/nc1.jpg)

Nextcloud login portal

![nc](images/nc1.jpg)



### 9. Jellyfin Deployment
Configure Jellyfin Config Storage to a Host Path on the Mirror pool.

Add Additional Storage with a Host Path to your media folder (e.g., JellyFin_Data) and set the mount path to /media.

Jellyfin configuration storage mapping

![jf](images/jf1.jpg)

Jellyfin media library additional storage mapping

![jf](images/jf2.jpg)



Complete the initial setup wizard to create an admin account and add your media library.

Jellyfin welcome screen

![jf](images/jf3.jpg)

Jellyfin user account creation

![jf](images/jf4.jpg)

Jellyfin library setup screen

![jf](images/jf5.jpg)

Metadata language selection in Jellyfin

![jf](images/jf6.jpg)

Enabling remote access in Jellyfin

![jf](images/jf7.jpg)

Jellyfin dashboard with Add Media Library option

![jf](images/jf8.jpg)

Selecting content type and library name

![jf](images/jf9.jpg)

Selecting the /media/Movies folder path

![jf](images/jf10.jpg)



### 10. PhotoPrism
Install using default settings on the fast NVMe pool.

In Settings -> Services, add your Nextcloud account to import photos automatically.

PhotoPrism gallery interface

![prism](images/prism.png)

## Remote Access with Cloudflare Tunnel
### Domain and Cloudflare Setup
Purchase a cheap TLD domain (e.g., .online or .pw).

Add the site to a free Cloudflare account.

Cloudflare login page

![cloudflare](images/cf1.jpg)


Adding a new site to Cloudflare dashboard

![cloudflare](images/cf2.jpg)


Entering the domain name

![cloudflare](images/cf3.jpg)


Selecting the Free Cloudflare plan

![cloudflare](images/cf4.jpg)


Delete any existing DNS records and update your domain registrar's Name Servers to those provided by Cloudflare.

Deleting old DNS records in Cloudflare

![cloudflare](images/cf5.jpg)


Cloudflare assigned Nameservers

![cloudflare](images/cf6.jpg)


Updating Nameservers at the domain registrar

![cloudflare](images/cf7.jpg)


Domain showing as active in Cloudflare

![cloudflare](images/cf8.jpg)

### Creating the Tunnel
In Zero Trust, navigate to Network -> Tunnel and add a new Cloudflared tunnel.

Cloudflare Zero Trust Tunnels page

![cloudflare](images/cf9.jpg)


Selecting the Cloudflared tunnel type

![cloudflare](images/cf10.jpg)


Name the tunnel and copy the secret token.

Naming the tunnel

![cloudflare](images/cf11.jpg)


Connector installation showing the token command

![cloudflare](images/cf12.jpg)


Install the Cloudflare Tunnel app on TrueNAS SCALE and paste the token.

Apps dashboard with Cloudflared running

![tunnel](images/tun1.jpg)


Add Public Hostnames in Cloudflare to map subdomains to your local App ports.

Example: nextcloud.lazytourer.online -> http://192.168.0.108:9001.

### 11. Nextcloud External Access Configuration
Open System Settings -> Shell and run sudo su.

TrueNAS Shell with root access

![shell](images/root1.jpg)


Edit config.php using Nano: nano /mnt/[Your_Pool]/ix-applications/releases/nextcloud/volumes/ix_volumes/html/config/config.php.

Add your subdomain to the 'trusted_domains' array.

Editing config.php to add trusted domains

![shell](images/root2.jpg)


## Mobile App Configuration
Download the Nextcloud app from the Play Store or App Store.

Login via your server IP or subdomain.

Enable Auto Upload in settings and configure your camera folders.

Nextcloud mobile app settings

![mobile](images/mobile1.jpg)


Enabling Auto upload in mobile app

![mobile](images/mobile2.jpg)


Configuring folder for auto upload

![mobile](images/mobile3.jpg)


Setting remote folder path to /Photos/

![mobile](images/mobile4.jpg)


Uploads progress screen in Nextcloud mobile app

![mobile](images/mobile5.jpg)


## Conclusion
Your self-hosted home cloud is now fully operational, accessible globally, and ready for private backups and streaming. Ongoing costs are limited to the monthly electricity (approx. Rs. 250–300) and your annual domain renewal
