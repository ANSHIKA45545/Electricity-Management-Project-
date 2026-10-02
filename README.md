# ⚡ Green Energy Open Access (GOA) Portal

A full-stack web portal for managing **Green Open Access (GOAR / NOAR)** electricity transactions between **Consumers**, **Suppliers** (renewable generators) and **Administrators**. It covers K-number based consumer onboarding, supplier registration, bidding on green energy, plant listings, open-access applications, documents, payments and market prices, in a multi-language interface.

| Layer      | Technology                               | Hosted on |
| ---------- | ---------------------------------------- | --------- |
| Frontend   | React + TypeScript + Vite + Tailwind CSS | Netlify   |
| Backend    | Node.js + Express + TypeScript           | Render    |
| Database   | PostgreSQL via Prisma ORM                | Neon      |

**Live backend:** `https://electricity-management-project.onrender.com`
**Live frontend:** `https://greenenergyoa.netlify.app/` 

---

## Table of Contents

1. [Features](#features)
2. [Architecture](#architecture)
3. [Project Structure](#project-structure)
4. [Tech Stack](#tech-stack)
5. [Getting Started (Local)](#getting-started-local)
6. [Environment Variables](#environment-variables)
7. [Database](#database)
8. [Demo / Test Data](#demo--test-data)
9. [API Reference](#api-reference)
10. [Authentication and Roles](#authentication-and-roles)
11. [Deployment](#deployment)
12. [Troubleshooting](#troubleshooting)
13. [Security Notes](#security-notes)
14. [Scripts](#scripts)
15. [Contributing](#contributing)

---

## Features

**Public**
- Landing page, regulations page and a charges calculator for open-access charges.
- Multi-language interface (i18next, with a language switcher and Google Translate wrapper).

**Authentication**
- **K-number verification:** users verify the K-number printed on their electricity bill before registering or signing in. K-numbers are checked against an `electricboard` table of registered connections.
- **DISCOM verification:** registration checks that the K-number belongs to the DISCOM the user selects.
- **Consumer registration** via K-number: name, mobile number and connection type are pulled from the electricity-board record. The user only supplies an email and password, and is signed in immediately.
- **Supplier registration** (renewable energy generators).
- **Sign in** with email and password, or with an **email OTP** (sent through Gmail/Nodemailer).
- **Separate admin login** with role enforcement.
- JWT-based sessions (24 hours for password login, 7 days for OTP login).

**Consumer**
- Consumer dashboard.
- Create and manage energy **bids** (MW, price, duration, drawal point, schedule type, renewable type).
- Browse suppliers and supplier details.
- Open-access applications, documents and payments.

**Supplier**
- Supplier dashboard.
- Manage **plants** (capacity, available capacity, price, injection point, status).
- Submit **offers** against consumer bids.
- Upload and track supplier documents.

**Admin**
- Admin dashboard.
- User and document status management.
- Application approval workflow.

---

## Architecture

```
┌────────────────────┐   HTTPS (fetch)    ┌─────────────────────┐   Prisma    ┌──────────────┐
│  React SPA (Vite)  │ ─────────────────▶ │  Express API (TS)   │ ──────────▶ │ PostgreSQL   │
│  Netlify           │  VITE_API_URL      │  Render             │             │ Neon         │
└────────────────────┘                    └─────────────────────┘             └──────────────┘
                                                    │
                                                    └── Nodemailer (Gmail) for email OTP
```

The frontend reads the backend address from `VITE_API_URL` **at build time**.

---

## Project Structure

```
ElectricTool/
├── netlify.toml                  # Netlify build config (base: frontend)
├── .gitignore
├── backend/
│   ├── package.json
│   ├── tsconfig.json
│   ├── prisma/
│   │   ├── schema.prisma         # Database models
│   │   ├── seed.ts               # Seed script
│   │   └── dummy_data.sql        # Demo / test data (add this file here)
│   ├── uploads/                  # Uploaded files (served at /uploads)
│   └── src/
│       ├── server.ts             # Express app entry point
│       ├── config/db.ts          # Prisma client + data-access helpers
│       ├── middleware/auth.ts    # JWT auth + user verification
│       ├── services/otpService.ts# Email/phone OTP generation and verification
│       └── routes/
│           ├── auth.routes.ts    # Register, login, OTP, K-number validation
│           ├── user.routes.ts
│           ├── supplier.routes.ts
│           ├── plant.routes.ts
│           ├── bid.routes.ts
│           ├── app.routes.ts     # Applications
│           ├── doc.routes.ts     # Documents
│           ├── pay.routes.ts     # Payments
│           └── market.routes.ts  # Market prices
└── frontend/
    ├── package.json
    ├── vite.config.ts
    ├── index.html
    └── src/
        ├── App.tsx
        ├── main.tsx
        ├── context/
        │   ├── AuthContext.tsx           # Session state (localStorage)
        │   ├── LanguageContext.tsx
        │   ├── LanguageSwitcher.tsx
        │   └── GoogleTranslateWrapper.tsx
        └── components/
            ├── auth/       # AuthPages.tsx, AdminAuthPages.tsx
            ├── landing/    # LandingPage, RegulationsPage, ChargesCalculator
            ├── dashboard/  # Admin, Consumer and Supplier dashboards
            ├── consumers/  # Supplier listing and detail views
            └── layout/     # DashboardLayout
```

---

## Tech Stack

**Frontend:** React 19, TypeScript, Vite, Tailwind CSS 4, lucide-react, i18next / react-i18next.

**Backend:** Node.js, Express 4, TypeScript, Prisma 6, bcryptjs, jsonwebtoken, Multer (uploads), Nodemailer, CORS.

**Database:** PostgreSQL (Neon, pooled connection).

---

## Getting Started (Local)

### Prerequisites
- Node.js 18 or newer
- npm
- A PostgreSQL database (a free [Neon](https://neon.tech) project works well)

### 1. Clone

```bash
git clone <your-repo-url>
cd ElectricTool
```

### 2. Backend

```bash
cd backend
npm install            # also runs `prisma generate`
```

Create `backend/.env` (see [Environment Variables](#environment-variables)), then create the tables:

```bash
npx prisma db push
```

Optionally seed data, then start the server:

```bash
npm run prisma:seed    # optional
npm run dev            # http://localhost:5000
```

Open `http://localhost:5000/`. You should see `{"status":"ONLINE", ...}`.

### 3. Frontend

```bash
cd ../frontend
npm install
```

Create `frontend/.env`:

```
VITE_API_URL=http://localhost:5000
```

```bash
npm run dev            # http://localhost:5173
```

---

## Environment Variables

### Backend (`backend/.env`, and the Render dashboard in production)

| Variable       | Required     | Description                                                                 |
| -------------- | ------------ | --------------------------------------------------------------------------- |
| `DATABASE_URL` | Yes          | PostgreSQL connection string (see the Neon notes below)                     |
| `JWT_SECRET`   | Yes (prod)   | Long random string used to sign and verify JWTs                             |
| `EMAIL_USER`   | For OTP      | Gmail address used to send OTP emails                                       |
| `EMAIL_PASS`   | For OTP      | Gmail **App Password** (not your normal password)                           |
| `EMAIL_FROM`   | Optional     | Custom "from" header, e.g. `"Green Energy Portal" <you@gmail.com>`          |
| `PORT`         | Optional     | Defaults to `5000` (Render sets this automatically)                         |

**Neon connection string format:**

```
postgresql://USER:PASSWORD@ep-xxxx-pooler.REGION.aws.neon.tech/DBNAME?sslmode=require&pgbouncer=true&connect_timeout=30
```

### Frontend (`frontend/.env`, and the Netlify dashboard in production)

| Variable       | Required | Description                                                                                  |
| -------------- | -------- | -------------------------------------------------------------------------------------------- |
| `VITE_API_URL` | Yes      | Base URL of the backend, with no trailing slash and no `/api`. Falls back to `http://localhost:5000` if unset. |

> `.env` files are git-ignored. Never commit real credentials.

---

## Database

Defined in `backend/prisma/schema.prisma` (PostgreSQL).

| Model              | Purpose                                                                    |
| ------------------ | -------------------------------------------------------------------------- |
| `User`             | Accounts: email, hashed password, role, K-number, phone, status            |
| `ConsumerProfile`  | Consumer drawal point and phone                                            |
| `SupplierProfile`  | Supplier injection point, renewable type, capacity, registration number    |
| `electricboard`    | Master list of valid K-numbers with connection, DISCOM and load details    |
| `Bid`              | Consumer energy bids                                                       |
| `BidOffer`         | Supplier offers made against bids                                          |
| `Plant`            | Supplier generation plants                                                 |
| `Application`      | Open-access applications with annexure and approval status                 |
| `Contract`         | Contracts generated from approved applications                             |
| `SupplierDocument` | Documents uploaded by suppliers                                            |
| `Document`         | General user documents                                                     |
| `Payment`          | Payment records                                                            |
| `Schedule`         | Energy schedules per time block                                            |
| `MarketPrice`      | Market price entries                                                       |

**Common commands**

```bash
cd backend
npx prisma generate    # regenerate the Prisma client
npx prisma db push     # create or update tables from schema.prisma
npx prisma studio      # browse data in the browser
```

> The project uses `prisma db push` (no migrations folder). Run it against any new database before first use.
> **Registration requires data in the `electricboard` table.** A K-number must exist there before anyone can register.

---

## Demo / Test Data

A ready-made SQL script, `backend/prisma/dummy_data.sql`, lets you try the portal without real K-numbers.

**How to load it**
1. Make sure the tables exist (`npx prisma db push`).
2. Open your Neon project, go to **SQL Editor**, paste the contents of `dummy_data.sql` and click **Run**.
3. The script is safe to run more than once.


**(optional): ready-made demo accounts.**

| Role     | Sign in with                                            
| -------- | --------------------------------------------------------- 
| Consumer | K-number `320223020301`, password : `qwertyuiop`
| Supplier | K-number `320223020298`,password : `qwertyuiop`


---

## API Reference

Base URL: `https://electricity-management-project.onrender.com` (or `http://localhost:5000` locally).

### Public

| Method | Endpoint | Description |
| ------ | -------- | ----------- |
| GET    | `/`      | Health check (`status: ONLINE`) |

### Auth (`/api/auth`), no token required unless noted

| Method | Endpoint                               | Body                                         | Description |
| ------ | -------------------------------------- | -------------------------------------------- | ----------- |
| POST   | `/register`                            | `email, password, name, role, ...`           | Register a SUPPLIER (or CONSUMER). `ADMIN` is rejected. |
| POST   | `/validate-knumber-for-registration`   | `k_number, discom?`                          | Check the K-number exists, matches the DISCOM and is not yet registered |
| POST   | `/register-with-knumber`               | `k_number, email, password`                  | Register a consumer from K-number data and return a JWT |
| POST   | `/validate-knumber`                    | `k_number`                                   | Check a registered user exists for a K-number (login step 1) |
| POST   | `/login`                               | `email, password`                            | Password login, returns JWT and user |
| POST   | `/send-email-otp`                      | `email`                                      | Send a 6-digit OTP by email (valid 10 minutes) |
| POST   | `/verify-email-otp`                    | `email, otp`                                 | Verify the OTP and return JWT and user |
| POST   | `/send-phone-otp`                      | `phone`                                      | Generate a phone OTP (SMS sending is not implemented) |
| POST   | `/verify-phone-otp`                    | `phone, otp`                                 | Verify the phone OTP |
| POST   | `/admin-login`                         | `email, password`                            | Admin-only login |
| GET    | `/me`                                  | Header `Authorization: Bearer <token>`       | Current user and profile |

### Protected (require `Authorization: Bearer <token>`)

| Prefix              | Purpose                                  |
| ------------------- | ---------------------------------------- |
| `/api/users`        | User management                          |
| `/api/market`       | Market prices                            |
| `/api/applications` | Open-access applications                 |
| `/api/documents`    | Documents                                |
| `/api/payments`     | Payments                                 |
| `/api/suppliers`    | Supplier data                            |

Also mounted: `/api/plants`, `/api/bids` and `/uploads` (static file serving).

---

## Authentication and Roles

- Passwords are hashed with **bcrypt**.
- Sessions use **JWT** sent as `Authorization: Bearer <token>`.
- The frontend stores the session in `localStorage` (`goar_token`, `goar_user`).
- Roles: `CONSUMER`, `SUPPLIER`, `ADMIN`.
- Admin accounts cannot be created through public registration. Create them directly in the database (see [Demo / Test Data](#demo--test-data) for an example).
- Middleware (`middleware/auth.ts`) validates the token and confirms the user still exists.

---

## Deployment

### Database: Neon
1. Create a Neon project and copy the **pooled** connection string.
2. Append `?sslmode=require&pgbouncer=true&connect_timeout=30`.
3. Run `npx prisma db push` once from `backend/` with that `DATABASE_URL`.
4. Import your K-number data into `electricboard` (or load the demo data above for testing).

### Backend: Render (Web Service)
| Setting         | Value                                |
| --------------- | ------------------------------------ |
| Root Directory  | `backend`                            |
| Build Command   | `npm install && npm run build`       |
| Start Command   | `npm start`                          |
| Environment     | `DATABASE_URL`, `JWT_SECRET`, `EMAIL_USER`, `EMAIL_PASS` |

`npm install` triggers `prisma generate` (postinstall) and `npm run build` compiles TypeScript to `dist/`.

### Frontend: Netlify
`netlify.toml` already configures the build:

```toml
[build]
  base = "frontend"
  command = "npm run build"
  publish = "dist"
```

Add the environment variable **`VITE_API_URL`** (your Render URL) in Netlify, then **redeploy**. Vite embeds it at build time.

> For client-side routing on Netlify, add a `_redirects` file in `frontend/public/` containing `/*  /index.html  200` if page refreshes return 404.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
| ------- | ------------ | --- |
| "Network error" on sign in or register | `VITE_API_URL` not set, so the app calls `localhost:5000` | Set `VITE_API_URL` on Netlify and redeploy |
| "Invalid K number" / "Invalid credentials" for valid data | Database unreachable or empty (errors are caught and return `null`) | Check Render logs for `DB ERROR ...`; verify `DATABASE_URL` and that tables and `electricboard` data exist |
| `Can't reach database server` (Prisma) | Neon compute asleep, or a bad connection string | Add `sslmode=require&pgbouncer=true&connect_timeout=30`, check the Neon project is active and no IP allow-list blocks Render |
| 404 on `/api/auth/validate-knumber` | No user is registered for that K-number | Register first, or check DB connectivity |
| First request takes 30–60 s | Render free-tier cold start | Wait, or use a paid instance or keep-alive pings |
| OTP email not sent | Gmail credentials wrong | Use a Gmail **App Password** in `EMAIL_PASS` |
| "Invalid or expired token" after OTP login | `JWT_SECRET` not set, so signing and verifying use different fallbacks | Set `JWT_SECRET` on Render |

---

## Security Notes

- Always set a strong `JWT_SECRET` in production. Do not rely on the built-in fallback values.
- Restrict CORS to your Netlify domain in production instead of `*`.
- Remove the debug log that prints the OTP value in `otpService.ts` before going live.
- Phone OTP currently stores a code but does not send an SMS (a provider such as Twilio is needed).
- Remove demo accounts and test K-numbers from production (see [Demo / Test Data](#demo--test-data)).
- Keep `.env` files out of version control and rotate any credentials that were ever committed or shared.

---

## Scripts

### Backend (`backend/`)
| Command                   | Description                          |
| ------------------------- | ------------------------------------ |
| `npm run dev`             | Run with nodemon and ts-node         |
| `npm run build`           | Compile TypeScript to `dist/`        |
| `npm start`               | Run the compiled server              |
| `npm run prisma:generate` | Generate the Prisma client           |
| `npm run prisma:seed`     | Run the seed script                  |

### Frontend (`frontend/`)
| Command           | Description                         |
| ----------------- | ----------------------------------- |
| `npm run dev`     | Start the Vite dev server           |
| `npm run build`   | Type-check and build for production |
| `npm run preview` | Preview the production build        |
| `npm run lint`    | Run ESLint                          |

---

## Contributing

1. Fork the repository and create a feature branch: `git checkout -b feature/your-feature`
2. Commit your changes: `git commit -m "Add your feature"`
3. Push the branch and open a Pull Request.

---



<sub>Built for Green Energy Open Access (GOAR / NOAR) management.</sub>
