# Login to your Azure account (opens browser)
az login

# Set your subscription (if you have multiple)
az account set --subscription "YOUR_SUBSCRIPTION_ID"

# 1. Create a Resource Group
az group create --name MyAngularAppGroup --location eastus

# 2. Create Storage Account (Must be globally unique, lowercase, numbers only)
az storage account create \
    --name myuniqueangularstorage \
    --resource-group MyAngularAppGroup \
    --location eastus \
    --sku Standard_LRS

# 3. Enable "Static Website" Hosting
# This automatically creates a '$web' container and sets up SPA routing
# This REPLACES the "CloudFront Error Pages" step—it's built-in here!
az storage blob service-properties update \
    --account-name myuniqueangularstorage \
    --static-website \
    --404-document index.html \
    --index-document index.html

# 1. Get your Storage Endpoint URL (outputs something like 'myuniqueangularstorage.z13.web.core.windows.net')
# Copy the output (without 'https://')
az storage account show \
    --name myuniqueangularstorage \
    --resource-group MyAngularAppGroup \
    --query "primaryEndpoints.web" \
    --output tsv

# 2. Create CDN Profile (Collection of endpoints)
az cdn profile create \
    --name MyAngularCDNProfile \
    --resource-group MyAngularAppGroup \
    --sku Standard_Microsoft

# 3. Create CDN Endpoint (The actual distribution)
# Replace <STORAGE_ENDPOINT_HOST> with the URL you copied in Step 1 (remove https:// and trailing /)
az cdn endpoint create \
    --name my-angular-app-endpoint \
    --resource-group MyAngularAppGroup \
    --profile-name MyAngularCDNProfile \
    --origin <STORAGE_ENDPOINT_HOST> \
    --origin-host-header <STORAGE_ENDPOINT_HOST>    

# 1. Build
ng build --configuration production

# 2. Upload to Azure (Sync)
# Note: Source is 'dist/project-name', Destination container is '$web'
az storage blob upload-batch \
    --account-name myuniqueangularstorage \
    --source dist/your-project-name \
    --destination '$web' \
    --overwrite    

# Purge everything (equivalent to --paths "/*")
az cdn endpoint purge \
    --resource-group MyAngularAppGroup \
    --profile-name MyAngularCDNProfile \
    --name my-angular-app-endpoint \
    --content-paths "/*"    