# VANIX STUDIO — PHP & MySQL Full-Stack Portal

This documentation outlines the architecture, directory structure, compilation workflow, and deployment details for the full-stack **VANIX STUDIO Portal** running on standard PHP + MySQL shared web hosting.

---

## 🗺️ Project Architecture Overview

```
                          Browser (HTML/CSS/JS)
                                    │
                         HTTP /api/... (Relative)
                                    ▼
┌──────────────────────────────────────────────────────────────────────┐
│                  Apache / LiteSpeed Web Server                       │
│  ┌───────────────────────────────┐  ┌─────────────────────────────┐  │
│  │   Static File Serving         │  │   PHP Router (api/index.php)│  │
│  │  (Served directly from root)  │  │  (Rewritten via .htaccess)  │  │
│  └───────────────────────────────┘  └─────────────────────────────┘  │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │  PDO MySQL Connection
                                   ▼
                       ┌─────────────────────────┐
                       │    MySQL / MariaDB      │
                       │     (vanix_studio)      │
                       └─────────────────────────┘
```

1. **Frontend**: Pure vanilla HTML5, CSS3, and progressive JavaScript located at the project root. Interactive views utilize layout components and styles from the `css/` and `js/` directories.
2. **Backend**: Lightweight PHP backend controller system located inside `api/`. Endpoint routing to `/api/*` is handled via Apache `.htaccess` rewrite rules mapping requests to a single entry point `api/index.php`.
3. **Database**: Relational MySQL or MariaDB database running natively on PHP shared hosting.

---

## 🔐 Security & Credentials Protection

> [!IMPORTANT]
> **Never commit your `.env` file or raw credentials to the Git repository.**
>
> The project's `.gitignore` is configured to ignore `.env`, `api/.env`, and `.gitpat.txt` by default. 
> To configure credentials securely:
> 1. Copy the `.env.example` file in the root to `.env` (or create a `.env` in the `api/` directory).
> 2. Populate the `.env` file with your environment-specific credentials.

### ⚙️ Environment Configuration Template (`.env`)

```env
# Database connection string (supports mysql or postgres formats)
DATABASE_URL=mysql://your_db_username:your_db_password@your_db_host:3306/your_db_name

# JWT Security Configurations (use a long, random secret string in production)
JWT_SECRET_KEY=your_jwt_secret_key_here
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# Super Admin setup credentials (seeded on setup script execution)
SUPER_ADMIN_EMAIL=your_super_admin_email@domain.com
SUPER_ADMIN_PASSWORD=your_secure_password_here

# Gmail SMTP Configuration for email alerts & notifications
# Leave blank to use host's native mail() fallback
GMAIL_USER=your_gmail_user@gmail.com
GMAIL_APP_PASSWORD=your_gmail_app_password_here

# CORS and Domain Settings
FRONTEND_URL=http://localhost:8000
PROD_DOMAIN=yourdomain.com
ENV=production

# Firebase Web Client Configuration (For Google Sign-In verification)
FIREBASE_API_KEY=your_firebase_api_key
FIREBASE_AUTH_DOMAIN=your_firebase_auth_domain
FIREBASE_PROJECT_ID=your_firebase_project_id
FIREBASE_STORAGE_BUCKET=your_firebase_storage_bucket
FIREBASE_APP_ID=your_firebase_app_id
FIREBASE_MESSAGING_SENDER_ID=your_firebase_sender_id
FIREBASE_MEASUREMENT_ID=your_firebase_measurement_id
```

---

## 📁 Project Directory Structure

```
vanixstudio/
├── api/                        ← Backend REST API & server logic
│   ├── .htaccess               ← Apache rewrite rules for clean routes
│   ├── index.php               ← Unified router entry point & controller handlers (30+ routes)
│   ├── config.php              ← Environment configuration loader (.env) & PDO connection
│   ├── auth.php                ← Timing-safe HS256 JWT authorization & password bcrypt helper
│   ├── mail.php                ← Gmail SMTP/Socket helper and mailer templates
│   └── seed.php                ← Database bootstrap & initial Super Admin creation
│
├── assets/                     ← Project graphics, images, logos & video assets
│   └── images/                 ← Logo SVGs, JPEGs, and animations
│
├── css/                        ← Cascading Style Sheets
│   ├── main.css                ← Global/shared layout styling
│   ├── pages/                  ← Page-specific stylesheets
│   └── *.src.css               ← Original uncompiled styling source files
│
├── database/                   ← Database setup and initialization scripts
│   └── schema_mysql.sql        ← DDL SQL script defining tables, enums, FK constraints, and indexes
│
├── js/                         ← Client-side javascript controllers
│   ├── api-config.js           ← Configuration for client API domains
│   ├── pages/                  ← Page-specific controllers
│   └── *.src.js                ← Original uncompiled javascript source files
│
├── pages/                      ← Subpages & dashboards
│   ├── *.html                  ← Production dashboard and subpages
│   └── *.src.html              ← Original uncompiled subpages source files
│
├── .env.example                ← Template configuration file containing placeholders
├── .gitignore                  ← Configured to prevent committing sensitive keys and dependencies
├── index.html                  ← Portal Homepage (compiled production version)
├── index.src.html              ← Homepage source code (original uncompiled source)
├── compile.py                  ← Python compile script for obfuscating/minifying assets
├── router.php                  ← Local environment router mapping API endpoints and HTML pages
├── robots.txt                  ← Search engine crawler rules
└── sitemap.xml                 ← SEO Sitemap index
```

---

## 🛠️ Build & Compilation Workflow

The project contains a source directory structure designed to prevent casual code extraction by obfuscating static page elements in the production output files.

* **Source Files (`*.src.html`, `*.src.css`, `*.src.js`)**: These are your working files where you should write/edit code.
* **Compiled Files (`*.html`, `*.css`, `*.js`)**: These files are generated by the compiler script. They minify stylesheets, and base64 obfuscate JS/HTML templates so they do not render source code on client-side inspectors.

### 💻 Compiling Source Files

Whenever you make changes to files ending in `.src.*`, run the compilation script to generate the production-ready code:

```bash
python compile.py
```

---

## 🚀 Commands & Local Development

### 1. Local Development Server

PHP includes a built-in server. You can spin up the environment locally without Apache:

```bash
php -S localhost:8000 router.php
```

Once running, navigate to `http://localhost:8000` in your web browser. The local development router (`router.php`) will handle API requests to `/api/*` and render directories as expected.

### 2. Manual Database Setup

If you prefer setting up the MySQL schema manually or importing it via PHPMyAdmin, run the DDL schema script:

```bash
mysql -u your_username -p your_database < database/schema_mysql.sql
```

---

## 📦 Production Deployment (e.g., Hostinger Shared Hosting)

To deploy the portal onto shared web hosting:

1. **Setup MySQL Database**:
   - Go to your Hosting Panel (e.g. hPanel).
   - Create a MySQL Database, a user, and record the host, name, username, and password.
2. **Configure Environment variables**:
   - Create a file named `.env` inside the `api/` folder (or at the root directory of your host, depending on the loader path).
   - Insert your `DATABASE_URL` and other configs using your real credentials.
3. **Upload Files**:
   - Connect via FTP (e.g. FileZilla) or use the Hostinger File Manager.
   - Upload the compiled files (like `index.html`, `css/`, `js/`, `assets/`, `pages/`, etc.) directly into the `public_html/` root.
   - Upload the `api/` directory into `public_html/api/`.
4. **Bootstrap & Seed**:
   - Run the initial database setup script by visiting:
     `https://yourdomain.com/api/seed.php`
   - This script creates all required MySQL tables and establishes your initial Super Admin user.
