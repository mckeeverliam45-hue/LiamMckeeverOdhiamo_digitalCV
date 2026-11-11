# Digital CV & Portfolio Website

## Project guide

This guide describes the implemented portfolio website, its admin dashboard, the PHP
and MySQL backend, and how to run and maintain the project with Laragon on Windows.
It is written for the site owner, administrators, and developers who may maintain
the application.

## 1. What the website does

The public site presents Liam Mckeever Odhiambo's professional profile and work.
Visitors can browse the portfolio and submit a contact enquiry. The site stores
valid enquiries in MySQL; it does not send email notifications.

An administrator signs in to a separate dashboard to read or delete enquiries and
manage the database-backed projects, education records, and skills shown on the
public site.

## 2. Pages and public features

The public portfolio is a single-page site with in-page navigation to:

- **Home:** introduction, professional focus, and links to projects and contact.
- **About:** profile summary and personal information.
- **Education:** education and qualification timeline. Database records replace
  the starter timeline when loaded.
- **Certificates:** links to certificate documents included in the project.
- **Experience:** practical and professional focus areas.
- **Skills:** skill name, category, and description. Database records replace the
  starter cards when loaded.
- **Projects:** project cards and optional project links and images. A project
  with the same URL as a persistent hand-coded project is not shown twice.
- **Contact:** contact details and an enquiry form.

The layout includes responsive navigation, reduced-motion-aware reveal effects,
scroll progress, and a back-to-top control. The contact form displays its result
on the page and disables the submit button while the request is in progress.

## 3. Admin dashboard

Open the dashboard at:

`http://localhost/liamPortfoloo/index/admin.html`

After signing in, the dashboard provides:

| Area | Available actions |
| --- | --- |
| Overview | Counts of messages, projects, education records, and skills |
| Messages | Read submitted messages, refresh the list, and delete a message |
| Projects | Create, edit, and delete projects |
| Education | Create, edit, and delete education records |
| Skills | Create, edit, and delete skills |

The sign-in session is stored in a server-side PHP session. Mutating requests use
a session CSRF token. Signing out clears the session. Use the same hostname
consistently (for example, `localhost`) so the browser sends the session cookie
back to Apache.

The dashboard's image field accepts an image URL or a path that the browser can
reach; it does not upload image files. Store image files in the project (or use
an accessible URL), then enter the appropriate path or URL.

## 4. Technology and request flow

- **Frontend:** HTML, CSS, and vanilla JavaScript; no frontend package manager is
  required.
- **Backend:** PHP with PDO and the PDO MySQL driver.
- **Database:** MySQL or a compatible MariaDB server.
- **Local environment:** Laragon's Apache and MySQL services on Windows.

The public page loads database-backed portfolio content from
`backend/admin-api.php?resource=portfolio`. The contact form submits a `POST` to
`backend/contact.php`. The admin interface calls `backend/admin-api.php` for
authentication, session status, data retrieval, and changes.

Important files:

| Path | Purpose |
| --- | --- |
| `index/index.html` | Public single-page portfolio |
| `index/admin.html` | Admin sign-in and dashboard markup |
| `index/styles.css` | Public-site styles |
| `css/styles.css` | Additional stylesheet in the repository; the current HTML pages load `index/styles.css` |
| `js/portfolio-content.js` | Navigation effects and loading dynamic public content |
| `js/contact.js` | Contact form submission and feedback |
| `js/admin.js` | Admin login, dashboard rendering, and form actions |
| `backend/admin-api.php` | Session, portfolio read, and admin CRUD API |
| `backend/contact.php` | Contact form validation and message creation |
| `backend/db.php` | PDO MySQL connection configuration |
| `backend/create_admin.php` | Command-line admin account creation/password reset |
| `database/digital_cv.sql` | Database, table, and starter-record definitions |
| `documents/` | Certificate and supporting documents linked from the site |
| `images/` | Site image assets |
| `assets/` | Brand and interface assets |

## 5. Database

The supplied SQL script creates the `digital_cv` database and these tables:

| Table | Contents |
| --- | --- |
| `admins` | Unique administrator username and password hash |
| `messages` | Name, email, subject, message, and submission timestamp |
| `projects` | Title, description, optional project URL and image path/URL |
| `education` | Institution, qualification, optional years, and description |
| `skills` | Name, category, description, and display order |

The SQL script seeds two projects, two education records, and five skills only
when each respective table is empty. It does not create an administrator account.
Importing the script again creates missing tables and adds starter data only to
empty tables; it does not replace existing portfolio rows.

### Connection defaults

`backend/db.php` uses these defaults:

| Setting | Default |
| --- | --- |
| Host | `localhost` |
| Port | `3306` |
| Database | `digital_cv` |
| Username | `root` |
| Password | Empty |

If your Laragon MySQL credentials differ, set `DB_HOST`, `DB_PORT`, `DB_NAME`,
`DB_USER`, and `DB_PASS` in the PHP/Apache environment. Do not commit credentials
to the repository or place them in a public document.

## 6. Run the website with Laragon

### Requirements

- Windows with Laragon installed.
- Apache and MySQL started in Laragon.
- PHP with PDO MySQL enabled.
- The project folder located at `C:\laragon\www\liamPortfoloo`.

### First-time database setup

1. Start **Apache** and **MySQL** from Laragon.
2. Open HeidiSQL from the Laragon menu and connect to the local MySQL server.
3. Open and execute `database/digital_cv.sql` from this project.
4. Confirm that the `digital_cv` database contains `admins`, `messages`,
   `projects`, `education`, and `skills`.

The SQL file begins with `CREATE DATABASE IF NOT EXISTS digital_cv;` and selects
that database, so it can be imported directly in HeidiSQL.

### Create or reset the admin account

Create the account from Laragon's PowerShell terminal. The password prompt is
hidden, requires at least 12 characters, and is not saved in this guide. Replace
`Mckeever` with the username you want:

```powershell
Set-Location C:\laragon\www\liamPortfoloo
$env:ADMIN_USERNAME = 'Mckeever'
$securePassword = Read-Host "Choose an admin password (12+ characters)" -AsSecureString
$secretPointer = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($securePassword)
try {
  $env:ADMIN_PASSWORD = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($secretPointer)
  php backend/create_admin.php
} finally {
  Remove-Item Env:ADMIN_PASSWORD, Env:ADMIN_USERNAME -ErrorAction SilentlyContinue
  [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($secretPointer)
}
```

If PHP is not recognized in the Laragon terminal, invoke Laragon's PHP executable
by its full path, or check Laragon's PHP version/path configuration. Running this
script again for the same username **resets that account's password**.

### Open the pages

- Public site: `http://localhost/liamPortfoloo/index/index.html`
- Admin dashboard: `http://localhost/liamPortfoloo/index/admin.html`

Do not open an HTML file with a `file:///C:/...` address. A file URL bypasses
Apache/PHP, so database and admin API requests cannot work.

## 7. Contact form behavior and validation

The browser sends the fields `name`, `email`, `subject`, and `message` to
`backend/contact.php`. PHP requires a `POST`, trims and validates the values, and
stores a valid message in the `messages` table.

| Field | Validation limit |
| --- | --- |
| Name | Required; up to 100 characters |
| Email | Required; up to 150 characters and must be a valid email address |
| Subject | Required; up to 200 characters |
| Message | Required; up to 5,000 characters |

Success and validation failures return JSON for the page to display. The contact
form stores messages for administrators; it does not send mail or notify the site
owner automatically.

## 8. Admin API reference

All API responses are JSON. The `resource` query parameter selects the operation.

| Method | Resource | Access | Purpose |
| --- | --- | --- | --- |
| `GET` | `session` | Public | Check whether the session is authenticated |
| `POST` | `session` | Public | Sign in with a username and password |
| `DELETE` | `session` | Authenticated + CSRF token | Sign out |
| `GET` | `portfolio` | Public | Read projects, education, and skills |
| `GET` | `messages` | Authenticated | List contact messages |
| `DELETE` | `messages&id=ID` | Authenticated + CSRF token | Delete a message |
| `GET` | `projects`, `education`, `skills` | Authenticated | List admin records |
| `POST` | `projects`, `education`, `skills` | Authenticated + CSRF token | Create a record |
| `PUT` | `projects`, `education`, `skills` | Authenticated + CSRF token | Update a record |
| `DELETE` | `projects&id=ID`, `education&id=ID`, `skills&id=ID` | Authenticated + CSRF token | Delete a record |

Admin create/update requests send JSON. The dashboard adds the CSRF token in the
`X-CSRF-Token` header for changes. IDs must be positive integers. Project links
accept HTTP(S) URLs; image values accept relative/local paths or HTTP(S) URLs.
Education years, when supplied, must be between 1901 and 2155 and an end year
cannot precede a start year.

## 9. Basic verification checklist

After setup, verify:

1. The public page loads at its HTTP localhost URL and its CSS, images, and
   JavaScript load.
2. The public projects, education, and skills sections show the database records.
3. Submitting an invalid contact form shows a validation message without adding a
   record.
4. Submitting a valid test enquiry shows success and adds one row to `messages`.
5. The admin page signs in and displays the dashboard.
6. The dashboard can read and delete the test message.
7. Creating or editing a project, education record, or skill is reflected on the
   public page after refresh.
8. Signing out returns to the sign-in panel; protected endpoints return HTTP 401
   when no admin session is present.

Delete only test records you created during verification.

## 10. Troubleshooting

### The page is blank, or changes made to files do not appear

- Confirm Apache is running in Laragon.
- Use the HTTP URL, not a `file:///` URL.
- Ensure the browser is loading the project under the expected folder name:
  `http://localhost/liamPortfoloo/index/index.html`.
- Hard-refresh the page to bypass cached JavaScript or CSS.

### “Cannot reach the admin service” or the API does not return JSON

- Open `http://localhost/liamPortfoloo/backend/admin-api.php?resource=session`.
  A working API should return JSON such as `{"authenticated":false}` when signed
  out.
- Confirm Apache is running and PHP is enabled for the project.
- Make sure the project URL uses the same host (`localhost`) on both pages and
  API requests. Do not mix `localhost` and `127.0.0.1`.
- Review the Apache error log and PHP error log for server-side failures.

### Portfolio endpoint returns HTTP 500

- Start MySQL in Laragon.
- Confirm the `digital_cv` database and all five tables exist.
- Check the database credentials and PHP's PDO MySQL extension.
- Review the PHP/Apache error log for the underlying database error.

### Login says the username or password is incorrect

- Check the exact administrator username in the `admins` table; usernames are
  case-sensitive in some database collations and should be entered as created.
- If necessary, run `backend/create_admin.php` again for that username and set a
  new password using the secure prompt above.
- Use the same `localhost` hostname throughout to keep the session cookie.

### Admin pages show HTTP 401 after signing out

This is expected for protected resources. Sign in again. If it happens
immediately after a successful login, ensure cookies are enabled and avoid
switching between `localhost`, `127.0.0.1`, HTTP, and HTTPS.

### Contact message is not saved

- Ensure MySQL is running and the `messages` table exists.
- Check the displayed validation message and all four required fields.
- Confirm the contact endpoint is reached at
  `http://localhost/liamPortfoloo/backend/contact.php`.

## 11. Security, privacy, and production notes

- Administrator passwords are stored as PHP password hashes, not plaintext.
- Never share or commit administrator passwords. Use a unique password and
  change it if it has been exposed.
- The create-admin script is command-line-only and refuses passwords shorter
  than 12 characters.
- Admin writes require authentication and a CSRF token. SQL access uses PDO
  prepared statements for user-supplied values.
- Contact messages contain personal information. Restrict dashboard access and
  keep database backups private.
- Local Laragon is for development. Before public deployment, configure HTTPS,
  production database credentials with least privilege, restricted error display,
  regular updates, and protected backups. Do not expose a development database or
  use the local root database account for a public website.
- The admin dashboard does not provide password reset by email, multi-admin
  roles, or automated message notifications.

## 12. Backup and maintenance

Before changing the schema or updating the project, export the `digital_cv`
database from HeidiSQL. Keep the export outside the public web root, and verify
that it contains the records you expect. Back up the project files separately,
including `database/`, `backend/`, `index/`, `js/`, `css/`, `images/`, `assets/`,
and `documents/`.

When restoring, import the database backup into MySQL and place the project files
back under `C:\laragon\www\liamPortfoloo`. Check the PHP database settings, start
Apache and MySQL, and run the verification checklist.

## 13. Current limitations

- The contact form saves messages in the database; it does not send email.
- Project images are referenced by URL/path; dashboard image upload is not
  implemented.
- Only the configured administrator account is supported by this workflow; there
  is no self-service account registration or password reset.
- The SQL seed content is inserted only when its respective table is empty.
- Public content updates require a working PHP/MySQL connection; static profile
  sections remain in the HTML.
