# Spotted Campus

A university social platform for sharing thoughts, questions and reports (the **"spots"**) with the campus community: find lost items, discover useful information about classrooms and exams, and share your experiences.

Every spot is moderated by an administrator before publication, so the information shared stays reliable.

## Table of Contents

- [Features](#features)
- [Technologies](#technologies)
- [Architecture](#architecture)
- [Database](#database)
- [Security](#security)
- [Repository Structure](#repository-structure)
- [Installation](#installation)
- [Project Documentation](#project-documentation)
- [Team](#team)

---

## Features

### Visitor

- Browse approved spots and their categories.
- Text search and category filters, updated in real time without reloading the page.
- Registration, with instant checking of username availability.
- "About us" page with the mission and the team.

### Registered User

- **Spot creation** with title, text, category and subcategory, with the option to publish **anonymously**.
- **Editing and deleting** of one's own spots: every edit sends the spot back to pending approval.
- **Comments and replies** to comments, submitted asynchronously.
- **Favorites**: save useful spots and browse them, sorted, on a dedicated page.
- **Notification center** with an unread counter, updated automatically every 30 seconds. Alerts are generated when someone comments on your spot or replies to one of your comments.
- **Personal profile** with a customizable profile picture and the history of one's own spots.

### Administrator

- **Review panel**: approve or reject pending spots.
- **User management**: block and unblock accounts; blocked users cannot log in.
- **Comment moderation**: deletion of inappropriate comments.
- **Category management**: add new categories and subcategories.
- **Statistics**: spots by moderation status, active users, number of anonymous spots, most active user, most popular category, most favorited spot and user with the most comments.

---

## Technologies

| Layer | Technologies |
| --- | --- |
| Backend | PHP, with the `mysqli` extension |
| Database | MySQL / MariaDB (InnoDB, referential integrity constraints) |
| Frontend | HTML, CSS, JavaScript (vanilla, `fetch` for asynchronous calls) |
| UI | Bootstrap 5.3 and Bootstrap Icons (via CDN), custom styles in `css/style.css` |
| Web server | Apache (404 page handled through `.htaccess`) |
| Design | Mobile and desktop mockups, Lunacy project file (`SPOTTED.lun`) |

---

## Architecture

The application follows a structure of PHP pages with a shared template and a single data access layer:

- **PHP pages** in the root (`login.php`, `dettaglio-spot.php`, `revisione.php`, ...): each handles one feature, checks permissions and prepares the data.
- **Templates** (`template/`): the `base.php` file defines the common layout (role-aware navigation bar, feedback messages, footer), while the individual views fill in the content through the `$templateParams` parameter.
- **Data access** (`db/database.php`): the `DatabaseHelper` class centralizes all SQL queries, all of them parameterized.
- **Utilities** (`utils/`): shared functions for session, roles and feedback messages, plus the registration logic.
- **Initialization** (`bootstrap.php`): starts the session and creates the database connection, and is included by every page.
- **Asynchronous endpoints** (`aggiungi-commento.php`, `toggle-preferiti.php`, `check-username.php`, `get-subcategories.php`, `notifiche-count.php`): return data used by the scripts in `js/` to update the interface without reloading the page.

## Database

The relational schema, defined in `db/spotted.sql`, consists of:

| Table | Content |
| --- | --- |
| `TIPI_UTENTI` | Roles: administrator and user |
| `UTENTI` | Credentials (hashed), personal details, status (`attivo` / `bloccato`), profile picture |
| `CATEGORIE`, `SOTTOCATEGORIE` | Two-level classification of spots |
| `SPOT` | Title, text, status (`in_attesa` / `approvato` / `rifiutato`), anonymity, author, approving admin and approval date |
| `PREFERITI` | Many-to-many relationship between users and spots |
| `COMMENTI` | Comments on spots, with a self-reference for replies |
| `NOTIFICHE` | Per-user alerts, with link and read status |

The initial categories (`db/popolaDB.sql`) are: *Info Aule* (classroom info), *Oggetti Smarriti* (lost items), *Info Universitarie* (university info) and *Persone* (people), each with its own subcategories.

---

## Security

- **Parameterized queries** (prepared statements) across the whole data access layer, against SQL injection.
- **Password hashing** with `password_hash` and verification with `password_verify`; minimum length of 8 characters at registration.
- **Output escaping** of comments with `htmlspecialchars`, against XSS.
- **Role-based access control**: restricted pages and endpoints check the session and redirect unauthorized users; spot editing is allowed only to their author.
- **Account blocking**: users blocked by the administrator cannot log in.
- **Upload validation**: the profile picture must be an image (JPG, JPEG, PNG or GIF) of at most 5 MB, with a server-generated file name.

---

## Repository Structure

```text
.
├── css/                        Custom styles
├── db/
│   ├── database.php            DatabaseHelper class (data access)
│   ├── spotted.sql             Database schema
│   └── popolaDB.sql            Sample seed data
├── js/                         Client-side scripts
│   ├── main.js                 Notifications, favorites, delete confirmations
│   ├── lista-spot.js           Asynchronous search and filters
│   ├── commenti.js             Posting comments and replies
│   ├── gestione-spot.js        Dynamic subcategories
│   ├── signup.js               Username check
│   └── profilo.js              Profile picture change
├── template/                   Views and shared layout (base.php)
├── upload/                     Profile pictures uploaded by users
├── utils/                      Helper functions and registration logic
├── bootstrap.php               Session and database initialization
├── index.php                   Home page
├── login.php, sign-up.php, logout.php
├── lista-categoria.php, ricerca.php, dettaglio-spot.php
├── gestione-spot.php, salva-spot.php
├── preferiti.php, toggle-preferiti.php
├── aggiungi-commento.php, elimina-commento.php
├── notifiche.php, notifiche-count.php
├── profilo.php, upload-profile-pic.php
├── revisione.php, gestione-utenti.php
├── admin-stats.php, inserisci-categoria.php
├── Mockup.pdf, Mockup desktop.pdf
├── design-spotted campus.pdf
└── SPOTTED.lun                 Graphic design project (Lunacy)
```

---

## Installation

### Requirements

- An environment with Apache, PHP (with the `mysqli` extension) and MySQL or MariaDB, for example [XAMPP](https://www.apachefriends.org/), MAMP or a LAMP stack.

### Steps

1. **Copy the project** into the web server's public folder (for example `htdocs` in XAMPP). The `.htaccess` file refers to the path `/WebTechProg/404.php`: if you use a different folder name, update that line.

2. **Create the database** by importing the schema and, optionally, the sample data. The `spotted.sql` script drops and recreates the `spotted` database, so any existing data will be lost.

   ```bash
   mysql -u root -p < db/spotted.sql
   mysql -u root -p < db/popolaDB.sql
   ```

   Alternatively, import the two files from phpMyAdmin.

3. **Check the connection parameters** in `bootstrap.php`. The defaults are those of a typical local environment:

   ```php
   new DatabaseHelper("localhost", "root", "", "spotted", 3306);
   ```

4. **Make sure the `upload/` folder is writable** by the web server, since it stores the profile pictures.

5. **Start Apache and MySQL** and open `http://localhost/<folder-name>/index.php`.

---

## Project Documentation

- [`design-spotted campus.pdf`](design-spotted%20campus.pdf): functional and non-functional requirements, personas and usage scenarios.
- [`Mockup.pdf`](Mockup.pdf): mockups of the mobile version.
- [`Mockup desktop.pdf`](Mockup%20desktop.pdf): mockups of the desktop version.
- `SPOTTED.lun`: source file of the graphic design project, which can be opened with [Lunacy](https://icons8.com/lunacy).

---
