---
title: Install PowerShell Az module for Azure Stack Hub 
description: Learn how to install PowerShell for Azure Stack Hub.
author: mattbriggs

ms.topic: article
ms.date: 02/18/2021
ms.author: mabrigg
ms.reviewer: raymondl
ms.lastreviewed:  02/18/2021

# Intent: As an Azure Stack operator, I want to install Powershell Az for Azure Stack.
# Keyword: install powershell azure stack Az

---

# Install PowerShell Az module for Azure Stack Hub

This article explains how to install the Azure PowerShell Az and compatible Azure Stack Hub administrator modules using PowerShellGet. The Az modules can be installed on Windows, macOS, and Linux platforms.

You can also run the Az modules for Azure Stack Hub in a Docker container. For instructions, see [Use Docker to run PowerShell for Azure Stack Hub](../user/azure-stack-powershell-user-docker.md).

If you would like to install PowerShell Resource Modules (AzureRM) module for Azure Stack Hub, see [Install PowerShell AzureRM module for Azure Stack Hub](azure-stack-powershell-install.md).

> [!IMPORTANT]
> There will likely not be new Azure Resource Modules module releases. The Azure Resource Modules modules are under support for critical fixes only. Going forward there will only be Az releases for Azure Stack Hub.

You can use *API profiles* to specify the compatible endpoints for the Azure Stack Hub resource providers.

API profiles provide a way to manage version differences between Azure and Azure Stack Hub. An API version profile is a set of Azure Resource Manager PowerShell modules with specific API versions. Each cloud platform has a set of supported API version profiles. For example, Azure Stack Hub supports a specific profile version such as **2019-03-01-hybrid**. When you install a profile, the Azure Resource Manager PowerShell modules that correspond to the specified profile are installed.

You can install Azure Stack Hub compatible PowerShell Az modules in Internet-connected, partially connected, or disconnected scenarios. This article walks you through the detailed instructions for these scenarios.

## 1. Verify your prerequisites

Az modules are supported on Azure Stack Hub with Update 2002 or later and with the current hotfixes installed. See the [Azure Stack Hub release notes](release-notes.md) for more information.

The Azure PowerShell Az modules work with PowerShell 5.1 or higher on Windows, or PowerShell Core 6.x and later on all platforms. You should install the
[latest version of PowerShell Core](/powershell/scripting/install/installing-powershell#powershell-core) available for your operating system. Azure PowerShell has no other requirements when run on PowerShell Core.

To check your PowerShell version, run the command:

```powershell  
$PSVersionTable.PSVersion
```

### Prerequisites for Windows
To use Azure PowerShell in PowerShell 5.1 on Windows:

1. Update to
   [Windows PowerShell 5.1](/powershell/scripting/windows-powershell/install/installing-windows-powershell#upgrading-existing-windows-powershell)
   if needed. If you're on Windows 10, you already have PowerShell 5.1 installed.
2. Install [.NET Framework 4.7.2 or later](/dotnet/framework/install).
3. Make sure you have the latest version of PowerShellGet. Run `Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force`. 

## 2. Prerequisites for Linux and Mac
PowerShell Core 6.x or later version is needed. Follow the [link](/powershell/scripting/install/installing-powershell-core-on-windows) for instructions

## 3. Uninstall existing versions of the Azure Stack Hub PowerShell modules

Before installing the required version, make sure that you uninstall any previously installed Azure Stack Hub Azure Resource Manager  or Az PowerShell modules. Uninstall the modules by using one of the following two methods:

1. To uninstall the existing Azure Resource Manager and Az PowerShell modules, close all the active PowerShell sessions, and run the following cmdlets:

    ```powershell
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Az.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    ```
    If you hit an error such as 'The module is already in use', close the PowerShell sessions that are using the modules and rerun the above script.

2. Delete all the folders that start with `Azure`, `Az` or `Azs.` from the `C:\Program Files\WindowsPowerShell\Modules` and `C:\Users\{yourusername}\Documents\WindowsPowerShell\Modules` folders. Deleting these folders removes any existing PowerShell modules.

## 4. Connected: Install with internet connectivity

The Azure Stack Az module will work Azure Stack Hub 2002 or later. In addition, the Azure Stack Az module will work with PowerShell 5.1 or greater on a Windows machine, or PowerShell 6.x or greater on a Linux or macOS platform. Using the PowerShellGet cmdlets is the preferred installation method. This method works the same on the supported platforms.

1. Run the following command from a PowerShell session to update PowerShellGet to a minimum of version 2.2.3

    ```powershell  
    Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force
    ```

2. Close your PowerShell session, then open a new PowerShell session so that update can take effect.

3. Run the following command from a PowerShell session:

    ```powershell  
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

> [!Note]  
> Azure Stack Hub module version 2.0.0 is a breaking change. Refer to the [Migrate from AzureRM to Azure PowerShell Az in Azure Stack Hub](migrate-azurerm-az.md) for details.

> [!WARNING]
> You can't have both the Azure Resource Manager (AzureRM) and Az modules installed for PowerShell 5.1 for Windows at the same time. If you need to keep Azure Resource Manager available on your system, install the Az module for PowerShell Core 6.x or later. To do this, [install PowerShell Core 6.x or later](/powershell/scripting/install/installing-powershell-core-on-windows) and then follow these instructions in a PowerShell Core terminal.

## 5. Disconnected: Install without internet connection

In a disconnected scenario, you first download the PowerShell modules to a machine that has internet connectivity. Then, you transfer them to the Azure Stack Development Kit (ASDK) for installation.

Sign in to a computer with internet connectivity and use the following scripts to download the Azure Resource Manager and Azure Stack Hub packages, depending on your version of Azure Stack Hub.

Installation has five steps:

1. Install Azure Stack Hub PowerShell to a connected machine.
2. Enable additional storage features.
3. Transport the PowerShell packages to your disconnected workstation.
4. Manually bootstrap the NuGet provider on your disconnected workstation.
5. Confirm the installation of PowerShell.

### Install Azure Stack Hub PowerShell

::: moniker range=">=azs-2002"
Azure Stack Hub 2002 or later.

You could either use Azure Resource Manager or Az modules. For Azure Resource Manager, see the instructions at [Install PowerShell AzureRM module](powershell-install-az-module.md). The following code saves modules from trustworthy online repository https://www.powershellgallery.com/.

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

Install-module -Name PowerShellGet -MinimumVersion 2.2.3 -Force
Import-Module -Name PackageManagement -ErrorAction Stop

$savedModulesPath = "<Path that is used to save the packages>"
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name Az -Path $savedModulesPath -Force -RequiredVersion 0.10.0-preview
Save-Package -ProviderName NuGet -Source https://www.powershellgallery.com/api/v2 -Name AzureStack -Path $savedModulesPath -Force -RequiredVersion 2.0.2-preview
```
::: moniker-end

> [!NOTE]  
> On machines without an internet connection, we recommend executing the following cmdlet for disabling the telemetry data collection. You may experience a performance degradation of the cmdlets without disabling the telemetry data collection. This is applicable only for the machines without internet connections
> ```powershell
> Disable-AzDataCollection
> ```

### Add your packages to your workstation

1. Copy the downloaded packages to a USB device.

2. Sign in to the disconnected workstation and copy the packages from the USB device to a location on the workstation.

3. Manually bootstrap the NuGet provider on your disconnected workstation. For instructions, see [Manually bootstrapping the NuGet provider on a machine that isn't connected to the internet](/powershell/scripting/gallery/how-to/getting-support/bootstrapping-nuget#manually-bootstrapping-the-nuget-provider-on-a-machine-that-is-not-connected-to-the-internet).

4. Register this location as the default repository and install the `AzureRM` and `AzureStack` modules from this repository:

   ```powershell
   # requires -Version 5
   # requires -RunAsAdministrator
   # requires -Module PowerShellGet
   # requires -Module PackageManagement

   $SourceLocation = "<Location on the development kit that contains the PowerShell packages>"
   $RepoName = "MyNuGetSource"

   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

   Register-PSRepository -Name $RepoName -SourceLocation $SourceLocation -InstallationPolicy Trusted

   Install-Module -Name AzureStack -Repository $RepoName -RequiredVersion 2.0.2-preview -AllowPrerelease -Scope AllUsers

   Install-Module -Name Az -Repository $RepoName -RequiredVersion 0.10.0-preview -AllowPrerelease -Scope AllUsers
   ```

### Confirm the installation of PowerShell

Confirm the installation by running the following command:

```powershell
Get-Module -Name "Az*" -ListAvailable
Get-Module -Name "Azs*" -ListAvailable
```

## 6. Configure PowerShell to use a proxy server

In scenarios that require a proxy server to access the internet, you first configure PowerShell to use an existing proxy server:

1. Open an elevated PowerShell prompt.
2. Run the following commands:

   ```powershell
   #To use Windows credentials for proxy authentication
   [System.Net.WebRequest]::DefaultWebProxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials

   #Alternatively, to prompt for separate credentials that can be used for #proxy authentication
   [System.Net.WebRequest]::DefaultWebProxy.Credentials = Get-Credential
   ```

## 7. Use the Az module

You can use the cmdlets and code samples based on Azure Resource Manager. However, you will want to change the name of the modules and cmdlets. The module names have been changed so that `AzureRM` and Azure become `Az`, and the same for cmdlets. For example, the `AzureRM.Compute` module has been renamed to `Az.Compute`.` New-AzureRMVM` has become ` New-AzVM`, and `Get-AzureStorageBlob` is now `Get-AzStorageBlob`.

For a more thorough discussion and guidance for moving AzurRM script to Az and breaking changes in Azure Stack Hub's Az module, see [Migrate from AzureRM to Azure PowerShell Az](migrate-azurerm-az.md).

## Known issues

[!Include[Known issue for install - one](../includes/known-issue-az-install-1.md)]

[!Include[Known issue for install - two](../includes/known-issue-az-install-2.md)]

## Next steps

- [Download Azure Stack Hub tools from GitHub](azure-stack-powershell-download.md)
- [Configure the Azure Stack Hub user's PowerShell environment](../user/azure-stack-powershell-configure-user.md)
- [Configure the Azure Stack Hub operator's PowerShell environment](azure-stack-powershell-configure-admin.md)
- [Manage API version profiles in Azure Stack Hub](../user/azure-stack-version-profiles.md)
