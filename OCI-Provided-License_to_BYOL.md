# Switch from OCI-Provided License to Bring Your Own License

This guide explains how to reconfigure a Windows instance from an **OCI-provided license** model to a **Bring Your Own License** model.

# Overview

To transition to a **Bring Your Own License** model, we will:

1. **Reset the Windows activation** state and remove any existing product key from the system.
2. **Register the instance with your own KMS** server if you are using Generic Volume License Keys (GVLK).
3. **Activate using a MAK key (Multiple Activation Key - Volume Licensing)** if you are not using GVLKs (no KMS server required in this case).


By completing these steps, your instance will no longer be licensed under OCI’s Windows licensing framework.
As a result, Windows license charges will no longer apply.


# Migration Steps:


### UPDATE LICENSE TYPE

If you want to use your own Activation Key you must [enable ‘Bring Your Own License’ under licence type](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/changewinlicense.htm#changing_the_windows_license) for each instance. 

**Instance will automatically restart to apply this change.**

### CHECK CURRENT ACTIVATION STATUS

Open a PowerShell Prompt **with Admin Rights**

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

**This step is not required if you don't have a KMS server and are using a MAK key.**

```ruby
slmgr /skms your_kms_ip:your_kms_tcp_port
```
Example:

```ruby
slmgr /skms 10.32.0.100:1688
```

### RETRIEVE THE PROPER GENERIC VOLUME LICENSE KEY

**This step is not required if you do not have a KMS server and are using a MAK key.**

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
| Windows Server 2012		              | BN3D2-R7TKB-3YPBD-8DRP2-27GG4 | 
| Windows Server 2012 Standard        | XC9B7-NBPP2-83J2H-RHMBY-92BT4 | 
| Windows Server 2012 Datacenter      | 48HP8-DN98B-MYWDG-T2DCC-8W83P | 
| Windows Server 2012 Essentials      | HTDQM-NBMMG-KGYDT-2DTKT-J2MPV | 
| Windows Server 2008 R2 Standard     | YC6KT-GKW9T-YTKYR-T4X34-R7VHC | 
| Windows Server 2008 R2 Enterprise   | 489J6-VHDMP-X63PK-3K798-CPX3Y | 
| Windows Server 2008 R2 Datacenter   | 74YFP-3QFB3-KQT8W-PMXWJ-7M648 | 


### INSTALL THE KMS CLIENT KEY

**This step is not required if you do not have a KMS server and are using a MAK key.**

Example using GVL key for Windows 2025 Standard:

```ruby
slmgr /ipk TVRH6-WHNXV-R9WG3-9XRFY-MY832
```

### INSTALL THE MAK KEY

**This step IS REQUIRED if you do not have a KMS server and are using a MAK key.**

```ruby
slmgr /ipk XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```

### FORCE ACTIVATION

```ruby
slmgr /ato
```

### CHECK THE ACTIVATION STATUS

```ruby
slmgr /dlv
```

