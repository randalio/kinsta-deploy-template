# 🚀 Full Instructions for GitHub → Kinsta Deployment

### 1️⃣ Generate an SSH Key Pair (if you don’t have one already)
On your local machine (Mac/Linux):
```bash
ssh-keygen -t rsa -b 4096 -C "github-kinsta-deploy"
```
- Save it somewhere safe (e.g. `~/.ssh/kinsta_github`)
- You’ll get two files:
  - `kinsta_github` → **private key**  
  - `kinsta_github.pub` → **public key**

---

### 2️⃣ Add Public Key to Kinsta Server
SSH into your Kinsta environment using the credentials they gave you (password or existing key):
```bash
ssh -p 19486 uavionix2025@35.224.70.159
```

Then add your **public key**:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "PASTE_CONTENTS_OF_kinsta_github.pub" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Repeat the same steps for **live** (port `26545`).

---

### 3️⃣ Add Private Key to GitHub
Go to your repo → **Settings → Secrets and variables → Actions → New repository secret**.  

Add a secret called:  
- `SSH_PRIVATE_KEY` → paste the **entire private key** (`kinsta_github`)  

⚠️ Ensure it starts with `-----BEGIN OPENSSH PRIVATE KEY-----` and ends with `-----END OPENSSH PRIVATE KEY-----`.  

---

### 4️⃣ Add Kinsta Connection Details to GitHub Secrets
Add these secrets in the same place:

| Secret Name             | Value                                                                 |
|--------------------------|-----------------------------------------------------------------------|
| `KINSTA_STAGING_IP`      | `00.000.00.000`                                                      |
| `KINSTA_STAGING_USER`    | `username12345`                                                       |
| `KINSTA_STAGING_PORT`    | `19486`                                                              |
| `KINSTA_STAGING_PATH`    | `/www/SITE_FOLDER/public/wp-content/themes/uavionix-theme`       |
| `KINSTA_LIVE_IP`         | `00.000.00.000`                                                        |
| `KINSTA_LIVE_USER`       | `username12345`                                                       |
| `KINSTA_LIVE_PORT`       | `26545`                                                              |
| `KINSTA_LIVE_PATH`       | `/www/SITE_FOLDER/public/wp-content/themes/uavionix-theme`       |

---

### 5️⃣ Add Workflow File to Repository
Create a file:  
`.github/workflows/deploy.yml`

```yaml
name: Deploy WordPress Theme to Kinsta

on:
  push:
    branches:
      - staging
      - main

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy Theme to Staging
        run: |
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > private_key
          chmod 600 private_key

          rsync -az --delete             -e "ssh -i private_key -p ${{ secrets.KINSTA_STAGING_PORT }} -o StrictHostKeyChecking=no"             ./ ${{ secrets.KINSTA_STAGING_USER }}@${{ secrets.KINSTA_STAGING_IP }}:${{ secrets.KINSTA_STAGING_PATH }}/

          rm private_key

  deploy-live:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy Theme to Live
        run: |
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > private_key
          chmod 600 private_key

          rsync -az --delete             -e "ssh -i private_key -p ${{ secrets.KINSTA_LIVE_PORT }} -o StrictHostKeyChecking=no"             ./ ${{ secrets.KINSTA_LIVE_USER }}@${{ secrets.KINSTA_LIVE_IP }}:${{ secrets.KINSTA_LIVE_PATH }}/

          rm private_key
```

---

### 6️⃣ Test the Connection Locally
Before letting GitHub Actions run, test your key locally:
```bash
ssh -i ~/.ssh/kinsta_github -p 19486 uavionix2025@35.224.70.159
```
- If it connects without asking for a password → ✅ keys are correct.  
Do the same for live (`-p 26545`).  

---

### 7️⃣ Trigger a Deployment
- Push to **`staging` branch** → deploys to Kinsta staging  
- Push to **`main` branch** → deploys to Kinsta live  

You’ll see the workflow run in **GitHub → Actions** tab.  

---

### 8️⃣ Verify Files on Server
After a deploy, SSH into Kinsta:
```bash
ssh -p 19486 uavionix2025@35.224.70.159
cd /www/uavionix2025_383/public/wp-content/themes/uavionix-theme
ls -la
```
You should see your `style.css`, `functions.php`, etc. directly in that folder (not nested in another `uavionix-theme` folder).  

---

✅ Done! You now have **zero-touch deployments**: just push code, and GitHub sends your theme to the right Kinsta environment.  
