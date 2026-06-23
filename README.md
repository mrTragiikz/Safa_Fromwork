# SAFA Formwork & Scaffolding — Web Platform

A full-stack PHP/MySQL website for **SAFA Formwork & Scaffolding**, a construction services company. It pairs a public marketing site (home, about, services, projects, contact) with a custom-built admin dashboard for managing projects, project galleries, and client inquiries — no third-party CMS required.

---

## Table of Contents

- [Live Structure](#live-structure)
- [Technology Stack](#technology-stack)
- [Features](#features)
- [Admin Dashboard](#admin-dashboard)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [API Endpoints](#api-endpoints)
- [Security Notes](#security-notes)
- [Deployment](#deployment)
- [License](#license)
- [Support / Hire the Developer](#support--hire-the-developer)

---

## Live Structure

| Layer | Pages |
|---|---|
| Public site | `index.php`, `about.php`, `services.php`, `projects.php`, `project.php`, `contact.php` |
| Admin dashboard | `ghar/` (login, projects CRUD, inquiries inbox) |
| Backend APIs | `api/get_projects.php`, `api/send_inquiry.php` |

---

## Technology Stack

**Frontend**
- HTML5, CSS3 (per-page stylesheets in [css/](css/)), vanilla JavaScript (per-page scripts in [js/](js/))
- Custom image/gallery slider, video handling (`js/slider.js`, `js/video-handler.js`)
- PWA-ready: includes a service worker ([sw.js](sw.js))

**Backend**
- PHP 7.4+ / PHP 8.x, procedural with included helper modules (no framework)
- PDO (MySQL) with prepared statements throughout

**Database**
- MySQL / MariaDB — schema and seed data in [safa_formwork.sql](safa_formwork.sql)
- Core tables: `projects`, `project_images`, `inquiries`, `admins` (auto-created on first run if missing)

**Email**
- SMTP-based notifications (Gmail SMTP by default) via [includes/smtp_mail.php](includes/smtp_mail.php) and [includes/send_mail.php](includes/send_mail.php)
- Configured in [includes/smtp_config.php](includes/smtp_config.php)

**Server / Infra**
- Apache with `.htaccess` (pretty URLs, security headers, upload limits)
- `php.ini` / `.user.ini` overrides for large file uploads (up to 512M) and longer execution time on shared hosting

---

## Features

### Public site
- Responsive multi-page marketing site (Home, About, Services, Projects, Contact)
- Dynamic project listing pulled live from the database via JSON API
- Project detail pages with image galleries
- Contact form with server-side validation, stored in DB, and emailed to the admin via SMTP

### Admin dashboard
- Custom-built dashboard (no third-party admin panel) to manage all dynamic content
- Add / edit / delete projects, each with a cover image and multiple gallery images
- Bulk image upload support (large file size limits configured for high-res project photos)
- Inquiries inbox — view, mark read/unread, and manage all client submissions from the contact form
- Session-based authentication with CSRF protection, login rate-limiting, and security event logging

---

## Admin Dashboard

The admin area lives in [ghar/](ghar/) (intentionally not named `admin/`, to reduce automated probing).

| Purpose | File |
|---|---|
| Login page | `ghar/xtzprabin12.php` |
| Dashboard (projects management) | `ghar/xtztragiikz.php` |
| Inquiries inbox | `ghar/inquiries.php` |
| Project deletion handler | `ghar/project_delete.php` |
| Logout | `ghar/logout.php` |

**First-time login:** On first visit to the login page, the app auto-creates an `admins` table and seeds a default account if none exists:

```
Username: admin
Password: admin123
```

> ⚠️ Change this password immediately after first login (update the `admins` table / add a password-change flow before going live). Never leave the default credentials active on a public deployment.

**What the dashboard does:**
- **Projects (`xtztragiikz.php`)** — create/edit/delete projects (title, category: current/completed/past, location, description), upload a cover image plus a gallery of additional images per project.
- **Inquiries (`inquiries.php`)** — lists every contact-form submission with name, email, phone, project type, subject, and message; lets the admin triage read/unread status.
- **Auth & sessions** — `includes/security.php` handles session hardening (HttpOnly/SameSite cookies, 30-min session rotation, 1-hour inactivity timeout), CSRF tokens on all forms, and a 5-attempts-per-5-minutes login rate limiter (resettable via `?reset_attempts=admin` for local dev).

---

## Getting Started

### Requirements
- PHP 7.4+ (8.x recommended)
- MySQL 5.7+ / MariaDB
- Apache with `mod_rewrite` and `mod_headers` (XAMPP/WAMP/LAMP/MAMP all work locally)

### 1. Clone the repository
```sh
git clone https://github.com/PDInepal-Pvt-Ltd/group3.git
cd group3
```

### 2. Create the database
```sh
mysql -u root -p -e "CREATE DATABASE safa_formwork"
mysql -u root -p safa_formwork < safa_formwork.sql
```
(Tables also self-create on first request if you skip the import — see `api/get_projects.php`, `api/send_inquiry.php`, `ghar/xtzprabin12.php`.)

### 3. Configure the database connection
Edit [includes/db_connect.php](includes/db_connect.php):
```php
$dbConfig = [
    'host'    => 'localhost',
    'dbname'  => 'safa_formwork',
    'user'    => 'root',
    'pass'    => '',
    'charset' => 'utf8mb4',
];
```

### 4. Configure outgoing email
Edit [includes/smtp_config.php](includes/smtp_config.php) with your SMTP credentials (Gmail App Password recommended if using Gmail SMTP):
```php
return [
    'smtp_host'     => 'smtp.gmail.com',
    'smtp_port'     => 587,
    'smtp_secure'   => 'tls',
    'smtp_username' => 'you@gmail.com',
    'smtp_password' => 'your-app-password',
    'from_email'    => 'you@gmail.com',
    'from_name'     => 'Safa Formwork & Scaffolding',
    'to_email'      => 'inbox@yourdomain.com',
    'to_name'       => 'Safa Formwork Team',
    'reply_to_email'=> 'you@gmail.com',
    'reply_to_name' => 'Safa Formwork & Scaffolding',
];
```

### 5. Serve the app
Point your web server's document root at the project folder, then visit:
```
http://localhost/                 → public site
http://localhost/ghar/xtzprabin12.php → admin login
```

### 6. Make `uploads/` writable
```sh
chmod -R 755 uploads/
```

---

## Configuration

| What | Where |
|---|---|
| DB credentials | `includes/db_connect.php` |
| SMTP credentials | `includes/smtp_config.php` |
| Upload size limits | `php.ini`, `.htaccess`, `.user.ini` (default 512M / 600s execution for large galleries) |
| Security headers / CSP | `includes/security.php`, `ghar/.htaccess` |

---

## Project Structure

```
├── index.php, about.php, services.php, projects.php, project.php, contact.php
├── footer.php                  # Shared footer include
├── sw.js                       # Service worker
├── safa_formwork.sql           # Full DB schema + seed data
├── api/
│   ├── get_projects.php        # GET — JSON list of all projects + images
│   └── send_inquiry.php        # POST — store inquiry + trigger email
├── ghar/                       # Admin dashboard (auth-protected)
│   ├── xtzprabin12.php         # Login
│   ├── xtztragiikz.php         # Projects dashboard (CRUD + uploads)
│   ├── inquiries.php           # Inquiries inbox
│   ├── project_delete.php      # Delete handler
│   └── logout.php
├── includes/
│   ├── db_connect.php          # PDO connection
│   ├── security.php            # Auth, CSRF, rate limiting, logging
│   ├── send_mail.php / smtp_mail.php / smtp_config.php
│   └── header.php
├── css/                        # Per-page stylesheets
├── js/                         # Per-page scripts
├── assets/                     # Static media (gallery, etc.)
├── uploads/                    # User-uploaded project/gallery images
└── logs/                       # Security event log (gitignored in production)
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/get_projects.php` | Returns all projects with nested gallery images as JSON |
| POST | `/api/send_inquiry.php` | Accepts `name, email, phone, project_type, subject, message`; stores + emails the inquiry |

---

## Security Notes

- Passwords are hashed with `password_hash()` / verified with `password_verify()` (bcrypt) — never stored in plaintext.
- All admin queries use PDO prepared statements (no string-concatenated SQL).
- CSRF tokens required on the login form; session cookies are HttpOnly + SameSite=Strict.
- Security-relevant events (login success/failure, CSRF mismatch, unauthorized access, rate-limit hits) are logged to `logs/security.log`.
- File uploads are validated by MIME type (`finfo`) and size before being written to disk.
- **Before going to production:** rotate the default admin password, set real SMTP credentials, set `session.cookie_secure` to `1` once served over HTTPS, and ensure `logs/` and `.sql` files are not web-accessible (already blocked in `ghar/.htaccess`, verify root `.htaccess` covers the rest).

---

## Deployment

Works on any standard LAMP/WAMP stack or shared PHP hosting (cPanel, Plesk, etc.):

1. Upload all files via FTP/Git, keeping the folder structure intact.
2. Create the MySQL database and import `safa_formwork.sql`.
3. Update `includes/db_connect.php` and `includes/smtp_config.php` with production credentials.
4. Set `uploads/` and `logs/` to writable (755).
5. Point your domain/document root at the project root.
6. Log in at `/ghar/xtzprabin12.php`, change the default admin password, and start adding real projects.

---

## License

This project is proprietary software built for SAFA Formwork & Scaffolding. All rights reserved unless otherwise licensed by the owner.

---

## Support / Hire the Developer

This platform — public site + custom admin dashboard + secure backend — was built end-to-end as a freelance/internship project.

**Interested in buying this template, a custom version for your own business, or hiring for similar work?**
Reach out: **sharmaprabin160@gmail.com**

Available for:
- Selling/licensing this codebase (with or without setup support)
- Custom builds based on this stack (PHP/MySQL business sites + admin dashboards)
- Ongoing maintenance, hosting setup, and feature additions
