# az-subscription-migration

## What is this?  

...use this as a starting point if you need to migration your stuff from one azure subscription to another.  Specifically, this is for moving from one azure tenant to another.  In my case this was migration to the FDPO tenant. 

According to the docs you can migration an entire subscription to another tenant.  I don't trust that something won't break so I recreated everything manually, usually from IaC.  

## General Notes

* FDPO sucks, use the hybrid-external option


## Table of Contents

* [Migrating storage accounts](storage.md)
* [databricks](databricks.md)
* km rg (nothing to do)
* rg MLOpsWorkshop
* rgadfWorkshop
* synapsetemp 
  * copy the synapse sandbox data



## Status of Migration

* still need 891 data

