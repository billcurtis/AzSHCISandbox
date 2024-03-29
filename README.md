# Azure Stack HCI Operator`s Sandbox (3/11/2023)

Notes for 2023 version:

1. Standardized on Server 2022.
2. Windows Admin Center is now downloaded automatically, so you'll always have the latest version!
3. Google Chrome is no longer installed as Edge is now in Server 2022.
4. Azure CLI is installed during the installation in the AdminCenter virtual machine
5. Static RAM assigments are now made instead of dynamic. **Highly** recommend using a system with 128GB of RAM and 3 TB of storage.




![alt text](res/AzSHCISandbox.png "Graphic of a fully deployed Azure Stack HCI Operator's Sandbox")

## Start learning how to operate Azure Stack HCI without the complicated setup!

Deploy a full Azure Stack HCI environment on either a Server 2022 Hyper-V Host or a Server 2022 Azure VM and start learning how to operate Azure Stack HCI! Did you mess something up?  Just re-deploy! No muss, no fuss!

This version is set for deployment in non-cloud environment.  Microsoft has taken this project and set it to work in the cloud. If you are planning on deploying this sandbox to Azure, it is recommended that you go here:  https://learn.microsoft.com/en-us/azure-stack/hci/guided-quick-deploy-eval?tabs=global-admin 

Want to test the deployment of VMs? PaaS services? Do you need a development environment for Azure Stack HCI without the huge Azure bill?  This is for you.

The Azure Stack HCI Operator's Sandbox is a series of scripts that creates a [HyperConverged](https://docs.microsoft.com/en-us/windows-server/hyperconverged/) environment using four nested Hyper-V Virtual Machines. The purpose of the Azure Stack HCI Operator's Sandbox  is to provide operational training on Microsoft Azure Stack HCI as well as provide a development environment for DevOPs to assist in the creation and
validation of some Azure Stack HCI features without the time consuming process of setting up physical servers and network routers\switches.

>**Azure Stack HCI Operator's Sandbox is not a production solution!** The Azure Stack HCI Operator's Sandbox's scripts have been modified to work in a limited resource environment as well as in a Microsoft Azure virtual machine. Because of this, it is not fault tolerant, is not designed to be highly available, and lacks the nimble speed and resilience of a **real** Microsoft Azure Stack HCI deployment.

Also, be aware that Azure Stack HCI Operator's Sandbox  is **NOT** designed to be managed by System Center Virtual Machine Manager (SCVMM), but by Windows Admin Center. Let me know if you need a version that uses SCVMM. If there are enough requests, I will create a version for that.

## History

Azure Stack HCI Operator's Sandbox  is based on a *really* fast refactoring of scripts that I wrote for myself to rapidly create online labs for Microsoft Software Defined Networking using SCVMM. The SCVMM scripts have been stripped out and replaced with a stream-lined version that uses Windows Admin Center for the management of Microsoft SDN. As time has progressed, this environment has become invaluable to performing operational evaluations of Windows Admin Center and SDN without the requirement for hard to
configure hardware.

## Scenarios

The ``SCRIPTS\Scenarios`` folder in this solution will be updated quite frequently with full solutions\examples of popular SDN scenarios. Please keep checking for updates!

## Quick Start (TLDR)

You probably are not going to read the requirements listed below, so here are the steps to get SDN Sandbox up and running on a **single host** :

1. Download and unzip this solution to a drive on a Intel based System with at least 128gb of RAM, 2022 (or higher) Hyper-V Installed, and a Hyper-V External Switch attached to a network that can route to the Internet and provides DHCP. Proxies are not supported.

> **Note** - It is best to use Windows Server **Desktop Experience** on a single machine as it is easier to RDP into the **Console** VM.

2. Create VHDX files for the **2022 Datacenter Desktop Experience** and for Azure Stack HCI. Using **Convert-WindowsImage** to create images from the ISO is recommended.


```
Install-PackageProvider -Name Nuget
Install-Module -Name Convert-WindowsImage  

Convert-WindowsImage -SourcePath 'M:\temp\New-SDNVHDXFile\en-us_windows_server_2022_updated_oct_2022_x64_dvd_c5b651c9.iso'  `
 -Edition "Windows Server 2022 Datacenter Evaluation (Desktop Experience)" `
 -VHDFormat VHDX `
 -DiskLayout UEFI  `
 -VHDPath 'M:\temp\New-SDNVHDXFile\GUI.VHDX'

```


3. Edit the .PSD1 configuration file (do not rename it) to set:
    
    * The Password needs to be the same as the local administrator password on your physical Hyper-V Host

    * Product Key for Server 2022
      
    >**Warning!** The Configuration file will be copied to the console drive during install. **The 2022 product key will be in plain text and not deleted or hidden!**     
    
    * The paths to the VHDX files that you just created.
    * Set ``HostVMPath`` where your VHDX files will reside. (*Ensure that there is at least 250gb of free space!*)
   
4. On the Hyper-V Host, open up a PowerShell console (with admin rights) and navigate to the ``AzSHCISandbox` folder and run ``.\New-AzSHCISandbox``.

5. It should take a little over an hour to deploy (if using SSD drives).

6. Using RDP, log into the 'Admincenter' virtual machine with your creds: User: Contoso\Administrator Password: Password01

7. Launch the link to Windows Admin Center

8. Add the Hyper-Converged Cluster *AzStackCluster* to *Windows Admin Center*  and you're off and ready to go!

![alt text](res/AddHCCluster.png "Add Hyper-Converged Cluster Connection")

## Configuration Overview

AzSHCISandbox will automatically create and configure the following:

* Active Directory virtual machine
* Windows Admin Center virtual machine
* Routing and Remote Access virtual machine (to emulate a *Top of Rack (ToR)* switch)
* Two node Hyper-V S2D cluster with each having a SET Switch
* One Single Node Network Controller virtual machine
* One Software Load Balancer virtual machine
* Two Gateway virtual machines (one active, one passive)
* Management and Provider VLAN and networks 
* Private, Public, and GRE VIPs and automatically configured in Network Controller
* VLAN to provide testing for L3 Gateway Connections (VLAN 2000)


## Hardware Prerequisites

The AzSHCISandbox can only run on a single host.

|  Number of Hyper-V Hosts | Memory per Host   | HD Available Free Space   | Processor   |  Hyper-V Switch Type |
|---|---|---|---|---|
| 1  | 128gb | 250gb SSD\NVME   | Intel\AMD - 4 core Hyper-V   | External Hyper-V VMSwitch **with** Internet Access |

Please note the following regarding the hardware setup requirements:

* Windows Server 2022 (Standard or Datacenter) or higher Hyper-V **MUST** already have been installed along with the RSAT-Hyper-V tools.

* It is recommended that you disable all disconnected network adapters or network adapters that will not be used.

* It is **STRONGLY** recommended that you use SSD or NVME drives (especially in single-host). This project has been tested on a single host with four 5400rpm drives in a Storage Spaces pool with acceptable results, but there are no guarantees.

## Software Prerequisites

### Required VHDX files:

 **GUI.vhdx** - Sysprepped **Desktop Experience** version of Windows Server 2022 **Datacenter**. Only Windows Server 2022 Datacenter is supported.       
  
**AzSHCI.vhdx** - Same requirements.

>**Note:** A Product Key WILL be required to be entered into the Configuration File for Windows Server 2022 Datacenter. If you are using VL media, use the [KMS Client Keys](https://docs.microsoft.com/en-us/windows-server/get-started/kmsclientkeys) keys for the version of Windows you are installing.


## Configuration File (AzSHCISandbox-Config) Reference

The following are a list of settings that are configurable and have been fully tested. You may be able to change some of the other settings and have them work, but they have not been fully tested.

>**Note:** Changing the IP Addresses for Management Network (*default of 192.168.1.0/24*) has been succesfully tested.


| Setting                  |Type| Description                                                                                                                         |  Example                           |
|--------------------------------------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|
| ﻿ConfigureBGPpeering                  | bool   | Peers the GW and MUX VMs with the BGP-ToR-Router automatically if ProvisionNC = $true  
| azsHCIVHDXPath                         | string | This value controls the location of the Azure Stack HCI OS Image.                                                                                                                                                              | C:\AzHCIVHDs\AzStackHCIPreview1.vhdx |
| DCName                               | string | Name of the domain controller. Must be limited to 14 characters.                                                                                                                                                | fabrikam.dc                 |
| GUIProductKey                        | string | Product key for GUI.                                                                                                                                                               |                             |
| guiVHDXPath                          | string | This value controls the location of the Windows Server 2022 Image VHDX.                                                                                                                                                               | C:\2022 VHDS\2022_GUI.vhdx  |
| HostVMPath                           | string | This value controls the path where the Nested VMs will be stored on all hosts                                                                                                                                   | V:\VMs                      |
| InternalSwitch                       | string | Name of internal switch that the SDN Lab VMs will use in Single Host mode. This only applies when using a single host. If the internal switch does not exist, it will be created.                               | Fabrikam                    |                                                                                                                                                                      |                             |
| natDNS                               | string | DNS address for forwarding from Domain Controller. Currently set to Cloudflare's 1.1.1.1 by default.                                                                                                            | 1.1.1.1                     |
| natExternalVMSwitchName              | string | Name of external virtual switch on the physical host that has access to the Internet.                                                                                                                           | Internet                    |
| natSubnet                            | string | This value is the subnet is the NAT router will use to route to  SDNMGMT to access the Internet. It can be any /24 subnet and is only used for routing. Keep the default unless it overlaps with a real subnet. | 192.168.46.0/24             |
| natHostSubnet                            | string    | This value is the subnet is the NAT router will use to route to  AzSMGMT to access the Internet. It can be any /24 subnet and is only used for routing. 
| natHostVMSwitchName                | int    | Internal Switch that will be used for NAT access from the VM.                                                                                                                             | InternalNat                      |                                                                              |                             |
| NestedVMMemoryinGB  (Deprecated)                 | int    | This has been replaced with a calculation that will maximize the use of RAM on the installed physical machine or virtual machine.                                                                                                                                | 16GB                        |
| ProvisionNC                          | bool   | Provisions Network Controller Automatically.                                                                                                                                                                    |                             |
| SDNAdminPassword                     | string | Password for all local and domain accounts.                                                                                                                                                                     | Password01                  |
| SDNDomainFQDN                        | string | Limit name (before the.xxx) to 14 characters as the name will be used as the NetBIOS name.                                                                                                                      | fabrikam.com                |
| SDNLABMTU                            | int    | Controls the MTU for all Hosts. If using multiple physical hosts. Ensure that you have configured MTU on physical nics on the hosts to match this value.                                                        |                             |
| SDNMGMTMemoryinGB                    | int    | This value controls the amount of RAM for the SDNMGMT Nested VM which contains only the Console, Router, Admincenter, and DC VMs.                                                                               | 13GB                        |
| Setting                              | Type   | Description                                                                                                                                                                                                     | Example                     |

