---
title: Change the performance of Azure managed disks - CLI/PowerShell
description: Learn how to change performance tiers for existing managed disks using either the Azure PowerShell module or the Azure CLI.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 06/29/2021
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell
---

# Change your performance tier using the Azure PowerShell module or the Azure CLI

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

[!INCLUDE [virtual-machines-disks-performance-tiers-intro](../../includes/virtual-machines-disks-performance-tiers-intro.md)]

## Restrictions

[!INCLUDE [virtual-machines-disks-performance-tiers-restrictions](../../includes/virtual-machines-disks-performance-tiers-restrictions.md)]

## Prerequisites
# [Azure CLI](#tab/azure-cli)
Install the latest [Azure CLI](/cli/azure/install-az-cli2) and sign in to an Azure account with [az login](/cli/azure/reference-index).

# [PowerShell](#tab/azure-powershell)
Install the latest [Azure PowerShell version](/powershell/azure/install-az-ps), and sign in to an Azure account in with `Connect-AzAccount`.

---

## Create an empty data disk with a tier higher than the baseline tier
# [Azure CLI](#tab/azure-cli)

```azurecli
subscriptionId=<yourSubscriptionIDHere>
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
diskSize=<yourDiskSizeHere>
performanceTier=<yourDesiredPerformanceTier>
region=westcentralus

az account set --subscription $subscriptionId

az disk create -n $diskName -g $resourceGroupName -l $region --sku Premium_LRS --size-gb $diskSize --tier $performanceTier
```
## Create an OS disk with a tier higher than the baseline tier from an Azure Marketplace image

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>
region=westcentralus
image=Canonical:UbuntuServer:18.04-LTS:18.04.202002180

az disk create -n $diskName -g $resourceGroupName -l $region --image-reference $image --sku Premium_LRS --tier $performanceTier
```

# [PowerShell](#tab/azure-powershell)

```azurepowershell
$subscriptionId='yourSubscriptionID'
$resourceGroupName='yourResourceGroupName'
$diskName='yourDiskName'
$diskSizeInGiB=4
$performanceTier='P50'
$sku='Premium_LRS'
$region='westcentralus'

Connect-AzAccount

Set-AzContext -Subscription $subscriptionId

$diskConfig = New-AzDiskConfig -SkuName $sku -Location $region -CreateOption Empty -DiskSizeGB $diskSizeInGiB -Tier $performanceTier
New-AzDisk -DiskName $diskName -Disk $diskConfig -ResourceGroupName $resourceGroupName
```
---

## Update the tier of a disk without downtime

# [Azure CLI](#tab/azure-cli)

### Prerequisites

You must enable the feature for your subscription before you can change the performance tier of a disk without downtime. The following steps will enable the feature on your subscription:

1.	Execute the following command to register the feature for your subscription

    ```azurecli
    az feature register --namespace Microsoft.Compute --name LiveTierChange
    ```
 
1.	Confirm that the registration state is **Registered** (may take a few minutes) using the following command before trying out the feature.

    ```azurecli
    az feature show --namespace Microsoft.Compute --name LiveTierChange
    ```

```azurecli
resourceGroupName=<yourResourceGroupNameHere>
diskName=<yourDiskNameHere>
performanceTier=<yourDesiredPerformanceTier>

az disk update -n $diskName -g $resourceGroupName --set tier=$performanceTier
```

# [PowerShell](#tab/azure-powershell)

You must enable the feature for your subscription before you can change the performance tier of a disk without downtime. The following steps will enable the feature on your subscription:

1.	Execute the following command to register the feature for your subscription

    ```azurepowershell
     Register-AzProviderFeature -FeatureName "LiveTierChange" -ProviderNamespace "Microsoft.Compute" 
    ```
 
1.	Confirm that the registration state is **Registered** (may take a few minutes) using the following command before trying out the feature.

    ```azurepowershell
    Register-AzProviderFeature -FeatureName "LiveTierChange" -ProviderNamespace "Microsoft.Compute" 
    ```

```azurepowershell
$resourceGroupName='yourResourceGroupName'
$diskName='yourDiskName'
$performanceTier='P1'

$diskUpdateConfig = New-AzDiskUpdateConfig -Tier $performanceTier

Update-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName -DiskUpdate $diskUpdateConfig
```
---

## Show the tier of a disk

# [Azure CLI](#tab/azure-cli)

```azurecli
az disk show -n $diskName -g $resourceGroupName --query [tier] -o tsv
```

# [PowerShell](#tab/azure-powershell)

```azurepowershell
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName -DiskName $diskName

$disk.Tier
```
---

## Next steps

If you need to resize a disk to take advantage of the higher performance tiers, see these articles:

- [Expand virtual hard disks on a Linux VM with the Azure CLI](linux/expand-disks.md)
- [Expand a managed disk attached to a Windows virtual machine](windows/expand-os-disk.md)
