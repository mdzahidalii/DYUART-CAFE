# DYU Art Cafe — Full-Stack Cafe Web Platform & Multi-Portal System

Welcome to **DYU Art Cafe**! This application is a complete, artisanal, full-stack cafe management and customer ordering platform built with **React, Vite, Node.js, Express, MongoDB Atlas, and Secure Online Payments**.

---

## 🎨 Visual Design & Customer Experience
- **Artisanal Aesthetics**: Warm cream (`#FAF7F2`), earthy espresso text (`#2C221E`), terracotta highlights (`#C85A32`), clean whitespace, and responsive layouts.
- **Customer Friendly Language**: Technical jargon ("MongoDB", "Razorpay", database collection names) has been completely removed from all customer-facing views and replaced with clear, inviting cafe language ("Secure Online Payment", "Order Confirmed", "Fresh Daily Menu • Courtyard Kitchen").
- **Preserved Core Features**:
  - Interactive food menu with instant search and category filters.
  - Persistent shopping cart with quantities, price math, and duplicate prevention.
  - Customer checkout with dine-in courtyard table selection and takeaway options.
  - Courtyard location section with Google Maps integration and universal directions links.
  - Real-time Kitchen Display System (KDS) kanban board.
  - Admin Dashboard for menu CRUD, live sales metrics, and order tracking.

---

## 🔐 Dedicated Portals & Separate Login Pages

### 1. Customer Authentication & Dashboard
- **Customer Login (`/customer/login`)**:
  - Dedicated customer sign-in page with email and password fields.
  - Interactive show/hide password toggle.
  - Clear, user-friendly error banners on invalid credentials.
  - Seamless redirection to the Customer Dashboard upon login.
  - Quick links to Customer Registration and back to Cafe Storefront.
- **Customer Registration (`/customer/register`)**:
  - Collects customer full name, email address, 10-digit mobile number, and secure password (minimum 6 characters).
  - Validates all input client-side and server-side.
  - Stores customer records securely in MongoDB `customers` collection with salted `scrypt` password hashing.
  - Auto-authenticates and redirects directly to Customer Dashboard.
- **Customer Dashboard (`/customer/dashboard`)**:
  - Displays customer name, email, phone number, and account badge.
  - Real-time order history retrieved via `GET /api/auth/customer/orders` matching customer phone/name.
  - Shows itemized breakdowns, unit prices, dining arrangements, order status, and payment verification.
  - Single-click token revocation and sign out.
  - Auto-populates customer name and phone into Checkout for one-click ordering.

### 2. Admin Authentication & Management Portal
- **Admin Login (`/admin/login`)**:
  - Separate, dedicated staff login page at `/admin/login`.
  - Authenticates against `/api/admin/auth/login`.
  - Interactive show/hide password toggle.
  - Quick development credentials hint (`admin@dyuartcafe.com` / `Admin@DYU2026!`).
  - Redirects to `/admin/dashboard` upon verification.
- **Admin Dashboard (`/admin/dashboard`)**:
  - Protected route requiring `role: 'admin'`.
  - Operations overview with live revenue, order counts, and kitchen workload from MongoDB.
  - Full Menu CRUD: Add dishes, edit details, toggle availability, and delete dishes.
  - Multi-filter order management (search by ref, phone, name, date, payment, or kitchen status).
  - Integrated Kitchen Display System tab.

### 3. Role-Based Access Control (RBAC) & Security
- **Strict Role Isolation**:
  - Customers receive `role: 'customer'` in their signed JWT token.
  - Administrators receive `role: 'admin'` in their signed JWT token.
  - Customer tokens are strictly rejected with **HTTP 403 Forbidden** on any `/api/admin/*` endpoint.
- **Cryptographic Security**:
  - Native `crypto.scryptSync` with unique 16-byte random salts.
  - Timing-safe password comparisons against timing attacks.
  - Server-side token revocation on logout.

---

## 💳 Checkout & Online Payment Resilience

### 1. Checkout Fixes
- **No Stuck Buttons**: Fixed the submit button by ensuring `isSubmitting` is safely reset to `false` across all error paths, modal cancellations, and gateway rejections.
- **Duplicate Prevention**: Reuses existing order references during payment retries to prevent duplicate database tickets.
- **Strict Server Math**: Grand totals and tax amounts are calculated authoritatively on the backend using trusted menu catalog prices.
- **Comprehensive Validation**: Validates full name (min 2 chars), phone number (10 digits), order type, table selection, and cart items.

### 2. Payment Gateway Robustness
- **Clear Missing Key Notice**: When live/test credentials are not configured in `.env`, the server reports `isConfigured: false` and guides customers to select *Pay at Counter*, preventing runtime script crashes.
- **Cryptographic Signature Verification**: HMAC-SHA256 signature verification occurs on the server before updating order `paymentStatus` to `paid`.
- **Zero False Success**: Never displays a successful payment confirmation for an unverified or failed transaction.
- **Decoupled Workflows**: Kitchen workflow (`pending` → `preparing` → `ready` → `completed`) remains strictly independent of payment status (`unpaid` vs `paid`).

---

## 📁 Project Architecture & Routes

```
DYU ART/
├── server/
│   ├── auth.js                      # scrypt hashing, JWT generation, requireCustomerAuth & requireAdminAuth (RBAC)
│   ├── db.js                        # Singleton MongoClient with sanitized logging
│   ├── seed.js                      # Initial menu catalog seeder
│   ├── index.js                     # Express API mounting /api/menu, /api/orders, /api/payments, /api/admin, /api/auth/customer
│   ├── routes/
│   │   ├── customerAuthRoutes.js    # [NEW] Customer register, login, profile, order history, logout
│   │   ├── adminRoutes.js           # Admin auth, live stats, menu CRUD, order filtering
│   │   ├── orderRoutes.js           # Order placement & kitchen status machine
│   │   ├── paymentRoutes.js         # Razorpay create-order, verify, & config status
│   │   └── menuRoutes.js            # Public available menu catalog
│   └── createAdmin.js               # CLI admin creation tool
│
├── src/
│   ├── components/
│   │   ├── CustomerLogin.jsx        # [NEW] Customer login page (/customer/login)
│   │   ├── CustomerRegister.jsx     # [NEW] Customer registration page (/customer/register)
│   │   ├── CustomerDashboard.jsx    # [NEW] Customer orders & profile page (/customer/dashboard)
│   │   ├── AdminLogin.jsx           # [NEW] Dedicated Admin login page (/admin/login)
│   │   ├── AdminDashboard.jsx       # [UPDATED] Protected Admin management portal (/admin/dashboard)
│   │   ├── Navbar.jsx               # [UPDATED] Distinct Customer Account & Admin navigation
│   │   ├── CheckoutModal.jsx        # [UPDATED] Clean customer wording, resilient submit, retry flow
│   │   ├── OrderSuccessModal.jsx    # [UPDATED] Clean cafe receipt, no technical jargon
│   │   ├── MenuSection.jsx          # [UPDATED] Clean daily menu badges
│   │   ├── KitchenDashboard.jsx     # Courtyard Kitchen Display System (/kitchen)
│   │   ├── GoogleMapView.jsx        # Google Maps location view
│   │   └── Hero.jsx, FeaturedSection.jsx, AboutSection.jsx, ContactSection.jsx, Footer.jsx
│   ├── context/
│   │   └── CartProvider.jsx         # Cart state & localStorage sync
│   ├── services/
│   │   └── api.js                   # Customer auth, admin auth, menu CRUD, orders & payments client
│   ├── App.jsx                      # [UPDATED] React Router DOM routing
│   └── tests/
│       ├── customerAuthAndCheckout.test.js # [NEW] 11 tests for customer auth, RBAC, checkout resilience
│       ├── adminDashboard.test.js   # 11 tests for admin auth, stats, CRUD, security
│       ├── razorpayPayments.test.js # 10 tests for payments and HMAC-SHA256 signatures
│       ├── kitchenDashboard.test.js # 9 tests for KDS kanban state transitions
│       ├── googleMapsLocation.test.js # 9 tests for location config & directions
│       ├── backendAndMongo.test.js  # 8 tests for MongoDB Atlas integration
│       └── cartAndOrdering.test.js  # 7 tests for cart & ordering
│
├── .env.example
├── package.json
└── README.md
```

---

## 💻 Terminal Commands

### 1. Start the Express Backend Server
```powershell
npm.cmd run server
```
*Listens on `http://localhost:5000` with MongoDB Atlas, Customer Auth, Razorpay payments, and Admin APIs.*

### 2. Start the Frontend Development Server (Separate terminal)
```powershell
npm.cmd run dev
```
*Runs on `http://localhost:5173` with full React Router client navigation.*

### 3. Run the Complete Test Suite
```powershell
npm.cmd test
```
*Executes all 65 automated unit and integration tests across 7 test suites.*

### 4. Run Linter
```powershell
npm.cmd run lint
```
*Runs oxlint — 0 warnings, 0 errors.*

### 5. Production Build
```powershell
npm.cmd run build
```
*Runs Vite production bundle build.*

---

## 🧪 Automated Test Verification (65/65 Passed)

| Suite | Tests | Result | What it Verified |
| :--- | :---: | :---: | :--- |
| **Customer Auth & Checkout Resilience** | 11 | **PASS** | Registration validation, customer JWT token, duplicate email rejection, invalid password rejection, customer profile endpoint, **customer token 403 rejection on admin endpoints (RBAC)**, admin token validation, customer order history matching phone/name, token revocation on logout, payment config status. |
| **Admin Dashboard & Management** | 11 | **PASS** | 401 on unauthenticated access, incorrect password rejection, successful login & JWT issuance, live stats from MongoDB, dish creation & validation, customer menu synchronization, dish editing & price update, dish availability toggle, dish deletion, customer order filtering, logout token revocation. |
| **Razorpay Payment & Security** | 10 | **PASS** | Server-calculated paise amount, HMAC-SHA256 signature verification, 404/tampered rejection, idempotency, retry without duplicates. |
| **Kitchen Dashboard (KDS)** | 9 | **PASS** | 4-column kanban board, state transitions (`pending` → `preparing` → `ready` → `completed`), status immutability, auto-polling. |
| **Google Maps & Cafe Location** | 9 | **PASS** | Coordinates, address placeholders, universal directions URLs, key detection, and security isolation. |
| **MongoDB Atlas Backend** | 8 | **PASS** | Single MongoClient, repeatable menu seeding, server-calculated totals, secure sanitized error messages. |
| **Customer Ordering & Cart** | 7 | **PASS** | Quantity limits, duplicate prevention, localStorage persistence, form validation. |

---

## 🌐 Application Route Guide

- **Customer Storefront**: `http://localhost:5173/`
- **Customer Sign In**: `http://localhost:5173/customer/login`
- **Customer Registration**: `http://localhost:5173/customer/register`
- **Customer Dashboard & Orders**: `http://localhost:5173/customer/dashboard`
- **Admin Portal Sign In**: `http://localhost:5173/admin/login`
- **Admin Operations Dashboard**: `http://localhost:5173/admin/dashboard`
- **Kitchen Display System**: `http://localhost:5173/kitchen`
- **Backend Health Check**: `http://localhost:5000/api/health`
