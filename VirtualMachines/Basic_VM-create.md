# CREATE A VM THROUGH AZURE CLI

#### CREATE THE RESOURCE GROUP
```bash
az group create --name basicVM --location centralindia
```

#### DYNAMIC IP and STANDARD HDD 
```bash
az vm create \
  --resource-group basicVM \
  --name YOUR_NAME \
  --image Ubuntu2404 \
  --size Standard_B1s \
  --admin-username apsit \
  --admin-password "YOUR_PASSWORD" \
  --public-ip-sku Basic \
  --storage-sku Standard_LRS \
  --location centralindia \
  --authentication-type password
```
 #### STATIC IP and STANDARD SSD 
```bash
az vm create \
  --resource-group basicVM \
  --name YOUR_NAME \
  --image Ubuntu2404 \
  --size Standard_B1s \
  --admin-username apsit \
  --admin-password "YOUR_PASSWORD" \
  --public-ip-sku Basic \
  --public-ip-address-allocation static \
  --storage-sku StandardSSD_LRS \
  --location centralindia \
  --authentication-type password
```


### CHECK THE RUNNING STATE and IP ALLOCATED
```bash
 az vm list --show-details --output table
```
OR

```bash
az vm list --show-details --query "[].{
  Name: name,
  ResourceGroup: resourceGroup,
  PowerState: powerState,
  VMSize: hardwareProfile.vmSize,
  PublicIP: publicIps,
  ImageOffer: storageProfile.imageReference.offer
}" --output table
```

