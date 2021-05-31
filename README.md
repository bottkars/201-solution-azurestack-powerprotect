# 201-solution-azurestack-powerprotect

This Template Deploys and Configures DEL|EMC PowerProtectr Virtual Edition onto Azurestack

Prework and Requirements:
  - extract and re-hydrate VHD´s from PPDM OVA VMDK´s
  -  Upload  VHD for Azure* to Blob in Subscription

```bash
7z e /mnt/c/Users/Karsten_Bott/Downloads/dellemc-ppdm-sw-azure-19.6.0-3.ova
for ((i =1; i <= 7; i++)); do
  qemu-img convert -f vmdk -o subformat=fixed,force_size -O vpc powerprotect-disk$i.vmdk powerprotect-disk$i.vhd
done
az storage blob upload-batch --account-name opsmanagerimage -d images --destination-path powerprotect --source /home/bottk/workspace/azurestack_source_ppdm/ --pattern "powerprotect-disk*.vhd"
```
## upload  VHD Example
```bash
export AZCOPY_DEFAULT_SERVICE_API_VERSION=2017-11-09
azcopy  copy --recursive /you/file/directory/ppdm https://your.azurestack.image.blob/container<sastoken>/ppdm-19.6.03/
```
AZ CLI Deployment Example from Git:

```azurecli-interactive
az group create --name ppdm_from_cli --location local
```

```azurecli-interactive
az deployment group validate  \
--template-uri https://raw.githubusercontent.com/bottkars/201-solution-azurestack-powerprotect/main/azuredeploy.json \
--parameters https://raw.githubusercontent.com/bottkars/201-solution-azurestack-powerprotect/main/azuredeploy.parameters.json \
--resource-group ppdm_from_cli
```

```azurecli-interactive
az group create --name ppdm_from_cli --location local
az deployment group create  \
--template-uri https://raw.githubusercontent.com/bottkars/201-solution-azurestack-powerprotect/main/azuredeploy.json \
--parameters https://raw.githubusercontent.com/bottkars/201-solution-azurestack-powerprotect/main/azuredeploy.parameters.json \
--resource-group ppdm_from_cli
```
delete

```azurecli-interactive
az group delete --name ppdm_from_cli  -y
```





AZ CLI Deployment Example local:

```azurecli-interactive
export FILE="$HOME/workspace/201-solution-azurestack-powerprotect"
az group create --name ppdm_from_cli --location local
```

```azurecli-interactive
SSH_KEYDATA=$(ssh-keygen -y -f ~/.ssh/id_rsa)
az deployment group validate  \
--template-file ${FILE}/azuredeploy.json \
--parameters ${FILE}/azuredeploy.parameters.json \
--parameters ppdmPasswordOrKey="${SSH_KEYDATA}" \
--parameters authenticationType="sshPublicKey" \
--resource-group ppdm_from_cli
```

```azurecli-interactive
az group create --name ppdm_from_cli --location local
az deployment group create  \
--template-file ${FILE}/azuredeploy.json \
--parameters ${FILE}/azuredeploy.parameters.json \
--parameters ppdmPasswordOrKey="${SSH_KEYDATA}" \
--parameters authenticationType="sshPublicKey" \
--resource-group ppdm_from_cli
```


delete

```azurecli-interactive
az group delete --name ppdm_from_cli  -y
```


## GitOps from direnv
validate
```bash
az group create --name ${AZS_RESOURCE_GROUP} \
  --location ${AZS_LOCATION}
az deployment group validate  \
--template-uri https://raw.githubusercontent.com/bottkars/201-solution-azurestack-powerprotect/main/azuredeploy.json \
--parameters https://raw.githubusercontent.com/bottkars/201-solution-azurestack-powerprotect/main/azuredeploy.parameters.json \
--parameters ppdmName=${AZS_HOSTNAME:?variable is empty} \
--parameters ppdmImageURI=${AZS_IMAGE_URI:?variable is empty} \
--parameters ppdmVersion=${AZS_IMAGE:?variable is empty} \
--parameters diagnosticsStorageAccountExistingResourceGroup=${AZS_diagnosticsStorageAccountExistingResourceGroup:?variable is empty} \
--parameters diagnosticsStorageAccountName=${AZS_diagnosticsStorageAccountName:?variable is empty} \
--parameters vnetName=${AZS_vnetName:?variable is empty} \
--parameters vnetSubnetName=${AZS_vnetSubnetName:?variable is empty} \
--resource-group ${AZS_RESOURCE_GROUP:?variable is empty}
```

deploy
```bash
az group create --name ${AZS_RESOURCE_GROUP} \
  --location ${AZS_LOCATION}
az deployment group create  \
--template-uri https://raw.githubusercontent.com/bottkars/201-solution-azurestack-powerprotect/main/azuredeploy.json \
--parameters https://raw.githubusercontent.com/bottkars/201-solution-azurestack-powerprotect/main/azuredeploy.parameters.json \
--parameters ppdmName=${AZS_HOSTNAME:?variable is empty} \
--parameters ppdmImageURI=${AZS_IMAGE_URI:?variable is empty} \
--parameters ppdmVersion=${AZS_IMAGE:?variable is empty} \
--parameters diagnosticsStorageAccountExistingResourceGroup=${AZS_diagnosticsStorageAccountExistingResourceGroup:?variable is empty} \
--parameters diagnosticsStorageAccountName=${AZS_diagnosticsStorageAccountName:?variable is empty} \
--parameters vnetName=${AZS_vnetName:?variable is empty} \
--parameters vnetSubnetName=${AZS_vnetSubnetName:?variable is empty} \
--resource-group ${AZS_RESOURCE_GROUP:?variable is empty}
```

```bash
# HAcking the Deployment

  NETWORK_GATEWAY=$(ip route | awk '/default/ { print $3 }')
  NETWORK_NETMASK=$(/sbin/ifconfig eth0 | awk -F: '/Mask:/{print $4}')
  IFS=' ' read -a ARR <<< $(grep "nameserver" /etc/resolv.conf)
  NETWORK_DNS_SERVER=${ARR[-1]}
#  NETWORK_DNS_SERVER=${ARR[1]}
#  NETWORK_FQDN=$(nslookup ${NODE1_NETWORK_IP_ADDRESS} ${NETWORK_DNS_SERVER} | grep "name =" | cut -d ' ' -f3)
  NETWORK_FQDN=$(nslookup ${HOST} ${NETWORK_DNS_SERVER} | grep "Name:")
  myarray=($NETWORK_FQDN)
  NETWORK_FQDN=${myarray[-1]}
#  NETWORK_FQDN=${NETWORK_FQDN%?}
```
