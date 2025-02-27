# Running Active Directory with KVM

Include Brief Summary/Explanation

---

### Initial Setup (Ubuntu/Debian):

---

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

- https://help.ubuntu.com/community/KVM/Installation
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

### Provisioning Virtual Machines

---

Download these two iso images from the sites below:

- [Windows Server 2019](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- [Windows 10](https://www.microsoft.com/en-us/software-download/windows10ISO)


#### Windows Server 2019

In Virt-Manager, click the `create a new virtual machine` button and follow steps below:

<!--8step-->
<!--9step-->

The option for auto detecting the iso media type is usually selected by default.
Toggle off that option, manually type in the OS, and it will give an autocompletion for you to select
, e.g. `Microsoft Windows Server 2019 (win2k19)`

<!--10step-->

Select a RAM and CPU amount for both virtual machines that your PC can handle when they are both running.

For example I have 32GB RAM and 24 CPU cores, and gave each of my VMs 8GB and 4 cores.

<!--11step-->

Be cautious how many cores you provision. The GUI is misleading by saying I have "24" CPUs
available, but that is actually my TOTAL amount of processing units. Type the command `lscpu` in
the terminal. Multiply the values of `Core(s) per socket:` and `Socket(s):` to find the amount of "physical cores" in your PC.
The general rule is to use 50-75% of your PC's physical cores for VMs.

<!--12step-->

Provide a minimum of 30GB for the VM. Make sure the network is on the default: 'NAT'. Toggle the option to customize
before install, then click `Finish`.

<!--13step-->

Click the `Add Hardware` button at the bottom of the window. This will open another window to add
virtual hardware. Go to `Network` and select the isolated network we created as the source.

<!--14step-->

You will now see two virtual "NICs", one for accessing the internet and one for an "internal" connection between
the VMs.

<!--15step-->

Select `Begin Installation` in the top right to launch the VM.

Click `Next` and `Install Now` then select either of the `Desktop Experience` OS options before continuing.

<!--16step-->

Accept "TOS" and select the `Custom Install` option and `Next` to start installing. Select an easy and memorable
password as this is just a lab. Click the `Send Key` button on the top bar and `Ctl+Alt+Delete` to login.

<!--17step-->

