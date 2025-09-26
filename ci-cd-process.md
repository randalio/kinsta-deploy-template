# 🛠️ CI/CD Process Documentation

This document outlines our standard **CI/CD workflow** — from setting up a local environment with **DevKinsta**, to branching and deploying changes to **Staging** and **Production** — using **Visual Studio Code (VS Code)** and its built-in terminal.

---

## 📁 1. Local Development Setup (DevKinsta + VS Code)

Use **[DevKinsta](https://kinsta.com/devkinsta/)** for local WordPress development and **VS Code** for the integrated development environment.

### 1.1. Clone the Git Repository in VS Code

1. Open **VS Code**.  
2. Use the built-in **Command Palette** (`⇧⌘P` / `Ctrl+Shift+P`) → type **“Git: Clone”** and press Enter.  
3. Paste the Git repo URL:  
   ```
   git@github.com:your-org/your-repo.git
   ```
4. Choose the destination folder for your local development site. If you already created a DevKinsta site, navigate to:  
   ```
   ~/DevKinsta/public/<site-name>/wp-content/themes/
   ```

✅ **Tip:** To avoid creating a nested folder with the repo’s name, you can clone directly into the theme directory from the integrated terminal:

- Open the **terminal** in VS Code:  
  - `View` → `Terminal` (or `` Ctrl+` ``)
- Navigate to your theme directory:
   ```bash
   cd ~/DevKinsta/public/<site-name>/wp-content/themes/
   ```
- Use a period (`.`) to clone directly into the current folder (`.`):
   ```bash
   git clone git@github.com:your-org/your-repo.git .
   ```

---

### 1.2. Moving Theme Files (if needed)

If you cloned into a subfolder by accident, you can move the files using the terminal inside VS Code:

```bash
mv your-repo/* .
mv your-repo/.* .
rm -rf your-repo
```

✅ Make sure your `style.css`, `functions.php`, etc., live directly in:

```
wp-content/themes/your-theme-name/
```

---

## 🌱 2. Branching Strategy

We follow a Git branching model to keep deployments stable and predictable.

| Branch Name | Environment         | Purpose                                |
|------------|---------------------|----------------------------------------|
| `main`     | **Production**      | Live site — always stable and deployable |
| `staging`  | **Staging**         | Preview environment — testing & QA      |
| `feature/*`| **Development**     | Individual features and bug fixes       |

---

## 🚀 3. Standard Development Workflow (in VS Code)

Follow this process whenever you build a new feature or make changes.

---

### 3.1. Create a Feature Branch

1. Open the **VS Code terminal** (`` Ctrl+` ``)  
2. Make sure you’re on the latest `main`:
   ```bash
   git checkout main
   git pull origin main
   ```
3. Create and switch to a new feature branch:
   ```bash
   git checkout -b feature/your-feature-name
   ```

4. Make changes in your code using VS Code’s editor.

5. Stage and commit your changes:
   ```bash
   git add .
   git commit -m "Add: brief description of feature"
   git push origin feature/your-feature-name
   ```

---

### 3.2. Merge Feature Branch → Staging (Deploy to Staging)

When your feature is ready and tested locally:

1. Switch to `staging`:
   ```bash
   git checkout staging
   git pull origin staging
   ```

2. Merge your feature branch:
   ```bash
   git merge feature/your-feature-name
   git push origin staging
   ```

✅ This triggers deployment to the **Staging Site**. Use this for QA, client review, and testing.

---

### 3.3. Merge Staging → Main (Deploy to Production)

After QA approval:

1. Switch to `main`:
   ```bash
   git checkout main
   git pull origin main
   ```

2. Merge staging into main:
   ```bash
   git merge staging
   git push origin main
   ```

✅ This will deploy to the **Production Site** (live).

---

## 🔁 4. Best Practices

- **Always pull before branching:**  
  ```bash
  git checkout main && git pull origin main
  ```
- **Use descriptive feature names:**  
  e.g. `feature/add-header-search` or `fix/form-validation`
- **Prefer Pull Requests:**  
  Use GitHub PRs for merging into `staging` and `main` to ensure review and CI checks.
- **Never commit directly to `main`.**
- **Clean up branches:**  
  After merging, delete feature branches:
  ```bash
  git branch -d feature/your-feature-name
  ```

---

### 📊 CI/CD Flow Overview

```
(feature/*)  -->  staging  -->  main
     |            |            |
     v            v            v
Local Dev   Staging Site   Production Site
```
