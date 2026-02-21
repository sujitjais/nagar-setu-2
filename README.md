# 🏛️ NagarSeva — Smart Civic Issue Management Platform

> **Full-stack civic platform with OTP authentication, PostgreSQL database, and admin dashboard.**

---

## 📁 Project Structure

```
nagarseva-full/
├── frontend/
│   ├── index.html          ← Original portal (unchanged)
│   ├── globals.css         ← Original styles (unchanged)
│   ├── auth.html           ← OTP Login / Register page (NEW)
│   └── admin.html          ← Admin dashboard (NEW)
│
├── backend/
│   ├── server.js           ← Express API server (entry point)
│   ├── package.json        ← Dependencies
│   ├── .env.example        ← Environment variables template
│   ├── config/
│   │   └── db.js           ← PostgreSQL connection pool
│   ├── routes/
│   │   ├── auth.js         ← OTP auth routes
│   │   ├── issues.js       ← Issues CRUD
│   │   └── admin.js        ← Admin management routes
│   ├── middleware/
│   │   └── auth.js         ← JWT authentication middleware
│   └── utils/
│       ├── otp.js          ← OTP generation, email sending & verification
│       └── jwt.js          ← JWT sign/verify helpers
│
└── database/
    ├── nagarseva_schema.sql          ← Original base schema
    └── nagarseva_auth_extension.sql  ← Auth extensions (refresh tokens, audit log)
```

---

## ⚡ Quick Start

### 1. Database Setup

1. Create a PostgreSQL database (Neon, Supabase, Railway, or local)
2. Run the schemas in order:

```sql
-- Step 1: Base schema
\i database/nagarseva_schema.sql

-- Step 2: Auth extensions
\i database/nagarseva_auth_extension.sql
```

Or combine them in a single run:
```bash
psql $DATABASE_URL -f database/nagarseva_schema.sql
psql $DATABASE_URL -f database/nagarseva_auth_extension.sql
```

### 2. Backend Setup

```bash
cd backend

# Copy environment config
cp .env.example .env

# Edit .env with your values:
# - DATABASE_URL (your PostgreSQL connection string)
# - JWT_SECRET (any random 32+ char string)
# - SMTP settings (for email OTP delivery)

# Install dependencies
npm install

# Start development server
npm run dev
```

The API will start at **http://localhost:5000**

### 3. Frontend

Open `frontend/` files in a browser. You can use VS Code Live Server or any HTTP server:

```bash
# Option 1: Python (simple)
cd frontend
python -m http.server 3000

# Option 2: npx
npx serve frontend -p 3000

# Option 3: VS Code Live Server extension
# Right-click index.html → Open with Live Server
```

---

## 🔐 Authentication Flow

### Citizen Login / Register
1. User visits `auth.html`
2. Enters email → clicks **Send OTP**
3. Backend sends 6-digit OTP via email
4. User enters OTP → gets JWT token
5. Token stored in `localStorage`, used for all API calls

### Admin / Officer Login
1. Admin visits `auth.html` → clicks **Admin / Officer tab**
2. Enters official `.gov.in` email
3. OTP sent only if email has `officer/admin/superadmin` role
4. After verification → redirected to `admin.html`

---

## 🔧 Environment Variables (.env)

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@host/db` |
| `JWT_SECRET` | Secret for signing access tokens | Random 32+ char string |
| `JWT_EXPIRES_IN` | Access token validity | `7d` |
| `JWT_REFRESH_SECRET` | Secret for refresh tokens | Different random string |
| `OTP_EXPIRY_MINUTES` | OTP validity in minutes | `10` |
| `OTP_DEV_MODE` | Log OTP to console (skip email) | `true` / `false` |
| `SMTP_HOST` | Email provider host | `smtp.gmail.com` |
| `SMTP_USER` | SMTP email address | `you@gmail.com` |
| `SMTP_PASS` | SMTP password / app password | Gmail app password |
| `FRONTEND_URL` | Frontend origin for CORS | `http://localhost:3000` |

---

## 📡 API Endpoints

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/send-otp` | Send OTP (login/register/reset) |
| POST | `/api/auth/verify-otp` | Verify OTP → login |
| POST | `/api/auth/register` | Create account + verify OTP |
| POST | `/api/auth/admin/send-otp` | Send admin OTP |
| POST | `/api/auth/admin/verify-otp` | Admin login via OTP |
| POST | `/api/auth/refresh` | Refresh access token |
| GET | `/api/auth/me` | Get current user profile |
| PATCH | `/api/auth/profile` | Update profile |

### Issues
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/issues` | Public feed with filters |
| GET | `/api/issues/:id` | Single issue details |
| POST | `/api/issues` | Create issue (auth required) |
| PATCH | `/api/issues/:id/status` | Update status (officer/admin) |
| POST | `/api/issues/:id/vote` | Vote/unvote on issue |
| POST | `/api/issues/:id/rate` | Rate resolved issue |
| POST | `/api/issues/:id/comments` | Post comment |

### Admin (officer/admin/superadmin)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/admin/stats` | Dashboard statistics |
| GET | `/api/admin/issues` | All issues with filters |
| GET | `/api/admin/users` | User management |
| PATCH | `/api/admin/users/:id` | Update user role/status |
| GET | `/api/admin/departments` | Department performance |
| GET | `/api/admin/notifications` | Admin notifications |

### Health
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Server + DB health check |

---

## 👥 Default Users (after schema seed)

| Email | Role | Notes |
|-------|------|-------|
| `admin@nagarseva.gov.in` | admin | Created by seed |
| `officer@nagarseva.gov.in` | officer | Created by auth extension seed |
| `supervisor@nagarseva.gov.in` | admin | Created by auth extension seed |

> All users authenticate via **OTP only** — no passwords stored.

---

## 🔒 Security Features

- ✅ No passwords stored — OTP-only authentication
- ✅ JWT access tokens (7d) + refresh tokens (30d)
- ✅ OTP rate limiting (5 per 15 min per IP)
- ✅ Global API rate limiting (100 req/15 min)
- ✅ OTP expiry (10 min) + max attempts (3)
- ✅ Helmet.js security headers
- ✅ CORS locked to frontend URL
- ✅ Input validation on all endpoints
- ✅ Role-based access control
- ✅ SQL injection prevention (parameterized queries)

---

## 🚀 Production Deployment

1. Set `NODE_ENV=production` in `.env`
2. Set `OTP_DEV_MODE=false`
3. Configure real SMTP credentials
4. Set a strong `JWT_SECRET`
5. Set `FRONTEND_URL` to your production domain
6. Deploy backend to Railway / Render / Fly.io
7. Deploy frontend to Vercel / Netlify or serve static from backend

---

## 📋 Notes

- The original `index.html` and `globals.css` are **completely unchanged**
- The backend is fully independent and can be used with any frontend
- In dev mode, OTPs are printed to the server console — no email needed for testing
- The admin dashboard fetches live data from the API; if backend is not running, a friendly error is shown
