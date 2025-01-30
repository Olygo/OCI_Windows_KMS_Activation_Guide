# OCI Windows KMS Activation Guide

This guide explains how to activate a Windows instance created on an imported custom image using OCI KMS.

# Overview

If you've imported a custom Windows image into OCI and wish to use an OCI-provided Windows license instead of your existing one, follow this guide.
This process applies to the following scenarios:
- Windows has **not yet been activated** (no license key provided).- Windows is **already activated** (a license key is present).

To transition to an OCI-provided Windows license, we will:
1. **Reset the activation state** and **remove the existing product key** from the system.2. **Register the instance with OCI’KMS** to obtain and activate an **OCI-managed Windows license.**
By completing these steps, your instance will be licensed under OCI's Windows licensing framework. 

You will be then charged for the Windows license.


[Download PDF Guide](./OCI_Windows_KMS_Activation_Guide.pdf)


# Migration Steps:


### UPDATE LICENSE TYPE

If you want to use OCI KMS you must [enable ‘OCI-Provided’ under licence type](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/changewinlicense.htm#changing_the_windows_license) for each instance. 

**Instance will automatically restart to apply this change.**

### CHECK CURRENT ACTIVATION STATUS

IT doesn't matter wheter the instance was activated or not.

open a PowerShell Prompt **with Admin Rights** and verify the upgraded Windows version using:

```ruby
Get-ComputerInfo -Property WindowsProductName
slmgr /dvl
```

### REMOVE THE EXISTING PRODUCT KEY


```ruby
slmgr /upk
```

### CLEAR THE LICENSE KEY FROM THE REGISTRY

```ruby
slmgr /cpky 
```

### RESTART THE INSTANCE

```ruby
shutdown /r /t 0
```

### REGISTER THE OCI KMS SERVER

```ruby
slmgr /skms 169.254.169.253:1688
```

### RESET ACTIVATION STATE

```ruby
slmgr /rearm
```

### RESTART THE INSTANCE

```ruby
shutdown /r /t 0
```

### RETRIEVE THE PROPER GENERIC VOLUME LICENSE KEY

[Key Management Services (KMS) client activation and product keys](https://learn.microsoft.com/en-us/windows-server/get-started/kms-client-activation-keys?tabs=server2022%2Cwindows1110ltsc%2Cversion1803%2Cwindows81)


### INSTALL THE KMS CLIENT KEY

Example using GVL key for Windows 2022 Standard:

```ruby
slmgr /ipk VDYBN-27WPP-V4HQT-9VMD4-VMK7H
```

### FORCE ACTIVATION

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

