# Active Directory Lab - VirtualBox

## Overview
This lab demonstrates setting up a **mini corporate network** using VirtualBox with a **Windows Server Domain Controller** and a **Windows 10 client machine**. The lab covers:

- Creating Organizational Units (OUs)  
- Bulk user creation with PowerShell  
- Domain joining a client machine  
- DHCP and NAT configuration  

> **Note:** Initial server preparation, installation of roles, promotion to domain controller, and DHCP configuration were performed manually using the **GUI**. Automation is only applied for user account creation and joining client machines to the domain.

---
<p align="center">
  <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/e9ce4bd2-061b-4ecb-aabc-d85f51e65b60" />
</p>


---
## Lab Components

- **Domain Controller VM (Windows Server)**  
  - Active Directory Domain Services (AD DS)  
  - DHCP Server  
  - NAT configured for internet access  

- **Client VM (Windows 10)**  
  - Internal network connection  
  - Domain-joined automatically using PowerShell script  

---

## Directory Structure
│
├─ Names.txt # List of users for bulk creation
├─ CreateUsers.ps1 # PowerShell script to create users in AD
├─ README.md # Documentation
└─ VirtualBox_ISOs/
├─ WindowsServer.iso
└─ Windows10.iso


## Manual Setup Steps

1. **Server Preparation**  
   - Install Windows Server on your VM (e.g., Windows Server 2019/2022).  
   - Set a static IP address for the server to ensure consistent network connectivity.  
   - Configure the subnet mask and default gateway according to your lab network design.  
   - Set a proper computer name for identification in the domain (e.g., `DC01`).  
   - Install updates and ensure the server has network connectivity.  

2. **Install Roles via GUI**  
   - Open **Server Manager** and navigate to **Add Roles and Features**.  
   - Select **Active Directory Domain Services (AD DS)**:  
     - This role allows the server to manage users, computers, and policies in a domain.  
   - Select **DHCP Server**:  
     - Enables the server to assign IP addresses to client machines dynamically.  
   - Complete the installation wizard and restart if prompted.  

3. **Promote Server to Domain Controller**  
   - In **Server Manager**, click the notification flag and select **Promote this server to a domain controller**.  
   - Choose **Add a new forest** and enter your desired domain name (e.g., `mydomain.com`).  
   - Configure the **Domain Controller Options**:  
     - Set the Directory Services Restore Mode (DSRM) password.  
   - Verify NetBIOS name and DNS options, then complete the wizard.  
   - Allow the server to restart after promotion.  

4. **Configure DHCP**  
   - Open **DHCP Management Console** from Server Manager > Tools.  
   - Create a **New Scope**:  
     - Define the range of IP addresses clients can receive.  
     - Configure lease duration.  
   - Configure **Server Options**:  
     - Add **Default Gateway** (typically the domain controller or router IP).  
     - Add **DNS Server** (usually the domain controller IP).  
   - Activate the scope to start assigning IP addresses.  

> These manual steps prepare the server to act as both a domain controller and DHCP server, forming the backbone of your lab Active Directory environment.

---

## Automated Setup Steps

### Bulk User Creation (PowerShell)
- Pull user names from `Names.txt`  
- Create a `SecureString` password object  
- Create a new OU named `_Users`  
- Loop through each user and generate:  
  - First initial + last name as username  
  - Display name and employee ID  
  - Password: `Password1` (never expires)  
- Enable the user account in Active Directory  

### Domain Join Client VM
- Change hostname to match the lab diagram (e.g., `Client1`)  
- Join the domain `mydomain.com`  
- Log in using one of the newly created domain accounts  

---

## PowerShell Script Usage

```powershell
# Navigate to the script directory
cd C:\Users\<YourUser>\Desktop\ADLab

# Run the script
.\CreateUsers.ps1

---
## Verification

### Active Directory
- Open **Active Directory Users and Computers**.  
- Confirm that the `_Users` OU contains all newly created user accounts.

### DHCP
- Ensure client machines receive IP addresses from the correct DHCP scope.  
- Verify that the **Default Gateway** and **DNS settings** are correct.

### Client VM
- Confirm the VM has successfully joined the domain (`mydomain.com`).  
- Log in using one of the newly created domain user accounts.  
- Test connectivity by pinging:  
  - The domain controller  
  - External websites (e.g., `google.com`) to ensure internet access

## Notes
- Duplicate usernames in `Names.txt` may generate errors, but the script will continue creating the remaining accounts.  
- Use **Windows 10 Pro** for client VMs; Home editions cannot join a domain.  
- Ensure the client VM uses an **Internal Network Adapter** to receive a DHCP lease from the domain controller.

