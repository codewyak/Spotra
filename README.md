# Spotra — AI-Powered Towing & Parking Enforcement Platform

Boston MVP · Private-property towing · Company drivers only

---

## Architecture

```
spotra/
├── apps/
│   ├── web/          Next.js 14 — Admin, Dispatcher, Property, Owner portals
│   └── driver/       Expo React Native — Driver mobile app
└── packages/
    ├── api/          tRPC server + Express + Socket.io
    ├── db/           Prisma schema + PostgreSQL client
    ├── shared/       Types, Zod schemas, constants
    └── ai/           OpenAI + Google Vision wrappers
```

## Quick Start

### Prerequisites

- Node.js 20+
- PostgreSQL 15+
- Redis 7+
- pnpm 9+ (`npm i -g pnpm`)

### 1. Clone and install

```bash
git clone https://github.com/your-org/spotra.git
cd spotra
pnpm install
```

### 2. Configure environment

```bash
cp .env.example .env
# Edit .env with your credentials
```

**Required for basic operation:**
- `DATABASE_URL` — PostgreSQL connection string
- `JWT_ACCESS_SECRET` / `JWT_REFRESH_SECRET` — min 32 chars each
- `REDIS_URL` — Redis connection string

**Required for payments:**
- `STRIPE_SECRET_KEY` / `STRIPE_WEBHOOK_SECRET`
- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`

**Required for AI features:**
- `OPENAI_API_KEY` — GPT-4o for evidence check, dispatch suggestions, summaries
- `GOOGLE_VISION_API_KEY` — Plate OCR (optional, GPT-4o used as fallback)

**Required for file storage:**
- `R2_ENDPOINT` / `R2_ACCESS_KEY` / `R2_SECRET_KEY` / `R2_BUCKET`

### 3. Database setup

```bash
cd packages/db

# Run migrations
pnpm prisma migrate dev

# Generate Prisma client
pnpm prisma generate

# Seed demo data
pnpm db:seed
```

### 4. Start development servers

```bash
# From repo root — starts all services
pnpm dev
```

This starts:
- `http://localhost:3000` — Next.js web app
- `http://localhost:3001` — Express API + Socket.io
- Expo driver app (run `pnpm expo start` in `apps/driver/`)

### 5. Demo accounts

All accounts use password: `Password123!`

| Role | Email |
|---|---|
| Company Admin | admin@bostontowco.com |
| Dispatcher | dispatch@bostontowco.com |
| Driver 1 | driver1@bostontowco.com |
| Driver 2 | driver2@bostontowco.com |
| Property Manager | manager@harborview.com |

### 6. Demo vehicle lookup

Search plate `7XKA123` (state: MA) on the public owner portal at `http://localhost:3000`.

---

## Key Routes

### Web App (port 3000)
| Path | Description |
|---|---|
| `/login` | Staff login |
| `/dashboard` | Company admin overview |
| `/dashboard/dispatch` | Live dispatch console |
| `/dashboard/jobs` | All jobs with filters |
| `/dashboard/impound` | Impound inventory |
| `/property` | Property manager portal |
| `/property/requests` | Submit tow request |
| `/` (public) | Vehicle owner search |
| `/result` | Vehicle found + fees |
| `/pay` | Stripe payment |
| `/confirmation` | Payment success |

### API (port 3001)
| Endpoint | Description |
|---|---|
| `GET /api/v1/vehicles/lookup?q=PLATE&type=plate&state=MA` | Public vehicle lookup |
| `POST /api/v1/payments/initiate` | Create Stripe PaymentIntent |
| `POST /api/v1/webhooks/stripe` | Stripe events |
| `GET /health` | Health check |
| `/api/trpc/*` | All authenticated tRPC procedures |

---

## Stripe Setup

1. Create a Stripe account at stripe.com
2. Add your keys to `.env`
3. For Connect (company payouts): each towing company must onboard via Settings → Payments in the admin dashboard
4. For webhooks in development: `stripe listen --forward-to localhost:3001/api/v1/webhooks/stripe`

---

## Deployment

### Web (Vercel)
```bash
vercel --prod
# Set all env vars in Vercel dashboard
```

### API (Railway / Render)
```bash
# Dockerfile at packages/api/Dockerfile
railway up
```

### Driver App (EAS)
```bash
cd apps/driver
eas build --profile production
eas submit
```

---

## MVP Scope (Phase 1)

✅ Multi-tenant auth with RBAC (6 roles)  
✅ Property onboarding (zones, spots, tenant roster)  
✅ Tow request intake with photo upload  
✅ Live dispatcher console (WebSocket queue + driver map)  
✅ Driver mobile app (GPS, evidence [min 4 photos], timer, drop-off)  
✅ Offline-capable photo capture with sync on reconnect  
✅ Impound inventory management with storage timer  
✅ Vehicle owner public lookup (plate/VIN/ticket)  
✅ Stripe Connect payment flow  
✅ Itemized fee calculator (tow + storage + optional convenience)  
✅ AI: OCR plate extraction, evidence quality check, dispatch suggestion, daily summary  
✅ Append-only audit log on all job events  
✅ Job state machine (9 statuses, valid transitions enforced)  

---

## Legal Notes

⚠️ Before launching in Boston, review with qualified legal counsel:
- MA Gen. Laws c. 159B — towing regulations
- MA Gen. Laws c. 159B § 6B — fee caps and disclosure requirements  
- Boston Transportation Dept. tow regulations
- Consumer convenience fee permissibility
- Vehicle owner document handling and retention policy
- TCPA compliance if SMS notifications are enabled

---

## Phase 2 Roadmap (post-MVP)

- Commercial lot enforcement
- Advanced AI anomaly detection
- Contractor marketplace (independent driver model)
- Multi-city expansion
- White-label portals
- API webhooks for third-party property management integrations
- Citywide vehicle lookup network
