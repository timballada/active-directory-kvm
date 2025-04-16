# Active Directory Lab Using KVM Virtual Machines

## Overview
This lab demonstrates how to deploy a complete **Active Directory (AD)** environment using **KVM virtual machines** on a Linux system (Ubuntu/Debian). By leveraging virtualization through **virt-manager**, this setup simulates a real-world Windows Server domain controller and Windows 10 client infrastructure entirely within a local Linux host.

## Learning Objectives
- Install and configure **KVM**, **libvirt**, and **virt-manager**
- Create isolated and NAT-based virtual networks for AD communication
- Install **Windows Server 2019** and promote it to a **Domain Controller**
- Install **Active Directory Domain Services (AD DS)**, **DHCP**, and **NAT**
- Create and manage AD users and Organizational Units (OUs)
- Use **PowerShell** for Active Directory automation
- Join a **Windows 10 client VM** to the domain

## Lab Architecture Diagram
```
                          +--------------------+
                          |   Linux Host (KVM) |
                          +--------------------+
                                     |
                 +-------------------+-------------------+
                 |                                       |
      +------------------+                  +-------------------+
      | Windows Server   |  <----> 172.16.0.1 (Internal NIC)    |
      | 2019 (Domain     |                                      |
      | Controller / DNS |                                      |
      | DHCP / NAT)      |                                      |
      +------------------+                  +-------------------+
                 |                                       |
        (External NAT)                             (Internal Isolated)
                 |                                       |
           Internet Access                     +--------------------+
                                               | Windows 10 Client  |
                                               | (Joined to Domain) |
                                               +--------------------+
```
---

### Initial Setup (Ubuntu/Debian):

---

Download these two iso images from the sites below:

- [Windows Server 2019](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- [Windows 10](https://www.microsoft.com/en-us/software-download/windows10ISO)

Verify CPU supports virtualization:

```sh
egrep -c '(vmx|svm)' /proc/cpuinfo
```

An output of 1 or more indicates the system does support virtualization, but you may have to go
into the BIOS and enable it manually.

Install KVM and dependencies:

```sh
sudo apt install bridge-utils cpu-checker libvirt-clients libvirt-daemon-system qemu-kvm virtinst
```

Verify installation:

```sh
kvm-ok
#Expected Output:
INFO: /dev/kvm exists
KVM acceleration can be used
```

Install Virt-Manager (GUI for KVM):

```sh
sudo apt install virt-manager
```

This is a very basic installation of the essential packages that allow running KVM and Virt-Manager.
If this does not work on your system or you would like a different set of packages, these sources may help:

- https://sysguides.com/install-kvm-on-linux
- https://phoenixnap.com/kb/ubuntu-install-kvm

---

### Network Configuration

---

Load into Virt-Manager by typing `virt-manager` in the terminal or finding it in the applications menu.

On the top bar click `Edit` then `Preferences`.

In the `General` section toggle the checkmark to `Enable XML editing`, then close out of `Preferences`.

![image](https://github.com/user-attachments/assets/e8476f55-2027-4616-9e56-3044b915ade3) ![image](https://github.com/user-attachments/assets/785877a4-57c0-45bc-9aee-85d85ca3953f)

---

Select `Edit` again, then `Connection Details`.

You should have a default `NAT` connection like this:

![image](https://github.com/user-attachments/assets/153de4c0-a915-4b85-848d-0cc25e004b42) ![image](https://github.com/user-attachments/assets/1ec84aa7-5d68-41cb-bd9e-bccae0963e0b)

At the bottom of the window click the `+` button to add a new network.

![image](https://github.com/user-attachments/assets/f8aa9dce-7561-4a20-a58b-7717bfb8f618)

Name it 'internal-network, set the mode to 'Isolated', and click Finish.

![image](https://github.com/user-attachments/assets/99676123-d2ef-42d7-a716-c7c575235921)

---

In the `Connection Details` window, select the new `internal-network`, then press the red 'x' to stop the network and allow editing.

![image](https://github.com/user-attachments/assets/cbb6d78e-b3d3-45ed-9bed-1852803d9d7b)

Click on the `XML` tab and delete all of the elements selected below.
This will disable automatic ip addressing from the DHCP server.

![image](https://github.com/user-attachments/assets/a201ebad-e688-4976-aceb-0d4bd6344df3)

Click `Apply` in the bottom right corner to save the changes.

Your final XML should look similar to this:

```xml
<network>
  <name>internal-network</name>
  <uuid>1e2c200d-baa4-47a2-be17-b7ecc8a1c678</uuid>
  <bridge name="virbr1" stp="on" delay="0"/>
  <mac address="52:54:00:c5:63:26"/>
</network>

```
Click the `▶` button to reactivate the internal network.

![image](https://github.com/user-attachments/assets/10fe0a3f-add6-4c0d-b7c0-3fc4b2610476)

---

### Creating & Configuring the Domain Controller Virtual Machine

---


In Virt-Manager, click the `create a new virtual machine` button and select local install, then find the iso file and select it:

![image](https://github.com/user-attachments/assets/9d73474a-d1ba-40da-9d89-3fc2ddd001f4)

The option for auto detecting the iso media type is usually selected by default.
Toggle off that option, manually type in the OS, and it will give an autocompletion for you to select
, e.g. `Microsoft Windows Server 2019 (win2k19)`

![image](https://github.com/user-attachments/assets/4718ccf1-1c05-4d79-8c01-5dfe089823de)

Select a RAM and CPU amount for both virtual machines that your PC can handle when they are both running.
For example I have 32GB RAM and 12 physical CPU cores, and gave each of my VMs 8GB and 4 cores. I recommend starting low and working up to see what your PC can handle.

Be cautious how many cores you provision. The GUI is misleading by saying I have "24" CPUs
available, but that is actually my TOTAL amount of processing units. Type the command `lscpu` in
the terminal. Multiply the values of `Core(s) per socket:` and `Socket(s):` to find the amount of "physical cores" in your PC.
The general rule is to use 50-75% of your PC's physical cores for VMs.

![image](https://github.com/user-attachments/assets/ec16f0e2-92f3-47f2-860c-ed637767fa82) ![new12step](https://github.com/user-attachments/assets/4636f03f-7f73-42ff-88a1-d6189bc6700d)

Provide a minimum of 30GB for the VM. Make sure the network is on the default: 'NAT'. Toggle the option to customize
before install, then click `Finish`.

![image](https://github.com/user-attachments/assets/c3e323ff-1985-4a28-84d1-3821a6b57707)

Click the `Add Hardware` button at the bottom of the window. This will open another window to add
virtual hardware. Go to `Network` and select the isolated network we created as the source.

![image](https://github.com/user-attachments/assets/01ef3c50-969e-419a-abea-9e4948d39d28)

You will now see two virtual "NICs", one for accessing the internet and one for an "internal" connection between
the VMs.

![image](https://github.com/user-attachments/assets/6ad57a56-9ac5-4b6d-a909-80af53eb4bec)

Select `Begin Installation` in the top left to launch the VM.

Click `Next` and `Install Now` then select either of the `Desktop Experience` OS options before continuing.

![image](https://github.com/user-attachments/assets/86cef0f3-c35c-4b97-8644-9cc3aa1c8caf)

Accept "TOS" and select the `Custom Install` option and `Next` to start installing. Select an easy and memorable
password as this is just a lab. Click the `Send Key` button on the top bar and `Ctl+Alt+Delete` to login.

![image](https://github.com/user-attachments/assets/d3e766eb-2330-4826-80bb-bcfac08b4503)

---

<!-- ### (OPTIONAL) Configure copy/paste functionality -->

Go to Settings > Network & Internet > Change adaptor options.
On each adaptor, Right click then go to Status > Details.
One adaptor will have an address assigned by the DHCP server, such as 192.168.122.3, 10.0.3.20, etc.

![image](https://github.com/user-attachments/assets/2c0982cc-1a3c-4625-866a-81f79801fe5c)
![image](https://github.com/user-attachments/assets/73e9d42b-2cfb-4bca-89b5-c255ca4c827f)

The other adaptor will have an address beginning with 169.254. This is the adaptor for
the internal/isolated network. Since we removed the option for the adaptor to automatically
obtain an ip address from the DHCP server, it was assigned a link-local address 169.254.x.x.
A link-local address only allows connections with other machines on the local network, but does not
allow external access to the internet.

Name the adaptor with a normal ip address to `__EXTERNAL__` and the one with a link-local address
to `__INTERNAL__`.

![image](https://github.com/user-attachments/assets/f2a10bd5-389c-45a5-bb22-ee8b363207c5)

`__INTERNAL__` > Properties > Internet Protocol Version 4

Assign static IP:

   IP address: `172.16.0.1`
   Subnet mask: `255.255.255.0`

Assign DNS as loopback address:

   Preferred DNS server: `127.0.0.1`

![image](https://github.com/user-attachments/assets/819a4dce-6324-4901-9e74-e2e093c6ba73)

Right click the start menu

Click on `System` > `Rename this PC`

Rename to `DC`, which stands for "Domain Controller"

`Next` > `Restart`

---

### Installing & Configuring AD Domain Services

---

1. Open `Server Manager`
2. Under `Configure this local server` click `Add roles and features`
3. In `Server Selection` pick the server where you want to install, which for us there is only one option
4. In `Server Roles` Add `Active Directory Domain Services`
5. Keep clicking Next and then Install
6. Click the flag icon on the top right and then click `Promote this server to a domain controller`
7. In `Deployment Configuration` Check `Add a new forest` & Root domain name: `mydomain.com`
8. In `Domain Controller Options` put same memorable password, then Install
9. Login after restart to `MYDOMAIN\Administrator` account
10. Click `START` > `Windows Administrative Tools` > `Active Directory Users and Computers`
11. Right click `mydomain.com` > `New` > `Organizational Unit`
12. Name it `_ADMINS` & uncheck `Protect container from accidental deletion`
13. Right click `_ADMINS` > `New` > `User`
14. Use your name & for logon name use `a-FirstInitialLastName` convention    
15. Same memorable password & uncheck `User must change password at next logon` & check `Password never expires` then `Finish`

 ![image](https://github.com/user-attachments/assets/79c3c99c-4a80-4e7a-85cb-90b3925e680e)
  ![image](https://github.com/user-attachments/assets/a9c7379b-595b-49f7-b466-9159a718891e)
    
16. Right click the new account in _ADMINS then click `Properties` > `Member Of` > `Add`
17. Type `domain admins` in object names search then click `Check Names`
18. Should resolve to `Domain Admins` then click `OK` > `Apply` > `OK`, then sign out
19. Click the `Send Key` button on the top bar and `Ctl+Alt+Delete` then `Other user`
20. Login with new domain admin account credentials

    ![image](https://github.com/user-attachments/assets/90d1e533-05dc-4ded-a394-f455828bbfa6)


---

### Installing & Configuring NAT and DHCP

---

1. In `Server Manager` click `Add roles and features` & in `Server Roles` Add `Remote Access`
2. In `Role Services` Add `Routing` (DirectAccess and VPN will be automatically added) then click Next until install

![image](https://github.com/user-attachments/assets/19f82369-ed65-4441-a6b6-dea669875f51)

3. After install click `Tools` > `Routing and Remote Access`
4. Right click `DC (local)` > `Configure and Enable Routing and Remote Access`
5. In `Configuration` check `Network address translation`
6. In `NAT Internet Connection` check `_EXTERNAL_` (retry until it shows that option) then Finish

![image](https://github.com/user-attachments/assets/c7baeb76-aade-4273-87ed-ace151269d8a)
   
7. Click `Add Roles and Features` & in `Server Roles` add `DHCP Server` & install
8. After install click `Tools` > `DHCP` & you will see both IPv4/IPv6 are down in `dc.mydomain.com`
9. Right click `IPv4` > `New Scope` & name it the IP range e.g. `172.16.0.100-200`
10. Set ip address range, length, & subnet mask e.g. `172.16.0.100-200`, `24`, `255.255.255.0`![image](https://github.com/user-attachments/assets/25a25651-927c-4c82-bd33-983bdcb4f031)

11. You don't need to change anything for the excluded addresses or lease range
12. Check `Yes` in `Configure DHCP Options`
13. Enter domain controller ip `172.16.0.1` for default gateway & click `Add`
14. Use domain controller as DNS server, with Parent domain: `mydomain.com` & IP address: `172.16.0.1`
15. Skip over `WINS Server`
16. Select `Yes` for `Activate Scope`
17. Right click `dc.mydomain.com` > `Authorize` then `Refresh`
18. Both IPv4 & IPv6 should both be up now

![image](https://github.com/user-attachments/assets/7c8b0efb-ea2f-4e1f-8e04-4760eceb688c)

---

### Creating Users in Active Directory using PowerShell

---

1. Click `Configure this local server` switch `IE Enhanced Security Configuration` to `Off`
2. Open Internet Explorer and search the below link to my repo with the powershell script & the txt file with 500 random names I generated, then click open (It will probably give you a warning to download since it can't verify it's from a trusted source aka github):

   https://github.com/timballada/ad-make-users/archive/main.zip

   **[Heres the actual repo if you want to view the two files](https://github.com/timballada/ad-make-users)**

![image](https://github.com/user-attachments/assets/fa377001-7748-4f81-8238-bddb0ee70916)
![image](https://github.com/user-attachments/assets/c5f02d39-882a-46bf-b894-91655312defa)


4. Run `PowerShell ISE` as Administrator (Optional: Add your name to the random names txt file)
5. `Open Script` & open the PowerShell script
6. In Terminal run these commands:

```pwsh
PS C:\Windows\system32> Set-ExecutionPolicy Unrestricted
PS C:\Windows\system32> cd C:\users\a-tballada\desktop\ad-make-users-main
```

7. Now that you are in the correct directory click play on the script

![image](https://github.com/user-attachments/assets/7b5a377e-18d6-4819-aac2-40770f685f92)
![image](https://github.com/user-attachments/assets/1a3d939f-b210-47b8-834d-be2a4677e17f)


---

### Creating & Configuring the Windows 10 Client Virtual Machine

---

1. Add new vm with the windows 10 iso
2. Make sure it's using the Internal Network Adaptor
3. Select `I don't have a product key` & then select `Windows 10 Pro`
4. Select `Custom install`
5. Select `I don't have internet` & `Continue with limited setup` (Don't create Microsoft account)
6. Name `user` & no password
7. Make sure you can access the internet
8. Right click `START` > `System` > `Rename this PC (advanced)` > `Change`
9. Computer name: `CLIENT1` & Domain: `mydomain.com`
10. Add either your domain or normal account to have permissions to join the domain & restart
11. Now you can login to any of the 500 created users on the windows 10 machine
