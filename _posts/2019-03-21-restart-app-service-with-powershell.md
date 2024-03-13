---
layout: post
title:  "Restarting all the Azure App Services in a resource group using PowerShell."
date:   2019-03-21 10:33:00 +1000
categories: [azure, development]
tags: [powershell, azure, development]
permalink: /blog/restarting-all-the-azure-app-services-in-a-resource-group-using-powershell/
---



Often I will need to restart the Azure App Services that I use for development and testing.
Using the portal web interface can be tedious.

A much easier was is to use powershell or the shell to loop through all the App Service in the resource group and restart them.

First you will need to be logged in using the command:

```
Connect-AzAccount
```

Then run the following script

    $resourceGroupName = 'Platform-Dev'
    
    Get-AzWebApp -ResourceGroupName $resourceGroupName | ForEach-Object {    
        #Set-AzWebApp -WebSockets $true -Name $_.Name -ResourceGroupName $resourceGroupName    
        Restart-AzWebApp -ResourceGroupName $resourceGroupName -Name $_.Name
        #Restart-AzWebAppSlot -Slot "PreProd" -ResourceGroupName $resourceGroupName -Name $_.Name
    }

This can also be used to restart the apps in deployment slots:

    Get-AzWebApp -ResourceGroupName $resourceGroupName | ForEach-Object {    
         Restart-AzWebAppSlot -Slot "PreProd" -ResourceGroupName $resourceGroupName -Name $_.Name
    }

Or to set the properties on all the app service in a resource group

    Get-AzWebApp -ResourceGroupName $resourceGroupName | ForEach-Object {    
        Set-AzWebApp -WebSockets $true -Name $_.Name -ResourceGroupName $resourceGroupName        
    }
