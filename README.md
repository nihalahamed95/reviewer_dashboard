# Reviewer Matching Board
**Full-Stack Assessment: PHP & Modern Angular (v18+)**

A single-page editorial dashboard designed for academic journal editors to review pending manuscripts, browse potential peer reviewers from an academic dataset (`mock_openalex_data.json`), enforce strict institutional conflict-of-interest policies, manage Master datasets (Articles & Reviewers), and update account credentials.

---

## 📋 Table of Contents
1. [Project Directory & Location](#-project-directory--location)
2. [Step-by-Step Setup Guide](#-step-by-step-setup-guide)
   * [Step 1: Start PHP API (XAMPP Apache)](#step-1-start-php-api-via-xampp-apache)
   * [Step 2: Start Angular Frontend](#step-2-start-angular-frontend)
3. [Master Data Management (Articles & Reviewers)](#-master-data-management)
4. [Authentication & Password Management](#-authentication--password-management)
5. [Core Business Rule (Institutional Eligibility)](#-core-business-rule-institutional-eligibility)
6. [REST API Documentation](#-rest-api-documentation)
7. [Project Structure](#-project-structure)
8. [Automated Verification Suite](#-automated-verification-suite)

---

## 📁 Project Directory & Location

Ensure this entire project folder is located inside your XAMPP `htdocs` directory:

```
C:\xampp\htdocs\straive_task\
```

---

## 🚀 Step-by-Step Setup Guide

### Step 1: Start PHP API via XAMPP Apache
You do **not** need to run any manual `php -S` command. XAMPP Apache serves the API automatically.

1. Open the **XAMPP Control Panel**.
2. Click the **Start** button next to **Apache** (Port 80/443).
3. Test that the API is running by opening this URL in your web browser:
   ```
   http://localhost/straive_task/api/articles
   ```
   *Expected result: You will see the JSON response of academic articles.*

---

### Step 2: Start Angular Frontend

1. Open your terminal (PowerShell or Command Prompt).
2. Navigate to the project root directory:
   ```powershell
   cd C:\xampp\htdocs\straive_task
   ```
3. Install dependencies (first time only):
   ```powershell
   npm install
   ```
   *(On Windows PowerShell, you can use `npm.cmd install` if execution policy prompts)*

4. Start the development server:
   ```powershell
   npm start
   ```
   *(or: `npx ng serve`)*

5. Open your browser and visit:
   ```
   http://localhost:4200
   ```

---

## 📚 Master Data Management

Accessible via the slim left navigation rail:

### 1. 📄 Article Master
* **Access:** Click the **📄 Articles** icon on the left dock rail.
* **Features:**
  * View all registered manuscripts with author, affiliation, and reviewer assignment status.
  * **Add Article:** Click **"➕ Add New Article"** to register a title, author name, and author university. Saves immediately to `api/data/articles.json` and updates stats.
  * **Delete Article:** Click **"🗑️ Delete"** to delete an article and cascade-remove its assignments.

### 2. 👥 Reviewer Master
* **Access:** Click the **👥 Reviewers** icon on the left dock rail.
* **Features:**
  * View all peer reviewers, their universities, and domain keyword chips.
  * **Add Reviewer:** Click **"➕ Add New Reviewer"** to register reviewer full name, university, and comma-separated expertise keywords. Saves to `api/data/reviewers.json`.
  * **Delete Reviewer:** Click **"🗑️ Delete"** to remove a reviewer and update assignment statuses.

---

## 🔐 Authentication & Password Management

### 1. Dedicated Login Page
* **Route/View:** When not authenticated, users see a dedicated, full-screen login page (no popups over the dashboard).
* **Demo Email:** `editor@straive.com`
* **Demo Password:** `admin123`
* **1-Click Access:** Click **"⚡ 1-Click Demo Login"** on the page.
* **Clean Logout:** Clicking **"🚪 Sign Out"** immediately clears the JWT session, unmounts the workspace dashboard from the DOM, and returns to the dedicated login page.
* **JWT Interceptor:** An Angular functional HTTP interceptor (`src/app/core/interceptors/jwt.interceptor.ts`) attaches `Authorization: Bearer <token>` to all API requests.

### 2. ⚙️ Settings & System Preferences
* **Access:** Click the **⚙️ Gear** icon on the dock rail or select **"⚙️ Settings & Data"** from the profile menu.
* **Account Security (Change Password):** Current Password, New Password (minimum 6 characters), and Confirm New Password. Persisted in `api/data/users.json`.
* **📧 Email Notifications on Assignment:** Toggle whether referees automatically receive email notifications when assigned to a manuscript.
* **🔄 Restore Default Mock JSON Data:** Resets all articles, reviewers, and reviewer assignments back to the initial `mock_openalex_data.json` dataset.

---

## ⚖️ Core Business Rule (Institutional Eligibility)

> **Specification Rule:** *The system must block the assignment if the selected reviewer and the article's author belong to the exact same university or institution.*

### 1. Dataset Scenario
* **Article `art_101`**: *"Optimizing State Management in Large Angular Applications"*
  * **Author:** David Smith • **Institution:** Stanford University
* **Reviewer `rev_001`**: **Dr. Alice Chen** • **Institution:** Stanford University
  * **Result:** **INELIGIBLE TO ASSIGN** (*Both belong to Stanford University*)

### 2. Frontend Enforcement
* The status badge displays **`🚫 Ineligible to assign`**.
* The **Assign Reviewer** button is **disabled** (`disabled="true"`, muted appearance, non-clickable).
* Directly below the reviewer card, the explicit reason is shown:
  > **Reason:** Ineligible to assign because author (David Smith) and reviewer (Dr. Alice Chen) both belong to **Stanford University**.

### 3. Backend Enforcement (`api/index.php`)
* If an assignment request is dispatched for a matching institution pair, the server strictly blocks it and returns **`HTTP 409 Conflict`**:
  ```json
  {
    "success": false,
    "status_code": 409,
    "error": "Conflict of Interest: Author (David Smith) and Reviewer (Dr. Alice Chen) both belong to 'Stanford University'."
  }
  ```

---

## 📡 REST API Documentation

* **Base URL:** `http://localhost/straive_task/api`

| Method | Endpoint | Description | Sample Response Status |
|---|---|---|---|
| `GET` | `/articles` | Returns list of academic articles | `200 OK` |
| `POST` | `/articles` | Creates a new manuscript in Article Master | `201 Created` |
| `POST` | `/articles/delete` | Deletes a manuscript by ID | `200 OK` |
| `GET` | `/reviewers` | Returns list of peer reviewers | `200 OK` |
| `POST` | `/reviewers` | Registers a new peer reviewer in Reviewer Master | `201 Created` |
| `POST` | `/reviewers/delete` | Deletes a reviewer by ID | `200 OK` |
| `POST` | `/assign` | Assigns reviewer to article with institutional conflict validation | `201 Created` / `409 Conflict` |
| `POST` | `/login` | Authenticates user and issues JWT Bearer token | `200 OK` / `401 Unauthorized` |
| `POST` | `/change-password` | Updates account password in persistent store | `200 OK` / `400 Bad Request` |
| `GET` | `/stats` | Returns real-time metrics (counts, conflicts detected, assignments) | `200 OK` |
| `POST` | `/reset` | Clears all stored assignments back to default | `200 OK` |

---

## 📂 Project Structure

```
C:\xampp\htdocs\straive_task\
├── api/                               # PHP REST API (Served by XAMPP Apache)
│   ├── data/
│   │   ├── articles.json             # Persistent articles database
│   │   ├── reviewers.json            # Persistent reviewers database
│   │   ├── assignments.json          # Persistent assignments database
│   │   └── users.json                # User accounts & passwords
│   ├── .htaccess                     # Apache mod_rewrite clean routing
│   ├── index.php                     # REST router, Master CRUD & conflict logic
│   └── verify.php                    # Automated test verification suite
├── src/                               # Angular 18 Standalone Frontend
│   ├── app/
│   │   ├── components/
│   │   │   ├── change-password/      # Change password modal dialog
│   │   │   ├── login/                # Login modal with 1-click demo access
│   │   │   └── toast/                # Floating notification alerts
│   │   ├── core/
│   │   │   └── interceptors/         # JWT Bearer token HTTP interceptor
│   │   ├── models/                   # Clean TypeScript interfaces
│   │   ├── services/
│   │   │   ├── auth.service.ts       # JWT token & password management
│   │   │   ├── reviewer-matching.ts  # Master CRUD, API & eligibility logic
│   │   │   └── toast.service.ts      # Toast notification system
│   │   ├── app.component.ts          # Master-detail & tab navigation controller
│   │   ├── app.component.html        # Clean Lumina template with Master views
│   │   ├── app.component.css         # Modern styling & master tables
│   │   └── app.config.ts             # Application config with HTTP client & interceptors
│   ├── index.html
│   ├── main.ts
│   └── styles.css
├── angular.json
├── package.json
├── proxy.conf.json                   # Reverse proxy configuration
├── mock_openalex_data.json           # Raw Academic Mock API dataset
└── README.md                         # Complete documentation & guide
```

---

## 🧪 Automated Verification Suite

Run the automated backend test suite from your terminal:

```powershell
& "C:\xampp\php\php.exe" api/verify.php
```

All 8 tests will report: `✅ PASSED`:
1. `Mock openalex data exists & reads articles/reviewers`
2. `JWT HMAC-SHA256 Token creation & signature validation`
3. `Stanford Lead Author vs Stanford Referee triggers Conflict of Interest`
4. `Stanford Lead Author vs MIT Referee does NOT trigger Conflict of Interest`
5. `Storage directory & data files writable`
6. `Article Master: add and remove simulated article`
7. `Reviewer Master: add and remove simulated reviewer`
8. `Password storage & change logic`
