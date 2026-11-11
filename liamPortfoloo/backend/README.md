# Admin setup

The portfolio and admin dashboard require Laragon's Apache, PHP with PDO MySQL enabled,
and the `digital_cv` MySQL database. Start Apache and MySQL from Laragon, then open
`http://localhost/liamPortfoloo/` for the public site or
`http://localhost/liamPortfoloo/index/admin.html` for the admin dashboard. Do not open
the HTML files with a `file://` URL; PHP and database requests need Apache.

1. Import `database/digital_cv.sql` into MySQL using Laragon's HeidiSQL
   (Laragon menu → MySQL → HeidiSQL). The script creates the required
   tables, including the editable skills catalog, and adds starter portfolio
   content only when the corresponding tables are empty. To add the skills
   feature to an existing installation, import the updated SQL file again; it
   creates and seeds the skills table without replacing existing records.
2. The defaults in `db.php` are `localhost`, `digital_cv`, `root`, and an empty
   password. Set `DB_HOST`, `DB_NAME`, `DB_USER`, and `DB_PASS` for PHP if your
   local MySQL credentials differ.
3. In Laragon's Terminal, change to the project directory and create the `MCKEEVER`
   admin account from PowerShell.
   The command securely prompts for a password of at least 12 characters:

   ```powershell
   Set-Location C:\laragon\www\liamPortfoloo
   $securePassword = Read-Host "Choose an admin password (12+ characters)" -AsSecureString
   $secretPointer = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($securePassword)
   try {
     $env:ADMIN_PASSWORD = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($secretPointer)
     php backend/create_admin.php
   } finally {
     Remove-Item Env:ADMIN_PASSWORD -ErrorAction SilentlyContinue
     [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($secretPointer)
   }
   ```

   Running the script again resets the `MCKEEVER` account's password. To use a
   different username, set `ADMIN_USERNAME` before running the script.
4. Visit `http://localhost/liamPortfoloo/index/admin.html` and sign in. The dashboard can review/delete contact
   messages and create, edit, or delete projects, education records, and skills.
   Portfolio content changes appear on the public site.

Use HTTPS when the site is deployed publicly so the admin session cookie is secure.
