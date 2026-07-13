# Deploying Black Label Automotive to Azure Static Web Apps

Two deployment paths — pick one. **Option A (GitHub)** is easiest and gives you free CI/CD.

---

## Prerequisites
- An Azure account (free tier is fine): https://azure.microsoft.com/free
- A GitHub account
- (Optional for CLI) Node.js installed

---

## Option A — Deploy via GitHub + Azure Portal (recommended)

### 1. Push the site to GitHub
```powershell
cd C:\Users\kamoa\_Personal\BlacklabelAutomotive
git init
git add .
git commit -m "Initial Black Label Automotive site"
gh repo create BlackLabelAutomotive --public --source=. --push
```
(Or create the repo manually on github.com and `git push`.)

### 2. Create the Static Web App in Azure
1. Go to https://portal.azure.com and search **Static Web Apps** → **Create**.
2. Fill in:
   - **Subscription**: your subscription
   - **Resource Group**: create new → `rg-blacklabel`
   - **Name**: `blacklabel-automotive`
   - **Plan type**: **Free**
   - **Region**: closest to your customers (e.g. West Europe)
   - **Source**: **GitHub** → sign in and authorise
   - **Organization / Repository / Branch**: pick your repo + `main`
   - **Build Presets**: **Custom**
   - **App location**: `/`
   - **Api location**: *(leave blank)*
   - **Output location**: *(leave blank)*
3. Click **Review + Create** → **Create**.

Azure adds a GitHub Actions workflow to your repo and deploys automatically. Every push to `main` redeploys.

### 3. Grab your URL
In the Azure portal, open the Static Web App resource → the **URL** is on the Overview page (e.g. `https://polite-sand-0123.azurestaticapps.net`).

### 4. Add a custom domain (optional)
1. In the SWA resource → **Custom domains** → **Add**.
2. Enter `www.blacklabelautomotive.co.uk`.
3. Add the CNAME record shown to your DNS provider.
4. Azure validates and issues a free SSL cert automatically.

---

## Option B — Deploy via Azure CLI (no GitHub)

```powershell
# Install tools
npm install -g @azure/static-web-apps-cli
winget install Microsoft.AzureCLI

# Login
az login

# Create resource group + SWA (skip source args for manual upload)
az group create --name rg-blacklabel --location westeurope
az staticwebapp create `
  --name blacklabel-automotive `
  --resource-group rg-blacklabel `
  --location westeurope `
  --sku Free

# Get the deployment token
$token = az staticwebapp secrets list --name blacklabel-automotive --query "properties.apiKey" -o tsv

# Deploy the site
cd C:\Users\kamoa\_Personal\BlacklabelAutomotive
swa deploy . --deployment-token $token --env production
```

The CLI prints the live URL once deployment completes.

---

## Free tier limits (Static Web Apps)
- 100 GB bandwidth / month
- 0.5 GB storage
- 2 custom domains
- Free managed SSL certificates
- Global CDN

Plenty for a business marketing site.

---

## Updating the site
- **Option A**: `git commit` and `git push` — GitHub Actions redeploys.
- **Option B**: re-run the `swa deploy` command.
