# Azure Policy

Check here for a current list of [**known issues**](#known-issues).

## Alias Requests

[**Requesting Policy Aliases**](#requesting-policy-aliases)

## Samples

This repository contains samples of Azure Policies that can be used as reference for creating and assigning policies to your subscriptions and resource groups. For a full list of samples with descriptions, see [Policy samples](https://docs.microsoft.com/azure/governance/policy/samples/) on docs.microsoft.com.

### Articles

- [Azure Policy overview](https://docs.microsoft.com/azure/governance/policy/overview)
- [How to assign policies using the Azure portal](https://docs.microsoft.com/azure/governance/policy/assign-policy-portal)
- [How to assign policies using Azure PowerShell](https://docs.microsoft.com/azure/governance/policy/assign-policy-powershell)
- [How to assign policies using Azure CLI](https://docs.microsoft.com/azure/governance/policy/assign-policy-azurecli)
- [Definition structure](https://docs.microsoft.com/azure/governance/policy/concepts/definition-structure)
- [Understand Policy effects](https://docs.microsoft.com/azure/governance/policy/concepts/effects)
- [Audit VMs with Guest Configuration](https://docs.microsoft.com/azure/governance/policy/concepts/guest-configuration)
- [Programmatically create policies](https://docs.microsoft.com/azure/governance/policy/how-to/programmatically-create)
- [Get compliance data](https://docs.microsoft.com/azure/governance/policy/how-to/get-compliance-data)
- [Remediate non-compliant resources](https://docs.microsoft.com/azure/governance/policy/how-to/remediate-resources)

### References

- [Azure CLI](https://docs.microsoft.com/cli/azure/policy)
- Azure PowerShell
  - [Policy](https://docs.microsoft.com/powershell/module/az.resources/#policies)
  - [Guest Configuration (preview)](https://www.powershellgallery.com/packages/AzureRM.GuestConfiguration)
- REST API
  - [Events](https://docs.microsoft.com/rest/api/policy-insights/policyevents)
  - [States](https://docs.microsoft.com/rest/api/policy-insights/policystates)
  - [Assignments](https://docs.microsoft.com/rest/api/resources/policyassignments)
  - [Policy Definitions](https://docs.microsoft.com/rest/api/resources/policydefinitions)
  - [Initiative Definitions](https://docs.microsoft.com/rest/api/resources/policysetdefinitions)
  - [Policy Tracked Resources](https://docs.microsoft.com/rest/api/policy-insights/policytrackedresources)
  - [Remediations](https://docs.microsoft.com/rest/api/policy-insights/remediations)
  - [Guest Configuration (preview)](https://docs.microsoft.com/rest/api/guestconfiguration/)

### Other

- [Video - Build 2018](https://channel9.msdn.com/events/Build/2018/THR2030)

## Contributing

To contribute and get started, please visit our [**contribution guide**](./1-contribution-guide/README.md#contribution-guide).

## Requesting Policy Aliases

To request a new alias, please open a new issue following the instructions [**here**](./1-contribution-guide/request-alias.md)

## Known Issues

Azure Policy operates at a level above other Azure services by applying policy rules against PUT requests and GET responses of resource types going between Azure Resource Manager and the owning resource provider (RP). In a few cases, the behavior of a given RP is unexpected or incompatible in some way with Azure Policy. The Azure Policy team works with the RP teams to close these gaps as soon as possible after they are discovered. Issues of this nature will be listed here until closed. To get something added to this list that isn't already reported in [**Issues**](https://github.com/Azure/azure-policy/issues), open a [**New issue**](https://github.com/Azure/azure-policy/issues/new/choose).

All cases of known resource types with anomalous policy behavior are listed here. Currently there is no way to make these resource types invisible at policy authoring time, so writing policies that attempt to manage these resource types cannot be prevented, despite the fact that the results of such policies will be either incomplete or incorrect.

### Resource Type query results incomplete/missing

In some cases, certain RPs may return incomplete or otherwise limited or missing information about resources of a given type. The Azure Policy engine is unable to determine the compliance of any resources of such a type. Here are the known resource types with this problem.

- Microsoft<span></span>.Web/sites/siteConfig
- Microsoft<span></span>.Web/sites/config/* (except Microsoft<span></span>.Web/sites/config/web)

Currently, there is no plan to change this behavior. If this scenario is important to you, please open a support ticket with the Web team.

### Resource Type not correctly published by resource provider

In some cases, a resource provider may implement a resource type, but not correctly publish it to the Azure Resource Manager. The result of this is that Azure Policy is unable to discover the type in order to determine compliance. In some cases, this still allows deny policies to work, but compliance results will usually be incorrect. These resource types exhibit this behavior:

- Microsoft.EventHub/namespaces/networkRuleSet
- Microsoft.ServiceBus/namespaces/networkRuleSet
- Microsoft.Storage/storageAccounts/blobServices

In many of these cases the unpublished resource type is actually a subtype of a published type, which causes aliases to refer to a parent type instead of the unpublished type. Evaluation of such policies fails, causing the policy to never apply to any resource. Here are the known resource types with this problem:

- Microsoft.EventHub/namespaces/networkRuleSets
- Microsoft.ServiceBus/namespaces/networkRuleSets
- Microsoft.ApiManagement/service/portalsettings/delegation
- Microsoft.Sql/servers/databases/backupShortTermRetentionPolicies

All of these are in the process of being addressed with the various resource provider teams. We will update this notice as things change.

### Resource management that bypasses Azure Resource Manager

Resource providers are free to implement their own resource management operations outside of Azure Resource Manager ("dataplane" operations). In almost every Azure resource type, the distinction between resource management and dataplane operations is clear and the resource provider only implements resource management one way. Occasionally, a resource provider may choose to implement a type that can be managed both ways. In this case, Azure Policy controls the standard Azure Resource Manager API normally, but operations on the direct resource provider API to create, modify and delete resources of that type bypass Azure Resource Manager so they are invisible to Azure Policy. Since policy enforcement is incomplete, we recommend that customers do not implement policies targeting such a resource type. Currently there is one such known resource type:

- Microsoft.Storage/storageAccounts/blobServices/containers

The storage team is working on implementing Azure Policy on its dataplane operations to address this scenario. This is expected to first be available later this year.

### Nonstandard creation pattern

In a few instances, the creation pattern of a resource type doesn't follow normal REST patterns. In these cases, deny policies may not work or may only work for some properties. For example, certain resource types may PUT only a subset of the properties of the resource type to create the entire resource. With such types the resource could be created with a non-compliant value even though a deny policy exists to prevent it. A similar result may occur if a set of resource types can be created using a collection PUT. Known resource types that exhibit this behavior:

- Microsoft.Sql/servers/firewallRules

The SQL team is working with the Azure Resource Manager team on changes that will implement firewall rule creation using a standard PUT method. This is expected to be available later this year.

*This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.*
