# Laravel cPanel Deployment Guide

This guide explains how to deploy a Laravel application on cPanel shared hosting by keeping everything in the public folder and using `.htaccess` for proper routing.

### 1. Project Structure in public_html

Upload your entire Laravel project to `public_html`. Your structure should look like:

```
public_html/
├── app/
├── bootstrap/
├── config/
├── database/
├── resources/
├── routes/
├── storage/
├── vendor/
├── public/
│   ├── index.php
│   ├── .htaccess
│   ├── css/
│   ├── js/
│   └── build/ (Vue.js built assets)
├── .env
├── artisan
└── composer.json
```

### 2. Root .htaccess Configuration

Create a `.htaccess` file in your `public_html` root directory:

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    
    # SSL Redirection (Force HTTPS)
    RewriteCond %{HTTPS} !on
    RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
    
    # Redirect all subdomains to main domain (dynamic) or show 404
    RewriteCond %{HTTP_HOST} ^(.+)\.(.+\..+)$ [NC]
    RewriteCond %1 !^www$ [NC]
    RewriteRule ^ - [R=404,L]
    
    # Alternative: Redirect subdomains to main domain instead of 404
    # RewriteCond %{HTTP_HOST} ^(.+)\.(.+\..+)$ [NC]
    # RewriteCond %1 !^www$ [NC]
    # RewriteRule ^ https://%2%{REQUEST_URI} [R=301,QSA,NC,L]
    
    # Deny access to sensitive directories and files (return 404 instead of 403)
    RewriteRule ^(app|bootstrap|config|database|resources|routes|storage|vendor|tests)(/.*)?$ - [R=404,L]
    
    # Prevent direct access to /public/ URLs - Option 1: Redirect to homepage
    RewriteRule ^public/?(.*)$ / [R=301,L]
    
    # Alternative Option 2: Show 404 for /public/ URLs (uncomment to use instead)
    # RewriteRule ^public/?(.*)$ - [R=404,L]
    
    # Allow access to Laravel's public index.php
    RewriteRule ^public/index\.php$ - [L]
    
    # Redirect everything to public folder
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_URI} !^/public/
    RewriteRule ^(.*)$ /public/$1 [L,QSA]
    
    # Handle Laravel routes in public folder
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^public/(.*)$ /public/index.php [L,QSA]
</IfModule>

# Security: Deny access to sensitive files (return 404 instead of 403)
<Files ~ "^(artisan|phpunit\.xml|\.env|composer\.(json|lock)|package\.(json|lock))$">
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteRule .* - [R=404,L]
    </IfModule>
</Files>

# Security: Deny access to hidden files (return 404 instead of 403)
<FilesMatch "^\.">
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteRule .* - [R=404,L]
    </IfModule>
</FilesMatch>

# Security: Deny access to backup and config files (return 404 instead of 403)
<FilesMatch "\.(htaccess|htpasswd|ini|log|sh|sql|yml|yaml|config)$">
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteRule .* - [R=404,L]
    </IfModule>
</FilesMatch>

# Disable directory browsing
<IfModule mod_autoindex.c>
    Options -Indexes
</IfModule>

# Browser Caching
<IfModule mod_expires.c>
    ExpiresActive on
    ExpiresDefault "access plus 1 month"
    
    # CSS and JavaScript
    ExpiresByType text/css "access plus 1 year"
    ExpiresByType application/javascript "access plus 1 year"
    ExpiresByType text/javascript "access plus 1 year"
    
    # Images
    ExpiresByType image/gif "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 month"
    ExpiresByType image/png "access plus 1 month"
    ExpiresByType image/webp "access plus 1 month"
    ExpiresByType image/svg+xml "access plus 1 month"
    ExpiresByType image/x-icon "access plus 1 week"
    
    # Fonts
    ExpiresByType application/font-woff2 "access plus 1 month"
    ExpiresByType application/font-woff "access plus 1 month"
    ExpiresByType application/vnd.ms-fontobject "access plus 1 month"
    ExpiresByType application/x-font-ttf "access plus 1 month"
    ExpiresByType font/opentype "access plus 1 month"
    
    # HTML and Data
    ExpiresByType text/html "access plus 0 seconds"
    ExpiresByType application/json "access plus 0 seconds"
    ExpiresByType application/xml "access plus 0 seconds"
</IfModule>

# Enable Compression
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/javascript
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/json
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE image/svg+xml
</IfModule>
```

### 3. Public Folder .htaccess

Ensure your `public/.htaccess` file contains the standard Laravel configuration:

```apache
<IfModule mod_rewrite.c>
    <IfModule mod_negotiation.c>
        Options -MultiViews -Indexes
    </IfModule>

    RewriteEngine On

    # Handle Authorization Header
    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    # Redirect Trailing Slashes If Not A Folder...
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_URI} (.+)/$
    RewriteRule ^ %1 [L,R=301]

    # Send Requests To Front Controller...
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
</IfModule>
```

### 4. Environment Configuration

Create/update your `.env` file in the root of `public_html`:

```env
APP_NAME=Laravel
APP_ENV=production
APP_KEY=base64:your-generated-key-here
APP_DEBUG=false
APP_URL=https://yourdomain.com

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=your_cpanel_database_name
DB_USERNAME=your_cpanel_db_username
DB_PASSWORD=your_cpanel_db_password

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MEMCACHED_HOST=127.0.0.1

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

### 5. Database Setup

1. **Create Database in cPanel:**

   - Go to "MySQL Databases"
   - Create a new database
   - Create a database user
   - Assign user to database with all privileges
2. **Update .env with database credentials**
3. **Run migrations** (if you have terminal access):

   ```bash
   php artisan migrate --force
   ```

### 6. Generate Application Key

**If you have terminal/SSH access:**

```bash
php artisan key:generate
```

**If you don't have terminal access:**

- Generate a key online using Laravel key generators
- Add it manually to your `.env` file

### 7. Build and Deploy Vue.js Assets

**Before uploading, build your Vue.js assets locally:**

```bash
npm install
npm run build
```

This creates production-ready files in the `public/build/` directory.

### 8. Set Proper Permissions

If you have file manager access, set these permissions:

- `storage/` folder: 755 or 775
- `bootstrap/cache/` folder: 755 or 775
- `.env` file: 644

### 9. Cache Configuration (Optional)

If you have terminal access, optimize for production:

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## Troubleshooting

### Common Issues

**500 Internal Server Error:**

- Check cPanel error logs
- Verify `.env` file exists and has correct syntax
- Ensure proper file permissions on storage folders
- Check if `APP_KEY` is generated

**Assets not loading:**

- Verify `npm run build` was executed
- Check if `public/build/` folder exists with compiled assets
- Ensure Vite configuration is set for production

**Database connection errors:**

- Double-check database credentials in `.env`
- Verify database and user exist in cPanel
- Confirm user has proper privileges

**Routes not working:**

- Verify `.htaccess` files are uploaded correctly
- Check if mod_rewrite is enabled on server
- Ensure no syntax errors in `.htaccess`

### File Permissions Guide

```
Folders: 755 or 775
Files: 644
.env: 644 (important for security)
```

## Security Notes

- The root `.htaccess` denies access to sensitive Laravel directories
- Never expose your `.env` file publicly
- Keep `APP_DEBUG=false` in production
- Consider adding additional security headers to `.htaccess`

## Additional Security .htaccess (Optional)

Add to your root `.htaccess` for extra security:

```apache
# Prevent access to .env file
<Files .env>
    Order allow,deny
    Deny from all
</Files>

# Prevent access to composer files
<Files composer.json>
    Order allow,deny
    Deny from all
</Files>

<Files composer.lock>
    Order allow,deny
    Deny from all
</Files>
```

## Pre-deployment Checklist

- [ ] Run `npm run build` locally
- [ ] Upload entire project to `public_html`
- [ ] Create/upload root `.htaccess` file
- [ ] Verify `public/.htaccess` exists
- [ ] Create `.env` file with production settings
- [ ] Set up database in cPanel
- [ ] Update database credentials in `.env`
- [ ] Generate/add `APP_KEY`
- [ ] Set proper file permissions
- [ ] Test the website

---

**Note:** This deployment method keeps all files in the public directory for simplicity on shared hosting. For better security on VPS/dedicated servers, consider moving Laravel files outside the web root.
