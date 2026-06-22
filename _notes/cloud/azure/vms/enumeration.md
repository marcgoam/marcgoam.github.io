---
title: "Virtual Machines Enumeration"
order: 41
description: Enumeration of Virtual Machines
---

## VM Enumeration

### List all VMs and get info about one

```bash
az vm list --output table
az vm list --show-details --query "[].{Name:name, RG: resourceGroup, OS:storageProfile.osDisk.osType, PublicIP:publicIps, 
PrivateIP:privateIps, Status:powerState, Identity:identity.type, Size:hardwareProfile.vmSize}" --output table
az vm show --name $VM_NAME --resource-group $RESOURCE_GROUP
```

### List OS of the machine

```bash
az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --query "storageProfile.osDisk.osType" \
  -o tsv
```

### List all available VM images and get info about one

```bash
az vm image list --all --output table
```

## VM Extensions

### List all VM extensions

```bash
az vm extension image list --output table
```

### Get extensions by publisher

```bash
az vm extension image list --publisher "Site24x7" --output table
```

### List extensions in a VM

```bash
az vm extension list -g $RESOURCE_GROUP --vm-name $VM_NAME
```

### List managed identities in a VM

```bash
az vm identity show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME

az vm show \
  --name "$VM_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --query "{SystemIdentityOID:identity.principalId, UserIdentities:identity.userAssignedIdentities}" \
  --output json
```

## Disks

### List all disks and get info about one

```bash
az disk list --output table
az disk show --name <disk-name> --resource-group <rsc-group>
```

## Snapshots

### List all snapshots and get info about one

```bash
az snapshot list --output table
az snapshot show --name <name> --resource-group <rsc-group>
```

## Shared Image Galleries & Compute Galleries

### List all galleries and get info about one

```bash
az sig list --output table
az sig show --gallery-name <name> --resource-group <rsc-group>
```

### List all community galleries

```bash
az sig list-community --output table
```

### List galleries shared with me

```bash
az sig list-shared --location <location> --output table
```

### List all image definitions in a gallery and get info about one

```bash
az sig image-definition list --gallery-name <name> --resource-group <rsc-group> --output table
az sig image-definition show --gallery-image-definition <name> --gallery-name <gallery-name> --resource-group <rsc-group>
```

### List all the versions of an image definition in a gallery

```bash
az sig image-version list --gallery-image-name <image-name> --gallery-name <gallery-name> --resource-group <rsc-group --output table
```

### List all VM applications inside a gallery

```bash
az sig gallery-application list --gallery-name <gallery-name> --resource-group <res-group> --output table
```

## Images

### List all managed images in your subscription

```bash
az image list --output table
```

## Restore points

### List all restore points and get info about one

```bash
az restore-point collection list-all --output table
az restore-point collection show --collection-name <collection-name> --resource-group <rsc-group>
```

## Bastion

### list all bastions

```bash
az network bastion list -o table
```

## Network

### List VNets

```bash
az network vnet list --query "[].{name:name, location:location, addressSpace:addressSpace}"
```

### List subnets of a VNet

```bash
az network vnet subnet list --resource-group <ResourceGroupName> --vnet-name <VNetName> --query "[].{name:name, addressPrefix:addressPrefix}" -o table
```

### List public IPs

```bash
az network public-ip list --output table
```

### Get NSG rules

```bash
az network nsg rule list --nsg-name <NSGName> --resource-group <ResourceGroupName> --query "[].{name:name, priority:priority, direction:direction, access:access, protocol:protocol, sourceAddressPrefix:sourceAddressPrefix, destinationAddressPrefix:destinationAddressPrefix, sourcePortRange:sourcePortRange, destinationPortRange:destinationPortRange}" -o table
```

### Get NICs and subnets using this NSG

```bash
az network nsg show --name MyLowCostVM-nsg --resource-group Resource_Group_1 --query "{subnets: subnets, networkInterfaces: networkInterfaces}"
```

### List all Nics & get info of a single one

```bash
az network nic list --output table
az network nic show --name <name> --resource-group <rsc-group>
```

### List Azure Firewalls

```bash
az network firewall list --query "[].{name:name, location:location, subnet:subnet, publicIp:publicIp}" -o table
```

### Get network rules of a firewall

```bash
az network firewall network-rule collection list --firewall-name <FirewallName> --resource-group <ResourceGroupName> --query "[].{name:name, rules:rules}" -o table
```

### Get application rules of a firewall

```bash
az network firewall application-rule collection list --firewall-name <FirewallName> --resource-group <ResourceGroupName> --query "[].{name:name, rules:rules}" -o table
```

### Get nat rules of a firewall

```bash
az network firewall nat-rule collection list --firewall-name <FirewallName> --resource-group <ResourceGroupName> --query "[].{name:name, rules:rules}" -o table
```

### List Route Tables

```bash
az network route-table list --query "[].{name:name, resourceGroup:resourceGroup, location:location}" -o table
```

### List routes for a table

```bash
az network route-table route list --route-table-name <RouteTableName> --resource-group <ResourceGroupName> --query "[].{name:name, addressPrefix:addressPrefix, nextHopType:nextHopType, nextHopIpAddress:nextHopIpAddress}" -o table
```
