# Build reusable bicep templates by using parameters

Bicep files made during this study session can be found [here](../bicep-files/fundamentals/build-reusable-templates)

## Parameter precedence

![Bicep Parameter precedence](https://docs.microsoft.com/en-us/learn/modules/build-reusable-bicep-templates-parameters/media/4-precedence.png)

Command-line parameters values override parameter files, parameter files override default values. 

## @secure decorator

The best approach is to avoid using credentials and use something like [Managed identities for Azure resources](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview). However this isn't available for some resources.

When you define a parameter as @secure, Azure won't make the parameter values available in the deployment logs. The values won't display in the terminal either. 

```sh
@secure()
param sqlServerAdministratorLogin string

@secure()
param sqlServerAdministratorPassword string
```

Avoid using parameter files for secrets Parameter files are often saved to a centralized version control system, avoid using them for secrets. 

## Integrate with Azure Key Vault

Avoid using parameter files for secrets Parameter files are often saved to a centralized version control system, avoid using them for secrets. Instead use KeyVault.

You can refer to secrets in key vaults that are located in a different resource group or subscription from the one you're deploying to.

![keyvault bicep integration](https://docs.microsoft.com/en-us/learn/modules/build-reusable-bicep-templates-parameters/media/5-parameter-file-key-vault.png)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "sqlServerAdministratorLogin": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/f0750bbe-ea75-4ae5-b24d-a92ca601da2c/resourceGroups/PlatformResources/providers/Microsoft.KeyVault/vaults/toysecrets"
        },
        "secretName": "sqlAdminLogin"
      }
    },
    "sqlServerAdministratorPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/f0750bbe-ea75-4ae5-b24d-a92ca601da2c/resourceGroups/PlatformResources/providers/Microsoft.KeyVault/vaults/toysecrets"
        },
        "secretName": "sqlAdminLoginPassword"
      }
    }
  }
}
```
## Use Key Vault with modules

Modules enable you to create reusable Bicep files that encapsulate a set of resources. Modules may have parameters that accept secret values, and you can use Bicep's Key Vault integration to provide these values securely. 

```sh
resource keyVault 'Microsoft.KeyVault/vaults@2021-04-01-preview' existing = {
  name: keyVaultName
}

module applicationModule 'application.bicep' = {
  name: 'application-module'
  params: {
    apiKey: keyVault.getSecret('ApiKey')
  }
}
```
- `existing` keyword. This tells Bicep that the Key Vault already exists, Bicep won't redeploy
- `getSecret()` Bicep function that can only be used with secure module parameters. Bicep translates this to the same kind of Key Vault reference in json.
