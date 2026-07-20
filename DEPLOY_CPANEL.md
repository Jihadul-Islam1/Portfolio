# Deploying the Portfolio to cPanel — Step by Step

Your project is a **Vite + React (SPA)** static site. cPanel can host it, but you must build first and handle SPA routing.

---

## 1. Build the production bundle

On your **local machine**, in the project folder:

```bash
cd d:\portfolio
npm install          # only first time
npm run build
```

This creates a `dist/` folder containing the static files (HTML, JS, CSS, assets).

> Note: `npm run build` outputs to `dist/` by default in Vite. Confirm by opening `d:\portfolio\dist` — you should see `index.html` and an `assets/` folder inside.

---

## 2. Log into cPanel

1. Open your cPanel URL (e.g. `https://yourdomain.com:2083` or `https://cpanel.yourhost.com`).
2. Login with the credentials your hosting provider gave you.

---

## 3. Upload the build to `public_html`

### Option A — File Manager (easiest)

1. In cPanel, open **File Manager**.
2. Go to `public_html/` (the root folder for your domain). If you want the site at `yourdomain.com/portfolio`, open that subfolder instead.
3. Click **Upload**, select everything **inside** the `dist/` folder (not the folder itself):
   - `index.html`
   - the `assets/` folder
4. Wait until upload finishes.

> Make sure `index.html` ends up directly under `public_html/`, not inside `dist/`.

### Option B — FTP / Git (faster for many files)

1. Use **FileZilla** or **WinSCP**.
2. Host: `yourdomain.com` (or the FTP hostname from cPanel → FTP Accounts).
3. User/pass: a cPanel FTP account.
4. Drag everything inside `dist/` into `public_html/`.

---

## 4. Fix SPA routing (so refreshes don't 404)

Because `React Router` (or any client-side routing) rewrites paths like `/about` in the browser, hitting them directly on the server returns a 404. Fix with an `.htaccess` file inside `public_html/`:

```apache
# public_html/.htaccess
RewriteEngine On
RewriteBase /

# If the request is not an existing file or folder, serve index.html
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^.*$ /index.html [L]

# Optional: cache static assets aggressively
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

In cPanel File Manager → click **Settings** (top right) → enable **Show Hidden Files (dotfiles)** so you can see `.htaccess`.

---

## 5. Set the Node version (only if your host runs Node at all)

You **do not need Node** on the server — the build is static HTML/JS. So you can skip this unless your host requires a Node version to be selected.

If cPanel asks:

1. Go to **MultiPHP Manager** / **Select PHP Version** (not needed for static site).
2. For the pure static version, this section is irrelevant.

---

## 6. Configure HTTPS (free Let's Encrypt)

1. In cPanel, open **SSL/TLS Status** or **Let's Encrypt™ SSL**.
2. Select your domain → click **Install**.
3. After install, **Enforce HTTPS** redirect (under **Domains → Redirects** or **Force HTTPS Redirect**).

---

## 7. Point your domain (if not already done)

If `yourdomain.com` is registered elsewhere:

1. cPanel → **Zone Editor** / **DNS Zone**.
2. Update the `A` record to point at the host's IP (your hosting provider will tell you this).
3. Or set the nameservers to the values cPanel shows under **Domains → Manage**.

DNS can take **24–48 hours** to propagate.

---

## 8. Verify

1. Visit `https://yourdomain.com` — you should see your portfolio.
2. Refresh on any route — no 404.
3. Check DevTools → Network — assets should load with `200`.

---

## Common gotchas

| Issue | Fix |
|---|---|
| 404 on `/about` (etc.) when reloading | Make sure `.htaccess` exists in `public_html/` with the rewrite rule above. |
| Site loads but assets 404 | You uploaded the `dist/` folder itself. Upload the **contents** of `dist/` so `index.html` is directly under `public_html/`. |
| `/assets/original/...svg` 404 | Make sure the `assets/` folder was uploaded intact — don't rename it. |
| Old site still showing | Clear browser cache, or open in incognito. cPanel may also cache via Varnish — ask host to flush. |
| 500 error after upload | Check `.htaccess` syntax; ensure `mod_rewrite` is enabled (most shared hosts enable it by default). |

---

## Optional: deploy via Git from cPanel

If you'd rather push updates from GitHub:

1. cPanel → **Git Version Control**.
2. Click **Create**. Point it at your GitHub repo URL.
3. Set the deployment path to `public_html/` (or a temp folder) and run `npm ci && npm run build` as the deployment script, then move `dist/*` to `public_html/`.
4. After first deploy, hit **Update** in cPanel whenever you push.

---

You're done — your portfolio should be live at your domain. 🎉
