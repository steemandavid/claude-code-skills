# Deploy steeman.be Website

Build and deploy the Hugo website to production via FTP.

## Prerequisites
- Project path: `/home/john/claudecode/projects/website-steeman.be`
- FTP credentials in `~/.netrc`
- Hugo installed (`hugo` in PATH)

## Workflow

### 1. Build Production Site

```bash
cd /home/john/claudecode/projects/website-steeman.be
hugo --gc --minify
```

Verify the build succeeds with no errors.

### 2. Deploy to FTP

**FTP connection details:**
- Host: `ftp.steeman.be` (resolves to `83.229.19.64`)
- Server: Pure-FTPd with TLS
- Username: `steem1482777`
- Password: stored in `~/.netrc`
- Remote root: `/` (site files are at root level, NOT inside `/www/`)
- Files to deploy: contents of `public/` directory

**Important:** The password contains special characters. When using curl, URL-encode `@` as `%40`.

**Connection test:**
```bash
curl -v --connect-timeout 10 "ftp://steem1482777:NusBnvg7P%40XUMdm@ftp.steeman.be/" --list-only
```

**Full site mirror (preferred when lftp works):**
```bash
lftp -c 'set ftp:ssl-allow false; set ftp:passive-mode true; open -u steem1482777,<password> ftp.steeman.be; mirror --reverse --delete --verbose public/ /'
```

**Fallback — curl file-by-file upload:**
If lftp hangs or fails, use curl to upload changed files individually. This is the reliable approach:

```bash
FTPCREDS="ftp://steem1482777:NusBnvg7P%40XUMdm@ftp.steeman.be"

# Upload a single file
curl -s --connect-timeout 15 --ftp-create-dirs -T <local-path> "$FTPCREDS/<remote-path>"
```

### 3. What to Upload

For a **new blog post** (`content/posts/my-post.md`):
- `public/posts/my-post/index.html` → `/posts/my-post/index.html`
- `public/index.html` → `/index.html`
- `public/index.xml` → `/index.xml` (RSS)
- `public/index.json` → `/index.json`
- `public/sitemap.xml` → `/sitemap.xml`
- `public/posts/index.html` → `/posts/index.html`
- `public/posts/index.xml` → `/posts/index.xml`
- All `public/page/*/index.html` → `/page/*/index.html`
- Category pages for the post's categories: `public/categories/<cat>/index.html` + `.xml`
- Category pagination: `public/categories/<cat>/page/*/index.html`

For a **full site deploy** (any change):
- Upload everything in `public/` using mirror or a recursive curl loop.
- Use `--ftp-create-dirs` to create directories automatically.

### 4. Verify

```bash
curl -s -o /dev/null -w "%{http_code}" https://www.steeman.be/
curl -s -o /dev/null -w "%{http_code}" https://www.steeman.be/posts/<post-slug>/
```

### 5. Hugo Dev Server (for previewing)

```bash
# Start dev server (correct IP for john-ai host)
hugo server -D --bind 0.0.0.0 --baseURL http://192.168.1.165

# Preview at: http://192.168.1.165:1313/
```

**Note:** The john-ai host IP is `192.168.1.165`, NOT `192.168.1.77`.

### Gotchas

- lftp `mirror` may hang silently on this FTP server. If no progress after 30 seconds, kill it and fall back to curl.
- FTP certificate CN does not match `ftp.steeman.be` — if using lftp with TLS, set `ssl:verify-certificate false`.
- Password `@` must be URL-encoded as `%40` in curl URLs.
- The `old-site/` directory has many modified binary files — do NOT `git add .` in the website repo. Always stage specific files.
