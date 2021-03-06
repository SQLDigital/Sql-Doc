---
title: "Set-PowerPivotSystemService cmdlet | Microsoft Docs"
ms.date: 05/02/2018
ms.prod: sql
ms.technology: analysis-services
ms.custom: powershell
ms.topic: reference
ms.author: owend
ms.reviewer: owend
author: minewiskan
manager: kfile
---
# Set-PowerPivotSystemService cmdlet
[!INCLUDE[ssas-appliesto-sqlas](../../includes/ssas-appliesto-sqlas.md)]
  Sets the global properties of the PowerPivotSystemService object at the farm-level.  

>[!NOTE] 
>This article may contain outdated information and examples. Use the Get-Help cmdlet for the latest.
  
 **Applies To:** SharePoint 2010 and SharePoint 2013.  
  
## Syntax  
  
```  
Set-PowerPivotSystemService [-Identity <PowerPivotMidTierServicePipeBind>] [-UpdateAssemblyInformation <switch>] [-WorkbookUpgradeOnDataRefresh <boolean>] [-DirectTCPConnections <boolean>] [-Confirm <switch>] [<CommonParameters>]  
```  
  
## Description  
 The Set-PowerPivotSystemService cmdlet updates the properties of the [!INCLUDE[ssGemini](../../includes/ssgemini-md.md)] System Service parent object in the farm.  
  
## Parameters  
  
### -Identity \<PowerPivotMidTierServicePipeBind>  
 Specifies the parent object for which you are updating properties. The value must be a valid GUID that uniquely identifies the object in the farm.  
  
|||  
|-|-|  
|Required?|false|  
|Position?|0|  
|Default value||  
|Accept pipeline input?|true|  
|Accept wildcard characters?|false|  
  
### -UpdateAssemblyInformation \<switch>  
 Used for upgrade purposes only. If the assembly version deployed in the farm is different from the version that is stored in the SharePoint configuration database, you can run this cmdlet to update the assembly information in the configuration database. Assembly version information is available in the file properties of the Microsoft.AnalysisServices.SharePoint.Integration.dll that is stored in the global assembly.  
  
|||  
|-|-|  
|Required?|false|  
|Position?|1|  
|Default value||  
|Accept pipeline input?|false|  
|Accept wildcard characters?|false|  
  
### -WorkbookUpgradeOnDataRefresh \<boolean>  
 Used to automatically upgrade a [!INCLUDE[ssKilimanjaro](../../includes/sskilimanjaro-md.md)] workbook at the start of a scheduled data refresh on the server. Data refresh is supported only for workbooks that match the current version of the server. If you enable this property, a workbook will be automatically upgraded so that data refresh can continue. This property is set at the server instance level. You cannot vary it for specific workbooks, libraries, sites, or users.  
  
|||  
|-|-|  
|Required?|false|  
|Position?|2|  
|Default value|false|  
|Accept pipeline input?|false|  
|Accept wildcard characters?|false|  
  
### -DirectTCPConnections \<boolean>  
 Specifies that Excel Services sends all queries directly to the instance of SQL Server Analysis Services (POWERPIVOT) that loaded a [!INCLUDE[ssGemini](../../includes/ssgemini-md.md)] database, bypassing the MSOLAP data provider and channel transport that are otherwise used for every query request sent to a [!INCLUDE[ssGemini](../../includes/ssgemini-md.md)] database.  
  
 Setting this parameter improves the performance and scalability of [!INCLUDE[ssGemini](../../includes/ssgemini-md.md)] queries by making the connections to loaded databases more efficiently. Note that this parameter does not change the behavior of how the initial load request is allocated. Other parameters, such as ???RoundRobinAllocation and ???HealthBasedAllocation, that are used to allocate database load requests among multiple [!INCLUDE[ssGemini](../../includes/ssgemini-md.md)] for SharePoint instance in the farm are unaffected because ???DirectTCPConnections applies only to queries that are issued after the database is loaded.  
  
 For farm topologies that include Excel Services and [!INCLUDE[ssGemini](../../includes/ssgemini-md.md)] for SharePoint running on separate application servers, you must open a port in the firewall to ensure that requests reach the SQL Server Analysis Services (POWERPIVOT) instance. For instructions on how to enable in-bound connections for a named instance of Analysis Services, see [Configure the Windows Firewall to Allow Analysis Services Access](../../analysis-services/instances/configure-the-windows-firewall-to-allow-analysis-services-access.md).  
  
|||  
|-|-|  
|Required?|false|  
|Position?|3|  
|Default value|false|  
|Accept pipeline input?|false|  
|Accept wildcard characters?|false|  
  
### -Confirm \<switch>  
 Prompts you for confirmation before executing the command. This value is enabled by default. To bypass the confirmation response in a command, specify Confirm:$false on the command.  
  
|||  
|-|-|  
|Required?|false|  
|Position?|named|  
|Default value||  
|Accept pipeline input?|false|  
|Accept wildcard characters?|false|  
  
### \<CommonParameters>  
 This cmdlet supports the common parameters: Verbose, Debug, ErrorAction, ErrorVariable, WarningAction, WarningVariable,OutBuffer and OutVariable. For more information, see [About_CommonParameters](http://go.microsoft.com/fwlink/?linkID=227825).  
  
## Inputs and Outputs  
 The input type is the type of the objects that you can pipe to the cmdlet. The return type is the type of the objects that the cmdlet returns.  
  
|||  
|-|-|  
|Inputs|None.|  
|Outputs|None.|  
  
## Example 1  
  
```  
C:\PS>Set-PowerPivotSystemService -WorkbookUpgradeOnDataRefresh:$true  
```  
  
 Enables automatic upgrade of previous version workbooks so that scheduled data refresh can proceed.  
  
  
