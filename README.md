# INCOGNITO

An Online Treasure Hunt web application built with **CodeIgniter** (PHP MVC framework), featuring Facebook OAuth login, multi-level puzzle gameplay, hidden clues, story progression, and a full admin panel.

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Database Schema](#database-schema)
- [Setup & Installation](#setup--installation)
- [Configuration](#configuration)
- [User Features](#user-features)
- [Admin Features](#admin-features)
- [Clue System](#clue-system)
- [Story Mode](#story-mode)

---

## Overview

Incognito is a multi-level online treasure hunt where players log in via Facebook, solve image-based puzzles, and discover hidden clues embedded in the page (HTML source, cookies, page title, and text boxes). Admins manage levels, monitor user progress, and control the entire game lifecycle.

---

## Tech Stack

| Layer      | Technology                        |
|------------|-----------------------------------|
| Backend    | PHP 5.4+ / CodeIgniter 2.x        |
| Database   | MySQL 5.6+                        |
| Auth       | Facebook PHP SDK (OAuth 2.0)      |
| Frontend   | HTML/CSS/JS (views via CI)        |
| Sessions   | Database-backed CI sessions       |

---

## Project Structure

```
incognito/
├── index.php                   # Front controller (CodeIgniter bootstrap)
├── tables.sql                  # Database schema
├── application/
│   ├── config/
│   │   ├── database.php        # DB connection settings
│   │   ├── facebook.php        # Facebook App ID & Secret
│   │   └── decepto.php         # Master admin Facebook user ID
│   ├── controllers/
│   │   ├── pages.php           # User-facing routes (home, levels, leaderboard, clues, story)
│   │   └── shadow.php          # Admin panel routes
│   ├── models/
│   │   ├── users_model.php     # User data, level progression, answer validation
│   │   └── shadow_model.php    # Admin queries (level CRUD, user management, logs)
│   ├── views/
│   │   ├── pages/              # User-facing templates
│   │   ├── shadow/             # Admin panel templates
│   │   └── templates/          # Shared header/footer layouts
│   └── libraries/              # Facebook SDK and other custom libraries
├── assets/                     # Static assets (images, CSS, JS)
└── system/                     # CodeIgniter core (do not modify)
```

---

## Database Schema

### `userlist`
Stores registered players.

| Column         | Type          | Description                          |
|----------------|---------------|--------------------------------------|
| `userid`       | INT (PK)      | Auto-increment primary key           |
| `fb_userid`    | BIGINT        | Facebook user ID (unique)            |
| `name`         | VARCHAR(40)   | Player's name                        |
| `user_profile` | VARCHAR(100)  | Facebook profile picture URL         |
| `level`        | INT           | Current level of the player          |
| `pass_time`    | TIMESTAMP     | Time of last level completion        |
| `college`      | VARCHAR(50)   | College/institution                  |
| `emailid`      | VARCHAR(50)   | Email address                        |
| `mobileno`     | VARCHAR(15)   | Mobile number                        |
| `status`       | INT           | 0 = normal, 1 = admin, 2 = blocked   |
| `phase`        | INT           | Game phase the user is in            |

### `levels`
Defines each puzzle level.

| Column         | Type          | Description                              |
|----------------|---------------|------------------------------------------|
| `level`        | INT (unique)  | Level number                             |
| `question`     | VARCHAR(40)   | Image filename for the question          |
| `answer`       | VARCHAR(100)  | Correct answer (case-insensitive match)  |
| `finish`       | VARCHAR(40)   | Image shown on level completion          |
| `title_clue`   | VARCHAR(40)   | Clue hidden in the browser tab title     |
| `html_clue`    | VARCHAR(40)   | Clue hidden in the page HTML source      |
| `cookie_clue`  | VARCHAR(40)   | Clue hidden in a browser cookie (`EULC`) |
| `textbox_clue` | VARCHAR(40)   | Clue hidden as textbox placeholder text  |
| `active`       | INT           | 1 = active, 0 = inactive                 |
| `difficulty`   | INT           | Difficulty rating                        |

### `clues`
Stores special clues per level (up to 4 extra clues).

| Column        | Type  | Description              |
|---------------|-------|--------------------------|
| `level`       | INT   | Associated level number  |
| `clue_status` | INT   | Whether clue is unlocked |
| `clue1–4`     | TEXT  | Clue content (HTML)      |

### `answer_log`
Tracks every answer attempt.

| Column    | Type          | Description                    |
|-----------|---------------|--------------------------------|
| `user_id` | BIGINT        | Facebook user ID               |
| `name`    | VARCHAR(40)   | Player name                    |
| `level`   | INT           | Level attempted                |
| `answer`  | VARCHAR(40)   | Answer submitted                |
| `time`    | TIMESTAMP     | Time of submission             |

### `story`
Maps story images to unlock triggers.

| Column       | Type | Description                              |
|--------------|------|------------------------------------------|
| `storyid`    | INT  | Story sequence ID                        |
| `imgid`      | INT  | Image asset ID                           |
| `storylevel` | INT  | Level after which this story part unlocks|

### `ci_sessions`
CodeIgniter database-backed session storage (managed automatically).

---

## Setup & Installation

### Prerequisites

- PHP 5.4+ (with `curl`, `json`, `mcrypt`, `mysql` extensions)
- MySQL 5.6+
- Apache/Nginx with `mod_rewrite` enabled
- A Facebook Developer account and app

### Steps

1. **Clone the repository**
   ```bash
   git clone <repo-url>
   cd incognito
   ```

2. **Create the database**
   ```bash
   mysql -u root -p < tables.sql
   ```

3. **Configure the application** (see [Configuration](#configuration) below)

4. **Set up your web server** to point the document root to the project directory. Ensure `mod_rewrite` is enabled for CodeIgniter's URL routing.

5. **Set the first admin manually** — after your first login via Facebook, update your user's `status` to `1` in the `userlist` table:
   ```sql
   UPDATE userlist SET status = 1 WHERE fb_userid = YOUR_FACEBOOK_ID;
   ```

---

## Configuration

### `application/config/database.php`
```php
$db['default']['hostname'] = 'localhost';
$db['default']['username'] = 'your_db_user';
$db['default']['password'] = 'your_db_password';
$db['default']['database'] = 'incognito';
$db['default']['dbdriver'] = 'mysqli';
```

### `application/config/facebook.php`
```php
$config['appId']  = 'YOUR_FACEBOOK_APP_ID';
$config['secret'] = 'YOUR_FACEBOOK_APP_SECRET';
```

### `application/config/decepto.php`
```php
$config['masterid'] = 'YOUR_FACEBOOK_USER_ID';  // Master admin — cannot be removed by other admins
```

---

## User Features

- **Facebook Login** — OAuth 2.0 login via the Facebook PHP SDK; collects email and profile info on first login
- **Image-Based Questions** — Each level presents a puzzle as an image
- **Hidden Clues** — Clues are embedded in multiple locations (see [Clue System](#clue-system))
- **Leaderboard** — Ranks players by level and time of completion
- **Story Mode** — Unlocks story images progressively as levels are completed
- **Profile Page** — Displays player stats and progress
- **Special Clues Page** — Reveals unlocked clues for the current level

## Admin Features

- **Level Management** — Add, edit, delete, activate, deactivate, and swap levels
- **User Monitoring** — View all users, their current level, and progress
- **Answer Logs** — Browse answer attempt logs per user or per level
- **User Control** — Block/unblock users
- **Site Toggle** — Deactivate the entire site (maintenance mode)
- **Bulk Level Adjustment**:
  - Upgrade all users to a specific level
  - Downgrade all users above a level back down to that level
- **Story Management** — Add story images tied to specific level completions
- **Admin Management**:
  - Admins can promote other users to admin
  - Master admin (set in `decepto.php`) can demote/remove admins
- **Level Testing** — Test any level from the admin side without affecting game state

---

## Clue System

Each level supports up to **4 types of hidden clues** embedded in the page:

| Clue Type     | Where it hides                          | DB Column       |
|---------------|-----------------------------------------|-----------------|
| Title Clue    | Browser tab / `<title>` tag            | `title_clue`    |
| HTML Clue     | HTML source comment                     | `html_clue`     |
| Cookie Clue   | Browser cookie named `EULC`            | `cookie_clue`   |
| Textbox Clue  | `placeholder` attribute of text input  | `textbox_clue`  |

Additionally, the `clues` table supports up to 4 extra rich-text clues per level that can be toggled on/off by an admin.

---

## Story Mode

The story is told through a series of images unlocked as the player progresses. Each story entry in the `story` table specifies an image (`imgid`) and the level (`storylevel`) after which it becomes visible to the player. Admins can add new story images through the admin panel.
