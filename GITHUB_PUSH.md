# How to push this repo to your GitHub (limserenahansol)

Do this **once** to create the repo on GitHub and push everything.

## 1. Create the repo on GitHub (in browser)

1. Go to **https://github.com/new**
2. **Repository name:** `2p-cell-extract-pipeline` (or any name you like)
3. **Description:** e.g. `2P calcium imaging cell extraction pipeline (EXTRACT + NoRMCorre + ActSort)`
4. Choose **Public**
5. **Do not** check “Add a README” or “Add .gitignore” (we already have them)
6. Click **Create repository**

## 2. Push from your PC (one-time setup + push)

Open **PowerShell** (or Command Prompt) and run:

```powershell
cd C:\Users\hsollim\behavior_task\2p-cell-extract-pipeline

git init
git add README.md PROTOCOL.md GITHUB_PUSH.md speed_dff_extract_HS_tiff.m .gitignore
git commit -m "Initial commit: 2P cell extraction pipeline with README and protocol"

git branch -M main
git remote add origin https://github.com/limserenahansol/2p-cell-extract-pipeline.git
git push -u origin main
```

- If your repo name is different, replace `2p-cell-extract-pipeline` in the `git remote add origin` URL with your repo name.
- If GitHub asks for login, use your username + **Personal Access Token** (not password):  
  **GitHub → Settings → Developer settings → Personal access tokens** → create a token with `repo` scope, then use it as the password when `git push` asks.

## 3. After the first push

- Repo URL: **https://github.com/limserenahansol/2p-cell-extract-pipeline**
- To update later:  
  `git add -A`  
  `git commit -m "Your message"`  
  `git push`
