# PHP Software Licensing System

A centralized web-based application built on Laravel 11 and Filament v3 designed to manage software licenses for multiple applications. This system controls remote software activations, validates license usage, enforces subscription tiers, handles grace periods, and logs full audit trails.

---

## 🚀 Key Features

*   **Customer Management**: Manage client details, contact information, subscription status, and view license histories.
*   **Product Management**: Organize multiple software offerings, versions, and descriptions.
*   **License Management**: Generate license keys, define activation limits, set plans, configure expiry, and track status.
*   **Remote Software Activation API**: Idempotent endpoint for register-once client devices with activation limits.
*   **Remote Validation API**: Periodically verifies key validity, checks grace periods, and returns features mapped to subscription plans.
*   **Deactivation & Force Reactivate**: Terminate installations either programmatically or from the admin panel (forcing client reactivation).
*   **Subscription & Expiry Engine**: Automatable CLI command scanning expiring contracts, warning users, and updating statuses.
*   **Admin Dashboard Widgets**: Visual statistics overview (total customers, active/expired licenses, estimated Monthly Recurring Revenue) and a feed of recent activations.
*   **Audit Logging**: Automatic event tracking of logins, license creation, activation, validation, deactivation, and plan upgrades.

---

## 🛠️ Technology Stack

*   **Backend Framework**: Laravel 11 (PHP 8.2+)
*   **Admin Portal**: Filament v3 (Tailwind CSS)
*   **API Security**: Laravel Sanctum (Token authentication for admin actions)
*   **Database**: MySQL 8+
*   **Asset Bundling**: Vite 6

---

## ⚙️ Installation & Setup

1.  **Clone the Repository & Navigate**:
    ```bash
    cd /Users/muhammadqudoos/php-licensing-system
    ```

2.  **Configure Environment**:
    Copy the example configuration file:
    ```bash
    cp .env.example .env
    ```
    Ensure your database credentials are correctly set in `.env` (e.g., `DB_DATABASE=licensing_db`, `DB_USERNAME`, `DB_PASSWORD`).

3.  **Install PHP Dependencies**:
    ```bash
    composer install
    ```

4.  **Install Node Dependencies & Compile Assets**:
    ```bash
    npm install
    npm run build
    ```

5.  **Run Migrations & Seed Database**:
    Wipe database, run migrations chronologically, and seed test data:
    ```bash
    php artisan migrate:fresh --seed
    ```

6.  **Run the Local Server**:
    ```bash
    php artisan serve
    ```
    Open your browser and navigate to `http://127.0.0.1:8000/admin`.
    *   **Login Email**: `admin@example.com`
    *   **Login Password**: `password`

---

## 📡 REST API Documentation

### 1. User Login (Admin Authentication)
*   **URL**: `POST /api/auth/login`
*   **Headers**: `Content-Type: application/json`
*   **Body**:
    ```json
    {
      "email": "admin@example.com",
      "password": "password"
    }
    ```
*   **Response (200)**: Returns bearer `token` for secured lookups.

### 2. Activate License
*   **URL**: `POST /api/licenses/activate` (Public)
*   **Headers**: `Content-Type: application/json`
*   **Body**:
    ```json
    {
      "license_key": "LIC-TESTKEY123",
      "installation_id": "inst-device-1",
      "domain": "client-site.com",
      "software_version": "1.0.0"
    }
    ```
*   **Response (200)**:
    ```json
    {
      "status": "success",
      "message": "License activated successfully",
      "expiry_date": "2027-06-16",
      "activation_token": "7-1781639503"
    }
    ```

### 3. Validate License
*   **URL**: `POST /api/licenses/validate` (Public)
*   **Headers**: `Content-Type: application/json`
*   **Body**:
    ```json
    {
      "license_key": "LIC-TESTKEY123",
      "installation_id": "inst-device-1"
    }
    ```
*   **Response (200)**:
    ```json
    {
      "status": "valid",
      "expiry_date": "2027-06-16",
      "subscription_status": "active",
      "allowed_features": ["core", "reporting", "api_access"],
      "message": "License is valid"
    }
    ```

### 4. Deactivate License
*   **URL**: `POST /api/licenses/deactivate` (Public)
*   **Headers**: `Content-Type: application/json`
*   **Body**:
    ```json
    {
      "license_key": "LIC-TESTKEY123",
      "installation_id": "inst-device-1"
    }
    ```
*   **Response (200)**:
    ```json
    {
      "status": "success",
      "message": "Installation deactivated successfully"
    }
    ```

### 5. Fetch License Information (Admin Only)
*   **URL**: `GET /api/licenses/{license_key}` (Protected)
*   **Headers**: `Authorization: Bearer <token>`
*   **Response (200)**: Returns complete License, Customer, and Product relationships.

---

## 🤖 Console Commands

To automate license expiry updates and warnings, add the following to your system CRON task runner to trigger daily:
```bash
php artisan app:check-license-expiry
```
*   Marks licenses past their grace periods as `expired` in database.
*   Sends email reminders to customers who are expiring in under 7 days or are currently within their grace periods.

---

## 🧪 Running Tests

To verify API endpoints and business logic, run the test suite:
```bash
vendor/bin/phpunit
```
This runs 10 integration tests validating authentication, activation limits, idempotency, validations, and deactivations.
