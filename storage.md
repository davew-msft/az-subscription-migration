Supposedly you can do this via a [support request](https://social.msdn.microsoft.com/Forums/en-US/1edf5d31-fc0c-4eb1-b6d2-52eef2dec742/moving-a-storage-account-between-subscriptions-on-different-azure-active-directory-tenants?forum=windowsazuredata).  I don't trust that I won't break something so I copied the data from one subscription to the other.  

This process is just for "data lake" data and basic block blobs functioning similarly to a data lake.  



```bash
## Login to cloud shell in the destination subscription and ensure it's set to BASH


# change vars as needed
# this block needs to be rerun if cloudshell times out
# run `watch ls` in cloudshell to avoid the timeout and ctl+c to abort it
# there are more vars to be changed below
export DES_SUBSCRIPTION="davew-compute"
export RES_GROUP="rgDemoData"
export LOCATION="eastus2"

export TEMP_RG="temp"
export TEMP_VM="tempVM"
export TEMP_ADM="davew"
export TEMP_PWD="Password123***"


az account set --subscription $DES_SUBSCRIPTION
az group create --name $RES_GROUP --location $LOCATION

## we need a very small, basic ubuntu vm to run azcopy.  This will be deleted later
## it's possible this would work directly from cloudshell, but I'm not sure.  
az group create --name $TEMP_RG --location $LOCATION
az vm create \
    --name $TEMP_VM \
    --resource-group $TEMP_RG \
    --location $LOCATION \
    --admin-password $TEMP_PWD \
    --admin-username $TEMP_ADM \
    --image Debian \
    --public-ip-sku Standard \
    --size Standard_DS1

# copy the poublicIpAddress from the json and change this var
export IP="20.22.213.6"

ssh $TEMP_ADM@$IP


```

Run these within the VM

```bash 

# if you get disconnected, screen will allow you to reconnect by running screen -r and everything will continue
# running in the bg
# screen -r
# screen -ls
screen 

# one-time installation of az and azcopy
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
wget https://aka.ms/downloadazcopy-v10-linux
tar -xvf downloadazcopy-v10-linux
sudo rm /usr/bin/azcopy
sudo cp ./azcopy_linux_amd64_*/azcopy /usr/bin/
sudo chmod +x /usr/bin/azcopy

# vars to change
export DES_SUBSCRIPTION="davew-compute"
export SRC_SUBSCRIPTION="davew demo"
export RES_GROUP="rgDemoData"
export LOCATION="eastus2"
# I'm migrating two storage accounts
export DES_STORAGE_ACCT1="davewblob"
export DES_STORAGE_ACCT2="davewdatalake"
export SRC_STORAGE_ACCT1="davewdemoblobs"
export SRC_STORAGE_ACCT2="davewdemodata"

export SRC_STORAGE_STRING1="https://$SRC_STORAGE_ACCT1.blob.core.windows.net"
export DES_STORAGE_STRING1="https://$DES_STORAGE_ACCT1.blob.core.windows.net"
end=`date -u -d "30 days" '+%Y-%m-%dT%H:%MZ'`

az login 
#az account list -o table

# build the destination storage accounts
az account set --subscription "$DES_SUBSCRIPTION"
# my first lake
az storage account create \
    --name $DES_STORAGE_ACCT1 \
    --resource-group $RES_GROUP \
    --access-tier Premium \
    --enable-hierarchical-namespace true \
    --kind StorageV2 \
    --https-only true \
    --location $LOCATION \
    --public-network-access Enabled

# second lake
az storage account create \
    --name $DES_STORAGE_ACCT2 \
    --resource-group $RES_GROUP \
    --access-tier Premium \
    --enable-hierarchical-namespace true \
    --kind StorageV2 \
    --https-only true \
    --location $LOCATION \
    --public-network-access Enabled
    

# need to generate acct SAS for the source
az account set --subscription "$SRC_SUBSCRIPTION"
src_key=$(az storage account keys list \
    -n $SRC_STORAGE_ACCT1 \
    --subscription "$SRC_SUBSCRIPTION" \
    -g $RES_GROUP \
    --query '[0].value') 
src_sas=$(az storage account generate-sas \
    --expiry $end \
    --permissions cdlruwap \
    --resource-types sco \
    --services bfqt \
    --account-key $src_key \
    --account-name $SRC_STORAGE_ACCT1 \
    -o tsv)

# need to generate acct SAS for the destination
az account set --subscription "$DES_SUBSCRIPTION"
des_key=$(az storage account keys list \
    -n $DES_STORAGE_ACCT1 \
    --subscription "$DES_SUBSCRIPTION" \
    -g $RES_GROUP \
    --query '[0].value') 
des_sas=$(az storage account generate-sas \
    --expiry $end \
    --permissions cdlruwap \
    --resource-types sco \
    --services bfqt \
    --account-key $des_key \
    --account-name $DES_STORAGE_ACCT1 \
    -o tsv)

azcopy copy \
    "'$SRC_STORAGE_STRING1/'" \
    "'$DES_STORAGE_STRING1' " \
    --from-to BlobBlob \
    --recursive

export AZCOPY_AUTO_LOGIN_TYPE=DEVICE
azcopy login
azcopy copy \
    'https://davewdemoblobs.blob.core.windows.net/?se=2022-11-18T19%3A47Z&sp=rwdlacup&sv=2021-06-08&ss=tqbf&srt=sco&sig=iV%2BBbb7VBgIcQEuQ09uD6j9C18JMWQbfkIwXOJ4adrM%3D' \
    'https://davewblob.blob.core.windows.net?se=2022-11-18T19%3A47Z&sp=rwdlacup&sv=2021-06-08&ss=tqbf&srt=sco&sig=iV%2BBbb7VBgIcQEuQ09uD6j9C18JMWQbfkIwXOJ4adrM%3D' \
    --from-to BlobBlob \
    --recursive  


# storage_account_key = str(key[0][1:-1]) # this is used to strip opening and closing quotation marks

# need to get your tenantid
azcopy login --tenant-id e18c8112-3b45-4810-908f-163d21953506

azcopy copy \
    'https://<source-storage-account-name>.blob.core.windows.net/<SAS-token>' \
    'https://<destination-storage-account-name>.blob.core.windows.net/' 
    --recursive


https://learn.microsoft.com/en-us/azure/storage/common/storage-ref-azcopy-sync?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json
azcopy copy \
    /Source:$SRC_STORAGE_STRING1 \
    /Dest:$DES_STORAGE_STRING1 \

    --recursive=true \
    --overwrite ifSourceNewer 


grep UPLOADFAILED .\04dc9ca9-158f-7945-5933-564021086c79.log

azcopy jobs list

azcopy jobs show <job-id> --with-status=Failed


AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 
/Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /S

https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy#copy-blobs-in-blob-storage

# remove the temp VM
az group delete --name $TEMP_RG

```