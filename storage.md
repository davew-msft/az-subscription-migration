Supposedly you can do this via a [support request](https://social.msdn.microsoft.com/Forums/en-US/1edf5d31-fc0c-4eb1-b6d2-52eef2dec742/moving-a-storage-account-between-subscriptions-on-different-azure-active-directory-tenants?forum=windowsazuredata).  I don't trust that I won't break something so I copied the data from one subscription to the other.  

This process is just for "data lake" data and basic block blobs functioning similarly to a data lake.  



```bash
## Login to cloud shell in the destination subscription and ensure it's set to BASH


# change vars as needed
# this block needs to be rerun if cloudshell times out
# run `watch ls` in cloudshell to avoid the timeout and ctl+c to abort it
export DES_SUBSCRIPTION="davew data"
export RES_GROUP="rgDemoData"
export LOCATION="eastus2"
export DES_STORAGE_ACCT="davewdata"
#export DES_STORAGE_ACCT="davewlake"
export SRC_STORAGE_ACCT=""
#export SRC_STORAGE_ACCT=""
export TEMP_RG="temp"
export TEMP_VM="tempVM"
export TEMP_ADM="admin"
export TEMP_PWD="Password123!!!"


az account set --subscription $DES_SUBSCRIPTION
az group create --name $RES_GROUP --location $LOCATION

## we need a very small, basic ubuntu vm to run azcopy.  This will be deleted later
az vm create \
    --name $TEMP_VM \
    --resource-group $TEMP_RG \
    --admin-password $TEMP_PWD \
    --admin-username $TEMP_ADM \
    --image Debian \
    


azcopy copy 'https://<source-storage-account-name>.blob.core.windows.net/' 'https://<destination-storage-account-name>.blob.core.windows.net/' --recursive

--overwrite flag to ifSourceNewer

grep UPLOADFAILED .\04dc9ca9-158f-7945-5933-564021086c79.log

azcopy jobs list

azcopy jobs show <job-id> --with-status=Failed


AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 
/Dest:https://destaccount.blob.core.windows.net/mycontainer2 /SourceKey:key1 /DestKey:key2 /S

https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy#copy-blobs-in-blob-storage

# remove the temp VM
az group delete --name $RG

```