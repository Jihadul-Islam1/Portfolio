# Deploying to a Subdomain on cPanel (e.g. `jihad.olivosoft.com`)

This is the exact workflow to take your Vite/React project and serve it on a **cPanel subdomain**.

---

## 1. Make sure the subdomain exists in cPanel

1. Login to cPanel (e.g. `https://olivosoft.com:2083`).
2. Go to **Domains → Domains** (or **Subdomains** in older cPanel).
3. You should already see:
   - Subdomain: `jihad`
   - Domain: `olivosoft.com`
   - Document Root: `public_html/jihad` (this is the folder you'll upload to)
4. If you didn't add it yet, click **Create a New Domain** / **Subdomain**:
   - Subdomain: `jihad`
   - Document Root: `public_html/jihad`  ← important!
   - Click **Submit**.

> The subdomain's document root MUST be `public_html/jihad`. If you typed a different path, the URL won't load — fix it now.

---

## 2. Build the production bundle

On your local machine:

```bash
cd d:\portfolio
npm install          # only first time
npm run build
```

This produces `d:\portfolio\dist\` containing `index.html` and `assets/`.

> Important: for a subdomain deployment you may want a **relative base path** so the site works whether it's hosted at `/` or `/subfolder/`. Open `vite.config.ts` and make sure `base` is set:

```ts
export default defineConfig({
  base: './',        // ← relative paths (works at root AND subdomain)
  // ...rest
});
```

If `base` was `'/'`, the build will still work on a subdomain (since the subdomain serves from `/`), but relative is safer for previews.

---

## 3. Upload the build

### Option A — File Manager (easiest)

1. cPanel → **File Manager** → open `public_html/jihad/` (create it if missing).
2. Click **Upload** → upload **everything inside `dist/`** (NOT the `dist/` folder itself):
   - `index.html`
   - `assets/` folder
3. After upload, your structure should be:

   ```
   public_html/jihad/
   ├── .htaccess     ← (next step)
   ├── index.html
   └── assets/
       └── ...
   ```

### Option B — FTP with FileZilla

- Host: `ftp.olivosoft.com` (or your cPanel host)
- User: a cPanel FTP account
- Remote path: `/public_html/jihad/`
- Drag contents of `dist/` into that folder.

---

## 4. Add `.htaccess` for SPA routing

Inside `public_html/jihad/`, create `.htaccess` with:

```apache
RewriteEngine On

# If request is not a real file or folder, serve index.html
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^.*$ /index.html [L]

# Optional: cache static assets for a month
<IfModule mod_expires.c>
  ExpiresActive On
  ExpiresByType text/css "access plus 1 month"
  ExpiresByType application/javascript "access plus 1 month"
  ExpiresByType image/svg+xml "access plus 1 month"
  ExpiresByType image/png "access plus 1 month"
  ExpiresByType image/jpg "access plus 1 month"
  ExpiresByType image/jpeg "access plus 1 month"
  ExpiresByType image/webp "access plus 1 month"
</IfModule>
```

In File Manager, click **Settings** → enable **Show Hidden Files (dotfiles)** so you can see/edit `.htaccess`.

---

## 5. Enable HTTPS for the subdomain

1. cPanel → **SSL/TLS Status**.
2. Find `jihad.olivosoft.com` in the list → click **Install** (free Let's Encrypt).
3. After install: **Domains → Redirects** → add an HTTPS redirect from `http://jihad.olivosoft.com` to `https://jihad.olivosoft.com`.

---

## 6. DNS sanity check (only if it doesn't load)

Most shared hosts auto-create DNS for new subdomains, so nothing to do. If `jihad.olivosoft.com` doesn't resolve:

1. cPanel → **Zone Editor** → pick `olivosoft.com`.
2. Look for a `CNAME` or `A` record with name `jihad`.
3. If missing, add:
   - Type: `CNAME`
   - Name: `jihad`
   - Record: `olivosoft.com.`
   - TTL: `14400`

DNS propagation can take **5–30 minutes**, rarely up to 24 hours.

---

## 7. Verify

1. Visit **https://jihad.olivosoft.com** — should show your portfolio.
2. Refresh on `/about` (or any deep link) — should NOT 404.
3. Open DevTools → Network → confirm no 404 on `assets/*.svg` or `assets/*.js`.

---

## Common gotchas

| Issue | Fix |
|---|---|
| Subdomain shows "Default cPanel page" | You uploaded to the wrong folder. The doc root must be `public_html/jihad`. |
| 404 on `/about` refresh | Add the `.htaccess` rewrite rule in `public_html/jihad/`. |
| Assets 404 (`/assets/...svg`) | You uploaded the `dist/` folder itself. Upload its **contents** so `index.html` sits directly under `public_html/jihad/`. |
| HTTPS not available for subdomain | Some hosts only auto-issue SSL for the main domain. Install it manually in **SSL/TLS Status**. |
| Browser still shows old version | Hard reload (Ctrl+Shift+R) or open incognito. Ask host to flush Varnish if enabled. |

---

## Optional: one-click deploy script (Windows)

Save as `deploy.ps1` in the project root:

```powershell
# build the bundle
npm run build

# create a zip you can drag-drop into cPanel File Manager
$dist = ".\dist"
if (Test-Path "$dist\dist.zip") { Remove-Item "$dist\dist.zip" -Force }
Add-Type -AssemblyName "System.IO.Compression.FileSystem"
[System.IO.Compression.ZipFile]::CreateFromDirectory($dist, "$dist\dist.zip")
Write-Host "Built: $dist\dist.zip  — upload this into public_html/jihad/ and Extract"
```

In cPanel File Manager: upload `dist.zip` into `public_html/jihad/` → right-click → **Extract**.

You're done — `https://jihad.olivosoft.com` is now live. 🚀
