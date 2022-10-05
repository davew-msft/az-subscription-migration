Supposedly you can do this via a [support request](https://social.msdn.microsoft.com/Forums/en-US/1edf5d31-fc0c-4eb1-b6d2-52eef2dec742/moving-a-storage-account-between-subscriptions-on-different-azure-active-directory-tenants?forum=windowsazuredata).  I don't trust that I won't break something so I copied the data from one subscription to the other.  

This process is just for "data lake" data and basic block blobs functioning similarly to a data lake.  



```bash
## Login to cloud shell in the destination subscription and ensure it's set to BASH


# change vars as needed
# this block needs to be rerun if cloudshell times out
# run `watch ls` in cloudshell to avoid the timeout and ctl+c to abort it
export DES_SUBSCRIPTION="davew-data"
export RES_GROUP="rgDemoData"
export LOCATION="eastus2"
# I'm migrating two storage accounts
export DES_STORAGE_ACCT1="davewdata"
export DES_STORAGE_ACCT2="davewlake"
export SRC_STORAGE_ACCT1="davewdemoblobs"
export SRC_STORAGE_ACCT2="davewdemodata"

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
export IP="20.88.112.169"

ssh $TEMP_ADM@$IP
# if you get disconnected, screen will allow you to reconnect by running screen -r and everything will continue
# running in the bg
screen 

wget https://aka.ms/downloadazcopy-v10-linux
tar -xvf downloadazcopy-v10-linux
sudo rm /usr/bin/azcopy
sudo cp ./azcopy_linux_amd64_*/azcopy /usr/bin/
sudo chmod +x /usr/bin/azcopy

azcopy copy \
    'https://<source-storage-account-name>.blob.core.windows.net/' \
    'https://<destination-storage-account-name>.blob.core.windows.net/' \
    --recursive \
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