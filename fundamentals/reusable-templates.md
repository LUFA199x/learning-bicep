# Build reusable bicep templates by using parameters

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

## Avoid using parameter files for secrets

