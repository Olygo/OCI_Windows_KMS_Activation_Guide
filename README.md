# Switch from a Bring Your Own License model to an OCI-Provided License

If you have imported a custom Windows image into OCI and want to use an OCI-provided Windows license instead of your existing license, follow this guide.

This guide also applies if you created a Windows instance using a BYOL image from the OCI Marketplace and now want to transition to an OCI-provided license model.

**If you want to switch from an OCI-Provided License to a Bring Your Own License**, [follow this guide](./OCI-Provided-License_to_BYOL.md)

This process applies to the following scenarios:

- Windows has **not yet been activated** (no license key provided).
- Windows is **already activated** (a license key is present).

To transition to an OCI-provided Windows license, we will:

1. **Reset the activation state** and **remove the existing product key** from the system.
2. **Register the instance with OCI’KMS** to obtain and activate an **OCI-managed Windows license.**

By completing these steps, your instance will be licensed under OCI's Windows licensing framework. 

You will be then charged for the Windows license.


[Download PDF Guide](./OCI_Windows_KMS_Activation_Guide.pdf)


OCI Compute Instance - **License Type** Considerations:

- **Bring Your Own License (BYOL)**
The instance can establish connectivity to OCI KMS, however, it is not authorized to use KMS functionalities.

- **OCI-Provided License**
The instance can both connect to and fully use OCI Key KMS.

Important:
To be authorized to use OCI KMS, the instance must be created from an image whose Operating System field is set to Windows. Images with "Linux" or "Custom" as the operating system will not be authorized to connect to the OCI KMS.

If the **Operating System** field of your custom image is not set to **Windows**, you can update it from the CloudShell using:

```ruby
oci compute image update --image-id ocid1.image.oc1.region_identifier.xxxxxxxxxx --operating-system "Windows" --operating-system-version "server 2025 standard"
```


# Migration Steps:


### UPDATE LICENSE TYPE

If you want to use OCI KMS you must [enable ‘OCI-Provided’ under licence type](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/changewinlicense.htm#changing_the_windows_license) for each instance. 

**Instance will automatically restart to apply this change.**

### CHECK CURRENT ACTIVATION STATUS

It doesn't matter whether the instance has been activated or not.

Open a PowerShell Prompt **with Admin Rights** and verify the Windows version & edition using:

```ruby
Get-ComputerInfo -Property WindowsProductName
slmgr /dlv
```

### REMOVE THE EXISTING PRODUCT KEY


```ruby
slmgr /upk
```

### CLEAR THE LICENSE KEY FROM THE REGISTRY

```ruby
slmgr /cpky 
```

### RESET ACTIVATION STATE

```ruby
slmgr /rearm
```

### RESTART THE INSTANCE

```ruby
shutdown /r /t 0
```

### REGISTER THE OCI KMS SERVER

```ruby
slmgr /skms 169.254.169.253:1688
```

### RETRIEVE THE PROPER GENERIC VOLUME LICENSE KEY

[Key Management Services (KMS) client activation and product keys](https://learn.microsoft.com/en-us/windows-server/get-started/kms-client-activation-keys?tabs=server2022%2Cwindows1110ltsc%2Cversion1803%2Cwindows81)

| Operating system edition            | KMS Client Product Key        |
| ----------------------------------- | ----------------------------- |
| Windows Server 2025 Standard        | TVRH6-WHNXV-R9WG3-9XRFY-MY832 | 
| Windows Server 2025 Datacenter      | D764K-2NDRG-47T6Q-P8T8W-YP6DF | 
| Windows Server 2022 Standard        | VDYBN-27WPP-V4HQT-9VMD4-VMK7H | 
| Windows Server 2022 Datacenter      | WX4NM-KYWYW-QJJR4-XV3QB-6VM33 | 
| Windows Server 2019 Standard        | N69G4-B89J2-4G8F4-WWYCC-J464C | 
| Windows Server 2019 Datacenter      | WMDGN-G9PQG-XVVXX-R3X43-63DFG | 
| Windows Server 2019 Essentials      | WVDHN-86M7X-466P6-VHXV7-YY726 | 
| Windows Server 2016 Standard        | WC2BQ-8NRM3-FDDYY-2BFGV-KHKQY | 
| Windows Server 2016 Datacenter      | CB7KF-BWN84-R7R2Y-793K2-8XDDG | 
| Windows Server 2016 Essentials      | JCKRF-N37P4-C2D82-9YXRT-4M63B | 
| Windows Server 2012 R2 Standard     | D2N9P-3P6X9-2R39C-7RTCD-MDVJX | 
| Windows Server 2012 R2 Datacenter   | W3GGN-FT8W3-Y4M27-J84CP-Q3VJ9 | 
| Windows Server 2012 R2 Essentials   | KNC87-3J2TX-XB4WP-VCPJV-M4FWM | 
| Windows Server 2012		          | BN3D2-R7TKB-3YPBD-8DRP2-27GG4 | 
| Windows Server 2012 Standard        | XC9B7-NBPP2-83J2H-RHMBY-92BT4 | 
| Windows Server 2012 Datacenter      | 48HP8-DN98B-MYWDG-T2DCC-8W83P | 
| Windows Server 2012 Essentials      | HTDQM-NBMMG-KGYDT-2DTKT-J2MPV | 
| Windows Server 2008 R2 Standard     | YC6KT-GKW9T-YTKYR-T4X34-R7VHC | 
| Windows Server 2008 R2 Enterprise   | 489J6-VHDMP-X63PK-3K798-CPX3Y | 
| Windows Server 2008 R2 Datacenter   | 74YFP-3QFB3-KQT8W-PMXWJ-7M648 | 


### INSTALL THE KMS CLIENT KEY

Example using GVL key for Windows 2025 Standard:

```ruby
slmgr /ipk TVRH6-WHNXV-R9WG3-9XRFY-MY832
```

### FORCE ACTIVATION

```ruby
slmgr /ato
```

### CHECK THE ACTIVATION STATUS

```ruby
slmgr /dlv
```

## Why OCI-Provided licenses use KMS ?

Using a **Key Management Service (KMS)** allows assigning a temporary key that is periodically renewed, rather than a permanent key.

This activation model is dynamic and designed to support enterprises managing a large number of licenses across multiple servers and client computers in an internal network environment.

**How KMS Works:**
	
	- Temporary Key:

		- The key used for activation via KMS is not permanent and is regularly reactivated by an internal KMS server.
		
		- This key must be renewed at regular intervals to ensure that the operating system remains activated without interruption.

	- Renewal Period:
	
		- By default, a machine activated via KMS must "renew" or "revalidate" its activation on a regular basis. (Default: 6 months)
		
		- However, the attempts to reactivate are much more frequent.

	- Renewal Tolerance:
	
		- While the activation interval is set at 180 days, systems attempt to reactivate every 7 days.
		
		- This helps prevent service interruptions in case of KMS server issues or other network errors that could impede activation during this time.

	- Why a Renewal Model?
	
		- Using KMS as an activation method is particularly beneficial for large organizations for several reasons:
		
		- CENTRALIZED MANAGEMENT: KMS allows administrators to manage activations through an internal network, centralizing license administration without requiring an external connection for each activation.

		- LICENSE COMPLIANCE: This model helps businesses stay compliant with Microsoft licensing agreements, as it ensures only verified and approved machines are activated.

		- FLEXIBILITY: It provides flexibility in license management, particularly useful in cases of frequent hardware or software configuration changes, common in large IT environments.

		
This repeated temporary approach ensures that any machine no longer part of the network or organization loses its activation, thereby helping to secure and regulate the use of software licenses.

