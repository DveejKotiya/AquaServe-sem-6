# AquaServe — Water Delivery Management API

AquaServe is a Node.js backend for a water-can delivery service. It provides account authentication, customer orders, driver/owner workflows, wallet transactions, announcements, and recurring subscription deliveries.

> **Repository scope:** This repository contains the backend API. A separate frontend/client is not included here.

## Features present in the code

- Customer sign-up and email/password login using bcrypt password hashing.
- Google sign-in token verification.
- JWT-based authentication and role checks for user, driver, and owner operations.
- MongoDB persistence through Mongoose.
- Water delivery order records, including normal and subscription orders.
- Wallet balance and transaction history endpoints.
- Subscription creation, active-subscription lookup, subscription details, and carry-forward handling for missed bottles.
- Owner announcements and unread-announcement counts.
- Scheduled daily subscription processing using `node-cron` with the `Asia/Kolkata` timezone.
- Email transport configuration through Gmail SMTP.

## Technology stack

| Component | Technology |
|---|---|
| Runtime | Node.js |
| HTTP API | Express 4 |
| Database / ODM | MongoDB / Mongoose 8 |
| Authentication | JSON Web Tokens |
| Password hashing | bcryptjs |
| Google sign-in | google-auth-library |
| Scheduled tasks | node-cron |
| Email | Nodemailer |
| Configuration | dotenv |

## Repository structure

```text
AquaServe-sem-6/
├── server.js              # Express app, API routes, scheduled subscription task
├── db.js                  # MongoDB connection
├── authMiddleware.js      # JWT authentication middleware
├── roleMiddleware.js      # Driver/owner authorization middleware
├── models/
│   ├── user.js            # Customer profile and wallet fields
│   ├── order.js           # Delivery order schema
│   ├── subscription.js    # Subscription schema
│   ├── transaction.js     # Wallet transaction schema
│   └── announcement.js    # Announcement schema
├── package.json
└── package-lock.json
```

## Requirements

- Node.js (LTS recommended)
- npm
- A MongoDB database (local instance or hosted MongoDB)
- Google OAuth client ID if Google sign-in is to be used
- Gmail SMTP credentials if email functionality is to be used

## Setup

1. Clone the repository and enter its directory:

   ```bash
   git clone https://github.com/DveejKotiya/AquaServe-sem-6.git
   cd AquaServe-sem-6
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Create a local `.env` file in the project root. Do not commit this file.

   ```env
   PORT=3000
   MONGO_URI=mongodb://127.0.0.1:27017/aquaserve
   JWT_SECRET=replace_with_a_long_random_secret
   OWNER_EMAIL=owner@example.com
   OWNER_PASSWORD=replace_with_a_strong_password
   GOOGLE_CLIENT_ID=your_google_oauth_client_id
   EMAIL_USER=your_gmail_address
   EMAIL_PASS=your_gmail_app_password
   ```

   Use real credentials only in your local environment. The values above are placeholders.

4. Start the API:

   ```bash
   npm start
   ```

5. Check the root health response at `http://localhost:3000/`. A successful response is:

   ```text
   AquaServe API is awake and running!
   ```

## API areas

The routes are implemented in `server.js`. The following groups are present:

| Area | Example routes / operations | Access intent |
|---|---|---|
| Health | `GET /` | Public |
| Authentication | `POST /api/signup`, `POST /api/login`, `POST /api/auth/google` | Public |
| User profile | Profile-related `/api/user` routes | Authenticated user |
| Orders | `/api/orders` and delivery-related routes | Authenticated / driver or owner depending on route |
| Wallet | `GET /api/wallet/balance`, `POST /api/wallet/add`, `GET /api/wallet/history` | Authenticated |
| Announcements | `/api/announcements` | Authenticated; creation restricted to owner |
| Subscriptions | `/api/subscription/*` | Authenticated |
| Subscription task | `GET /api/test/run-subscriptions` | Test/maintenance endpoint |

For exact request bodies, response fields, and authorization requirements, inspect the corresponding route in `server.js`. This README intentionally avoids guessing undocumented payload schemas.

## Data models

- **User:** name, email, password hash, phone/address details, wallet balance, and announcement read timestamp.
- **Order:** user reference, product, quantity, empty-bottle count, delivery/return dates, amount, status, and order type.
- **Subscription:** subscription owner, product and quantity, date range, pricing, and active status.
- **Transaction:** wallet transaction records.
- **Announcement:** messages published by the owner.

## Scheduled subscriptions

The server schedules a daily subscription check at 12:01 AM using the `Asia/Kolkata` timezone. It finds active subscriptions for the current date and creates that day's subscription order if one does not already exist.

The route `GET /api/test/run-subscriptions` also triggers the processing function manually. Restrict or remove this route before deploying the API publicly.

## Security notes before deployment

- Keep `.env`, database credentials, JWT secrets, Google credentials, and SMTP credentials out of Git.
- Configure a strong, unique `JWT_SECRET` and rotate credentials if they were ever committed.
- The current code includes hard-coded sample driver credentials in `server.js`; replace them with securely stored, hashed driver accounts before production use.
- The current owner login compares configured owner credentials directly; use a secure credential-management approach before production.
- Add request validation, rate limiting, CORS policy, HTTPS, and centralized error handling before public deployment.
- Protect the manual subscription test route with owner authorization or disable it in production.
- Review wallet top-up logic carefully: the current endpoint credits a balance directly and should be connected to a verified payment provider before accepting real payments.
- Do not treat the current repository as a production-ready payment system without a security and business-logic review.

## Testing

The current `package.json` does not define an automated test suite. Its `npm test` script intentionally exits with “no test specified”. Validate changes manually with an API client such as Postman until automated tests are added.

## Current limitations / future improvements

- Add automated API and integration tests.
- Move driver credentials into the database and hash them.
- Add input validation schemas and consistent API documentation (for example, OpenAPI/Swagger).
- Add pagination and filtering for order/history endpoints.
- Add payment-provider verification for wallet funding.
- Add production deployment configuration, logging, and monitoring.

## Contribution workflow

1. Create a feature branch.
2. Make a focused change.
3. Test the affected API flow locally.
4. Open a pull request with a summary, setup notes, and test evidence.

---

**Maintainer:** [DveejKotiya](https://github.com/DveejKotiya)
