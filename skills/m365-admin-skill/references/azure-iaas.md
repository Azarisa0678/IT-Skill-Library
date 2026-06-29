# Azure IaaS & VMs

## VM-Deployment

### VM-Sizing Empfehlungen

| Workload | Serie | Beispiel SKU | Bemerkung |
|---------|-------|-------------|-----------|
| Web/App Server | B-Serie (Burstable) | B2s, B4ms | Kostenoptimiert für variable Last |
| Datenbankserver | E-Serie (Memory-optimized) | E4s_v5 | Hoher RAM-Bedarf |
| Anwendungsserver | D-Serie (General Purpose) | D4s_v5 | Ausgewogen |
| High Performance | F-Serie (Compute-optimized) | F8s_v2 | CPU-intensive Workloads |
| Domain Controller | B2s oder D2s | — | Kleine, stabile Last |

```powershell
# Azure PowerShell / Az-Modul
Connect-AzAccount -TenantId "your-tenant-id"
Set-AzContext -SubscriptionId "your-subscription-id"

# VM erstellen (sicher konfiguriert)
$resourceGroup = "rg-produktion-we"
$location      = "westeurope"
$vmName        = "srv-app-01"

# Resource Group
New-AzResourceGroup -Name $resourceGroup -Location $location

# VNet und Subnetz
$vnet = New-AzVirtualNetwork `
    -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name "vnet-produktion" `
    -AddressPrefix "10.10.0.0/16"

$subnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "snet-app" `
    -AddressPrefix "10.10.1.0/24" `
    -VirtualNetwork $vnet
$vnet | Set-AzVirtualNetwork

# NSG mit restriktiven Regeln
$nsg = New-AzNetworkSecurityGroup `
    -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name "nsg-app"

# Keine öffentliche IP für die VM!
$nic = New-AzNetworkInterface `
    -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name "nic-$vmName" `
    -SubnetId $vnet.Subnets[0].Id `
    -NetworkSecurityGroupId $nsg.Id

# VM konfigurieren
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize "Standard_B2s"
$vmConfig = Set-AzVMOperatingSystem -VM $vmConfig `
    -Windows `
    -ComputerName $vmName `
    -Credential (Get-Credential -Message "Lokaler Admin") `
    -EnableAutoUpdate

$vmConfig = Set-AzVMSourceImage -VM $vmConfig `
    -PublisherName "MicrosoftWindowsServer" `
    -Offer "WindowsServer" `
    -Skus "2022-datacenter-azure-edition" `
    -Version "latest"

$vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id

# Managed Disk mit Verschlüsselung
$vmConfig = Set-AzVMOSDisk -VM $vmConfig `
    -Name "disk-$vmName-os" `
    -CreateOption FromImage `
    -StorageAccountType "Premium_LRS" `
    -DiskEncryptionSetId "/subscriptions/.../diskEncryptionSets/myDES"

New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```

---

## Azure Bastion (sicherer RDP/SSH-Zugriff)

```powershell
# Azure Bastion als Ersatz für Public IP + RDP
# Kein RDP/SSH direkt aus dem Internet!

# Bastion Subnet erstellen (muss "AzureBastionSubnet" heißen)
$bastionSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "AzureBastionSubnet" `
    -AddressPrefix "10.10.255.0/27" `
    -VirtualNetwork $vnet
$vnet | Set-AzVirtualNetwork

# Public IP für Bastion
$bastionIp = New-AzPublicIpAddress `
    -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name "pip-bastion" `
    -AllocationMethod Static `
    -Sku Standard

# Bastion Host
New-AzBastion `
    -ResourceGroupName $resourceGroup `
    -Name "bastion-produktion" `
    -PublicIpAddressId $bastionIp.Id `
    -VirtualNetworkId $vnet.Id `
    -Sku "Standard"

# Just-in-Time VM Access aktivieren (Defender for Cloud erforderlich)
# Über Azure Portal: Defender for Cloud → JIT VM Access → aktivieren
```

---

## VM-Management & Monitoring

```powershell
# Azure Monitor Agent auf VMs installieren
$vms = Get-AzVM -ResourceGroupName $resourceGroup
foreach ($vm in $vms) {
    Set-AzVMExtension `
        -ResourceGroupName $resourceGroup `
        -VMName $vm.Name `
        -Name "AzureMonitorWindowsAgent" `
        -Publisher "Microsoft.Azure.Monitor" `
        -ExtensionType "AzureMonitorWindowsAgent" `
        -TypeHandlerVersion "1.0" `
        -Location $location
}

# Update Management: alle VMs für automatische Patches
# Über Azure Automation / Update Manager (neue Lösung)
New-AzMaintenanceConfiguration `
    -ResourceGroupName $resourceGroup `
    -Name "mc-monthly-patches" `
    -MaintenanceScope "InGuestPatch" `
    -Location $location `
    -StartDateTime "2024-01-10 02:00" `
    -TimeZone "W. Europe Standard Time" `
    -Duration "03:55" `
    -RecurEvery "1Month Third Tuesday"

# Auto-Shutdown für Non-Prod VMs (Kosteneinsparung)
$vmsNonProd = Get-AzVM -ResourceGroupName "rg-entwicklung-we"
foreach ($vm in $vmsNonProd) {
    $properties = @{
        status              = "Enabled"
        taskType            = "ComputeVmShutdownTask"
        dailyRecurrence     = @{ time = "1900" }
        timeZoneId          = "W. Europe Standard Time"
        notificationSettings = @{ status = "Disabled" }
    }
    Set-AzResource `
        -ResourceId "/subscriptions/.../virtualMachines/$($vm.Name)/providers/microsoft.devtestlab/schedules/shutdown-computevm-$($vm.Name)" `
        -Properties $properties -Force
}

# Alle VMs mit Kostenzuordnung exportieren
Get-AzVM -Status | Select-Object `
    Name, ResourceGroupName, Location, PowerState,
    @{N='SKU';E={$_.HardwareProfile.VmSize}},
    @{N='OS';E={$_.StorageProfile.OsDisk.OsType}} |
    Export-Csv "C:\Reports\AzureVMs.csv" -NoTypeInformation -Encoding UTF8
```

---

## Azure Networking

```powershell
# NSG-Regeln für Webserver (Beispiel)
$nsgRules = @(
    @{
        Name                     = "Allow-HTTPS-Inbound"
        Priority                 = 100
        Direction                = "Inbound"
        Access                   = "Allow"
        Protocol                 = "Tcp"
        SourceAddressPrefix      = "Internet"
        SourcePortRange          = "*"
        DestinationAddressPrefix = "*"
        DestinationPortRange     = "443"
    },
    @{
        Name                     = "Deny-All-Inbound"
        Priority                 = 4096
        Direction                = "Inbound"
        Access                   = "Deny"
        Protocol                 = "*"
        SourceAddressPrefix      = "*"
        SourcePortRange          = "*"
        DestinationAddressPrefix = "*"
        DestinationPortRange     = "*"
    }
)

foreach ($rule in $nsgRules) {
    Add-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg @rule | Out-Null
}
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

# VNet Peering (Verbindung zwischen VNets)
Add-AzVirtualNetworkPeering `
    -Name "peer-produktion-zu-hub" `
    -VirtualNetwork $vnet `
    -RemoteVirtualNetworkId "/subscriptions/.../virtualNetworks/vnet-hub" `
    -AllowForwardedTraffic `
    -AllowGatewayTransit:$false `
    -UseRemoteGateways:$false
```

---

## Azure IaaS Checkliste

- [ ] Keine Public IPs auf VMs — Zugriff über Azure Bastion
- [ ] JIT VM Access in Defender for Cloud aktiviert
- [ ] Alle Managed Disks verschlüsselt (Platform-managed oder Customer-managed Keys)
- [ ] Azure Monitor Agent auf allen VMs
- [ ] Update Manager konfiguriert (Maintenance Configurations)
- [ ] Auto-Shutdown für alle Non-Prod VMs
- [ ] NSG auf jedem Subnetz UND jeder NIC
- [ ] Azure Policy: Allowed Locations, VM SKUs, erforderliche Tags
- [ ] Backup über Azure Backup für alle Produktions-VMs
- [ ] Defender for Cloud: Standard-Tier für alle VMs
- [ ] Azure Cost Management: Budget-Alerts konfiguriert
