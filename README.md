# Running Active Directory with KVM

<!-- Brief Summary/Explanation-->

### Initial Setup (Ubuntu/Debian):

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

```
sudo apt install virt-manager
```

This is a very basic installation of the essential packages that allow running KVM and Virt-Manager.
If this does not work on your system or you would like a different set of packages, these sources may help:

- https://help.ubuntu.com/community/KVM/Installation
- https://sysguides.com/install-kvm-on-linux
- https://phoenixnap.com/kb/ubuntu-install-kvm

### Network Configuration

Load into Virt-Manager by typing `virt-manager` in the terminal or finding it in the applications menu.
It should open into the manager window, as shown below: 



|On the top bar click 'Edit' then 'Preferences'.| In the 'General' section toggle the checkmark to 'Enable xml editing'.|
|---|---|
|![image](https://github.com/user-attachments/assets/e8476f55-2027-4616-9e56-3044b915ade3)|![image](https://github.com/user-attachments/assets/785877a4-57c0-45bc-9aee-85d85ca3953f)|



On the top bar click 'Edit' then 'Connection Details'

You should have a default 'NAT' connection that looks similar to this.

<!-- 2step | 3step -->

At the bottom of the window click the `+` button to add a new network.

<!-- 4step -->

Name it 'internal-network, set the mode to 'Isolated', and click Finish.

<!-- 5step -->

Make sure 'internal-network' is selected, then press the red 'x' to stop the network and allow editing.

<!-- 6step -->

Click on the 'XML' tab and remove all of the elements selected below.
This will disable automatic ip addressing from the DHCP server.

<!-- 7step -->

Your final XML should look similar to this:

```xml
<network>
  <name>internal-network</name>
  <uuid>1e2c200d-baa4-47a2-be17-b7ecc8a1c678</uuid>
  <bridge name="virbr1" stp="on" delay="0"/>
  <mac address="52:54:00:c5:63:26"/>
</network>

```

### Provisioning Virtual Machines

Download these two iso images from the sites below:

- [Windows Server 2019](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- [Windows 10](https://www.microsoft.com/en-us/software-download/windows10ISO)
