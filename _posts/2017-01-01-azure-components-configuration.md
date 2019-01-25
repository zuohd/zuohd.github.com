---
layout: post
title:  "云服务组件配置"
date:   2017-01-01 19:37:48 +0800
categories: technology
tags: [Azure service fabric,arm]

---

*Note：打开模板文件请使用 [JSON对象查看器](http://www.jsoneditoronline.org/)*
***
#### 1. Set up local cluster
通过WebPI安装 Microsoft Azure Service Fabric SDK and Tools(VS2015) 即可，[参照链接](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-get-started/)

#### 2. Set up cluster on azure with arm template

- 通过WebPI安装 Microsoft Azure powershell ,[参照链接](https://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/)

- 打开 PowerShell 运行命令连接 Azure
``` powershell
Login-AzureRmAccount
```

- 查看云上所有订阅可输入命令并设置订阅
``` powershell
Get-AzureRmSubscription

Set-AzureRmContext -SubscriptionId <guid>
```

- 创建资源组
``` powershell
AzureRmResourceGroup -Name insclearcluster-keyvault -Location'West US'
```

- 创建 key vault
``` powershell
New-AzureRmKeyVault -VaultName 'insclearvault' -ResourceGroupName 'insclearcluster-keyvault' -Location 'West US' -EnabledForDeployment
```

- 为 key vault 添加证书,
download helper [链接](https://github.com/ChackDan/Service-Fabric/tree/master/Scripts/ServiceFabricRPHelpers),此处的第二步可以用CreateSelfSignedCertificate参数
``` powershell
  PS C:\Users\vturecek> Import-Module "C:\users\vturecek\Documents\ServiceFabricRPHelpers\ServiceFabricRPHelpers.psm1"

 Invoke-AddCertToKeyVault -SubscriptionId <guid> -ResourceGroupName mycluster-keyvault -Location "West US" -VaultName myvault -CertificateName mycert -Password "<password>" -UseExistingCertificate -ExistingPfxFilePath "C:\path\to\mycertkey.pfx"
 
 # 如果不存在证书则使用如下命令
 Invoke-AddCertToKeyVault -SubscriptionId <guid> -ResourceGroupName mycluster-keyvault -Location "West US" -VaultName myvault -CertificateName mycert -Password "<password>" -CreateSelfSignedCertificate -DnsName "<tempdomain>" -OutputPath "C:\Users\Administrator\mycertkey.pfx"
```

- 配置云AD为客户端认证，[下载脚本文件](http://servicefabricsdkstorage.blob.core.windows.net/publicrelease/MicrosoftAzureServiceFabric-AADHelpers.zip)
``` powershell
.\SetupApplications.ps1 -TenantId '690ec069-8200-4068-9d01-5aaf188e557a' -ClusterName 'mycluster' -WebApplicationReplyUrl 'https://mycluster.westus.cloudapp.azure.com:19080/Explorer/index.html'
```

- 创建ARM模板，[下载链接](https://github.com/Azure/azure-quickstart-templates/blob/master/service-fabric-secure-cluster-5-node-1-nodetype-wad/),用前面命令输出的值编辑填充azuredeploy.parameters.json

- 测试ARM模板参数文件
``` powershell
Test-AzureRmResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json
```

- 执行部署ARM Template命令
``` powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName "myresourcegroup" -TemplateFile .\azuredeploy.json -TemplateParameterFile .\azuredeploy.parameters.json
```

- 使用云门户Portal为用户分配Admin或只读角色,[参照链接](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-cluster-creation-via-arm/#assign-users-to-roles)

#### 3. Setup log diagnostics in Azure for Service Fabric cluster

- 修改创建cluster的模板template.json文件(或通过portal下载)，把下面内容加到resources的节 

``` JsonPart
{
  "apiVersion": "2015-05-01-preview",
  "type": "Microsoft.Storage/storageAccounts",
  "name": "[parameters('applicationDiagnosticsStorageAccountName')]",
  "location": "[parameters('computeLocation')]",
  "properties": {
    "accountType": "[parameters('applicationDiagnosticsStorageAccountType')]"
  },
  "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
  }
},
```

- 添加参数到模板的parameters节(storageAccountType定义之后,貌似模板里已经有applicationDiagnosticsStorageAccountType这个内容咯)

``` JsonPart
    "applicationDiagnosticsStorageAccountType": {
      "type": "string",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS"
      ],
      "defaultValue": "Standard_LRS",
      "metadata": {
        "description": "Replication option for the application diagnostics storage account"
      }
    },
    "applicationDiagnosticsStorageAccountName": {
      "type": "string",
      "defaultValue": "storage account name goes here",
      "metadata": {
        "description": "Name for the storage account that contains application diagnostics data from the cluster"
      }
    },
```

- 在extensions数组下添加下面的内容更新VirtualMachineProfile节（**JSON对象查看器中搜索**'extensions'）

```JsonPart
{
    "name": "[concat(parameters('vmNodeType0Name'),'_Microsoft.Insights.VMDiagnosticsSettings')]",
    "properties": {
        "type": "IaaSDiagnostics",
        "autoUpgradeMinorVersion": true,
        "protectedSettings": {
        "storageAccountName": "[parameters('applicationDiagnosticsStorageAccountName')]",
        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
        "storageAccountEndPoint": "https://core.windows.net/"
        },
        "publisher": "Microsoft.Azure.Diagnostics",
        "settings": {
        "WadCfg": {
            "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": "50000",
            "EtwProviders": {
                "EtwEventSourceProviderConfiguration": [
                {
                    "provider": "Microsoft-ServiceFabric-Actors",
                    "scheduledTransferKeywordFilter": "1",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableActorEventTable"
                    }
                },
                {
                    "provider": "Microsoft-ServiceFabric-Services",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableServiceEventTable"
                    }
                }
                ],
                "EtwManifestProviderConfiguration": [
                {
                    "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                    "scheduledTransferLogLevelFilter": "Information",
                    "scheduledTransferKeywordFilter": "4611686018427387904",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricSystemEventTable"
                    }
                }
                ]
            }
            }
        },
        "StorageAccount": "[parameters('applicationDiagnosticsStorageAccountName')]"
        },
        "typeHandlerVersion": "1.5"
    }
}
```
- 重新发布模板，如果模板是从portal导出来的，运行deploy.ps1，部署之后确保*ProvisioningState*为**Succeeded**.[操作指南](https://azure.microsoft.com/en-us/documentation/articles/resource-group-template-deploy/)
- 使用log analityics 查看Service fabric 日志，[参照 Adding an existing storage account to Log Analytics 部分](https://azure.microsoft.com/en-us/documentation/articles/log-analytics-service-fabric-azure-resource-manager/#adding-an-existing-storage-account-to-log-analytics)

> 参考来源:https://azure.microsoft.com/en-us/documentation/articles/service-fabric-diagnostics-how-to-setup-wad/


