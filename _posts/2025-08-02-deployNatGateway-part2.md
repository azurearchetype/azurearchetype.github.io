---
layout: post
title: "deployNatGateway"
subtitle: "- templateDevelopment"
date: 2025-08-02
author: Matthew
series: "deployNatGateway"
series_part: 2
series_title: "Building an Azure NAT Gateway Template"
---

## Building the ‘main’ ARM Template

In this second part of the deployNatGateway series, we are going dive into the actual creation process of the ‘main’ ARM template, and what is this Bicep thing everyone talks about when building an infrastructure as code (IaC) solution.

Our flow for this step will be to take a predefined Azure Bicep resource template for a NAT Gateway, modify it to meet the requirements of our planned solution, and then convert it to JSON formatting to be deployed as a custom template.

## What is Azure Bicep?

Azure Bicep is defined as “a domain-specific language (DSL) designed to simplify the authoring and management of Azure via infrastructure-as-code (IaC). It provides a more concise and human-readable syntax compared to JSON-based Azure Resource Manager (ARM) templates, making it easier to define and deploy Azure resources like virtual machines, storage accounts, and databases.”
The syntax of JSON is not friendly as we will see. Leveraging Bicep allows you to create code in manner that is methodical and easily understandable.

### Transpilative Nature of Bicep

I love this word! Bicep is transpilative, meaning that Bicep files are compiled (or transpiled) into JSON-based ARM templates before deployment. The Bicep compiler converts the Bicep syntax into the equivalent JSON format that Azure Resource Manager understands. This process is seamless and handled by the Bicep CLI or Azure CLI/PowerShell, allowing us to write simpler code while still leveraging the full power of ARM templates.

**How It Works:** When you run a Bicep file (e.g., main.bicep), the Bicep CLI transpiles it into a JSON ARM template (e.g., main.json). This JSON file is then used by Azure Resource Manager to provision resources.

**Benefits:** The transpilation ensures compatibility with existing ARM tools and workflows, as the output is standard JSON. This allows focus on Bicep's clean syntax without worrying about JSON's verbosity.

**Use it in a sentence:** 'Sorry I am late, I was caught up ‘transpiling some Bicep code into JSON templates for deployment to our dev infrastructure environment’.

## Template Structure

This ARM template will consist of the following key components:
1. Parameters for resource customization
2. Variables to compute unique values
3. Resource definitions
4. Outputs for reference

I feel we could spend an entire post just on template structures but I wanted to mention the main components in use for this solution.  This would be a good topic for Discussions in the meantime.

I started with a basic NAT Gateway template pulled from GitHub using Copilot and modified it with some basic edits.

When I am developing new IaC code, I like to add default parameter values within the template. This helps me with debugging. If I see a default value in the deployment, I know where the value is coming from and can identify if it wasn't passed down from a parameter file or another 'parent' template.

### Sample Bicep Code

```bicep
param location string = resourceGroup().location
param vnetResourceGroup = 'myResourceGroup'
param vnetName string = 'myVnet'
param subnetName string = 'mySubnet'
param natGatewayName string = 'myNatGateway'
param publicIpName string = 'myNatPublicIP'

resource natPublicIp 'Microsoft.Network/publicIPAddresses@2023-04-01' = {
  name: publicIpName
  location: location
  sku: {
    name: 'Standard'
  }
  properties: {
    publicIPAllocationMethod: 'Static'
  }
}

resource natGateway 'Microsoft.Network/natGateways@2023-04-01' = {
  name: natGatewayName
  location: location
  sku: {
    name: 'Standard'
  }
  properties: {
    publicIpAddresses: [
      {
        id: natPublicIp.id
      }
    ]
  }
}

resource vnet 'Microsoft.Network/virtualNetworks@2023-04-01' = {
  name: vnetName
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        '10.0.0.0/16'
      ]
    }
    subnets: [
      {
        name: subnetName
        properties: {
          addressPrefix: '10.0.1.0/24'
          natGateway: {
            id: natGateway.id
          }
        }
      }
    ]
  }
}
```

## Key Resources

Our template creates the following Azure resources:
- **NAT Gateway** - The star of the show.
- **Public IP** - For the NAT Gateway's outbound connectivity.

Our template references these existing Azure resources:
- **Virtual Network** - The foundation for Azure connectivity.
- **Subnet** - Resource that our NAT Gateway will be associated with.

## Conversion to JSON

To manually convert an Azure Bicep template to JSON, you can use the Azure CLI, Bicep CLI, or PowerShell. Here's a collection of methods, they are all very similar:

### Using Azure CLI

Ensure you have the Azure CLI installed with Bicep support and run the following command:
```bash
az bicep decompile --file deployNatGateway.bicep
```

This generates a JSON file (ARM template) in the same directory with the same name but with a .json file extension.

### Using Bicep CLI

Install the Bicep CLI (included with Azure CLI or standalone) and run:
```bash
bicep decompile deployNatGateway.bicep > output.json
```

### Using PowerShell

Ensure the Bicep module is installed (Install-Module -Name Az.Bicep).
Use the ConvertTo-Json cmdlet after compiling:
```powershell
$bicepContent = bicep build <your-bicep-file>.bicep --stdout | ConvertTo-Json
$bicepContent | Out-File -FilePath output.json
```

Notes:

The generated JSON is an ARM template that can be deployed using Azure CLI, PowerShell, or the Azure Portal.

You always want to check for warnings or errors during conversion, as complex Bicep features might require manual adjustments in the JSON output.

## mainTemplate.json

Using any of these methods, you will end up with a functional ARM template like this one

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location for all resources, derived from the virtual network's location."
            }
        },
        "vnetName": {
            "type": "string",
            "metadata": {
                "description": "Name of the existing virtual network."
            }
        },
        "vnetResourceGroup": {
            "type": "string",
            "metadata": {
                "description": "Resource group of the existing virtual network."
            }
        },
        "subnetNames": {
            "type": "object",
            "metadata": {
                "description": "Object containing the selected subnet names."
            }
        },
        "natGatewayName": {
            "type": "string",
            "defaultValue": "[concat('natGateway-', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Name of the NAT Gateway."
            }
        },
        "publicIpName": {
            "type": "string",
            "defaultValue": "[concat('pip-natGateway-', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Name of the public IP address for the NAT Gateway."
            }
        }
    },
    "variables": {
        "subnet1Name": "[parameters('subnetNames').subnet1Name]",
        "subnet2Name": "[if(parameters('subnetNames').includeSubnet2, parameters('subnetNames').subnet2Name, '')]",
        "subnet3Name": "[if(parameters('subnetNames').includeSubnet3, parameters('subnetNames').subnet3Name, '')]",
        "subnet4Name": "[if(parameters('subnetNames').includeSubnet4, parameters('subnetNames').subnet4Name, '')]",
        "subnet5Name": "[if(parameters('subnetNames').includeSubnet5, parameters('subnetNames').subnet5Name, '')]"
    },
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2021-08-01",
            "name": "[parameters('publicIpName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard"
            },
            "properties": {
                "publicIPAllocationMethod": "Static"
            }
        },
        {
            "type": "Microsoft.Network/natGateways",
            "apiVersion": "2021-08-01",
            "name": "[parameters('natGatewayName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]"
            ],
            "properties": {
                "publicIpAddresses": [
                    {
                        "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]"
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2021-08-01",
            "name": "[format('{0}/{1}', parameters('vnetName'), variables('subnet1Name'))]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
            ],
            "properties": {
                "addressPrefix": "[reference(resourceId(parameters('vnetResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), variables('subnet1Name')), '2021-08-01').addressPrefix]",
                "natGateway": {
                    "id": "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
                }
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2021-08-01",
            "name": "[format('{0}/{1}', parameters('vnetName'), variables('subnet2Name'))]",
            "condition": "[parameters('subnetNames').includeSubnet2]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
            ],
            "properties": {
                "addressPrefix": "[reference(resourceId(parameters('vnetResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), variables('subnet2Name')), '2021-08-01').addressPrefix]",
                "natGateway": {
                    "id": "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
                }
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2021-08-01",
            "name": "[format('{0}/{1}', parameters('vnetName'), variables('subnet3Name'))]",
            "condition": "[parameters('subnetNames').includeSubnet3]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
            ],
            "properties": {
                "addressPrefix": "[reference(resourceId(parameters('vnetResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), variables('subnet3Name')), '2021-08-01').addressPrefix]",
                "natGateway": {
                    "id": "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
                }
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2021-08-01",
            "name": "[format('{0}/{1}', parameters('vnetName'), variables('subnet4Name'))]",
            "condition": "[parameters('subnetNames').includeSubnet4]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
            ],
            "properties": {
                "addressPrefix": "[reference(resourceId(parameters('vnetResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), variables('subnet4Name')), '2021-08-01').addressPrefix]",
                "natGateway": {
                    "id": "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
                }
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks/subnets",
            "apiVersion": "2021-08-01",
            "name": "[format('{0}/{1}', parameters('vnetName'), variables('subnet5Name'))]",
            "condition": "[parameters('subnetNames').includeSubnet5]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
            ],
            "properties": {
                "addressPrefix": "[reference(resourceId(parameters('vnetResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('vnetName'), variables('subnet5Name')), '2021-08-01').addressPrefix]",
                "natGateway": {
                    "id": "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
                }
            }
        }
    ],
    "outputs": {
        "natGatewayId": {
            "type": "string",
            "value": "[resourceId('Microsoft.Network/natGateways', parameters('natGatewayName'))]"
        },
        "publicIpAddressId": {
            "type": "string",
            "value": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]"
        }
    }
}
```

## Disclaimer

The JSON template is the actual code used for the final solution and contains some additional functions for integration with the createUiDefinition.json file for collection of user inputs. It is not a direct 'transpilation' (remember?) of the example Bicep code above. The Bicep file is a generic NAT Gateway template generated by AI and modified for this post.  I wanted to show the path from a DSL like Bicep to native JSON and how the two files differ in complexity.

## So What About AI?

Leveraging a copilot or agentic AI agent you could do this function in less than 15 minutes, but that's not the goal.  We want to come out in the end with a known repeatable and consistent process for deploying a NAT Gateway and associate it with existing Azure resources that are potentially in a production environment. Eliminating risk, the potential for error, and efficiency are key to the concept.

Vibe coding with AI in a production infrastructure environment should not be a thing, not yet anyway.

## Coming Up Next

In Part 3, we'll jump into creating a custom UI and how to create an engaging deployment experience for users through the Azure portal with a createUiDefinition template.  We will also take a look at the differences between the UI definition schemas and limitations to dynamic resource 'lookups' for existing Azure resources with 'custom template' deployments versus an 'application'.
