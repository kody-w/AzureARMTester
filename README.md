# Azure ARM Template Tester - Copilot Agent 365

## Quick Deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fkody-w%2FAzureARMTester%2Fmain%2Fazuredeploy.json)

## What This Deploys

This template deploys a complete Copilot Agent 365 infrastructure with:
- Azure Functions (Python 3.11)
- Azure OpenAI Service (GPT-4o)
- Storage Account
- Application Insights

## Cross-Region Deployment Fix

This template includes fixes for the 403 Forbidden error that affects cross-region deployments:
- Proper dependency chains
- Explicit file service creation
- Network ACL configurations
- Unique resource naming

## Deployment Instructions

### Option 1: Deploy Button (Easiest)
1. Click the "Deploy to Azure" button above
2. Fill in the parameters:
   - **Resource Group**: Select or create new
   - **Location**: Choose your preferred region (must have OpenAI quota)
   - **Storage Account Name**: Must be globally unique (3-24 chars, lowercase/numbers only)
   - Leave other parameters as default or customize

### Option 2: Azure CLI
```bash
# Clone this repo
git clone https://github.com/kody-w/AzureARMTester.git
cd AzureARMTester

# Deploy
az deployment group create \
  --resource-group "YourResourceGroup" \
  --template-file azuredeploy.json \
  --parameters storageAccountName="uniquename$(date +%s)"
```

### Option 3: PowerShell
```powershell
# Clone this repo
git clone https://github.com/kody-w/AzureARMTester.git
cd AzureARMTester

# Deploy
$timestamp = Get-Date -Format "MMddHHmm"
New-AzResourceGroupDeployment `
  -ResourceGroupName "YourResourceGroup" `
  -TemplateFile "azuredeploy.json" `
  -storageAccountName "st$timestamp"
```

## If Deployment Fails with 403 Error

This usually happens with cross-region deployments. Try:

1. **Use a unique storage account name** with timestamp:
   - Example: `stcop365$(date +%s)` or `stcop365[MMDD][HHMM]`

2. **Wait and retry**:
   - If it fails, wait 2-3 minutes and try again
   - Azure needs time to propagate storage accounts globally

3. **Try a different region**:
   - Use a region closer to your location
   - Ensure the region supports Azure OpenAI

## Available Regions

Regions that support Azure OpenAI:
- **Americas**: eastus, eastus2, northcentralus, southcentralus, westus, westus3, canadaeast
- **Europe**: westeurope, northeurope, uksouth, francecentral, germanywestcentral, swedencentral, norwayeast, switzerlandnorth
- **Asia Pacific**: australiaeast, japaneast, centralindia
- **Africa**: southafricanorth

## Post-Deployment

After successful deployment, you'll get:
1. **Function App URL**: The endpoint for your bot
2. **All Credentials**: Available in deployment outputs
3. **Setup Scripts**: Windows and Mac/Linux scripts with your actual values

### Get Your Function URL
After deployment, find your function URL in the outputs:
```bash
# Azure CLI
az deployment group show \
  --resource-group "YourResourceGroup" \
  --name "YourDeploymentName" \
  --query properties.outputs.functionUrlWithKey.value
```

### Local Development Setup
The deployment outputs include complete setup scripts for:
- **Windows**: Copy the `windowsSetupScript` output to `setup.ps1`
- **Mac/Linux**: Copy the `macLinuxSetupScript` output to `setup.sh`

These scripts automatically:
- Install Python 3.11 (if needed)
- Install all dependencies
- Configure your local environment with YOUR Azure values
- Set up the development environment

## Testing the Deployment

### Test via cURL
```bash
curl -X POST "YOUR_FUNCTION_URL" \
  -H "Content-Type: application/json" \
  -d '{"user_input": "Hello", "conversation_history": []}'
```

### Test via PowerShell
```powershell
Invoke-RestMethod -Uri "YOUR_FUNCTION_URL" `
  -Method Post `
  -Body '{"user_input": "Hello", "conversation_history": []}' `
  -ContentType "application/json"
```

## Troubleshooting

### Storage Account Issues
- Error: "Storage account already exists"
  - Solution: Use a more unique name with timestamp

### OpenAI Quota Issues
- Error: "The subscription does not have QuotaId/Feature required"
  - Solution: Choose a region where you have OpenAI quota
  - Check quota: Azure Portal → Subscriptions → Usage + quotas

### Cross-Region 403 Errors
- Error: "Creation of storage file share failed with: 'The remote server returned an error: (403) Forbidden'"
  - Solution: This template includes fixes, but if still occurs:
    1. Use a unique storage name
    2. Wait 2-3 minutes between retries
    3. Try deploying to your local region

## Support

For issues or questions:
- Check the deployment outputs for detailed error messages
- Review the Azure Activity Log for more details
- Ensure you have appropriate Azure permissions and quotas

## License

MIT