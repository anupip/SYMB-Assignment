# Online Bank Mini System

A small full-stack banking demo focused on **validation logic**, **transaction handling**, and a **clean, interview-ready UI**.

Stack: **React + Vite (frontend)**, **Node.js + Express + MongoDB (backend)**.

---

## Features

- **Core data model**
  - `accountNo` (unique identifier)
  - `holderName`
  - `balance`
  - `isKYCVerified` (boolean)

- **Create Account**
  - Unique `accountNo` validation
  - Optional initial deposit
  - KYC flag toggle

- **Deposit Money**
  - Positive amount validation
  - Balance updates atomically on the backend

- **Withdraw Money**
  - Positive amount validation
  - Sufficient balance check
  - Clear error messages for invalid operations

- **Transfer Money**
  - `transferMoney(senderAccount, receiverAccount, amount)` style API
  - Validations:
    - Sender must be KYC verified
    - Sender must have sufficient balance
    - Sender and receiver must exist and be different
  - Implemented using a MongoDB transaction for consistency

- **UI**
  - Account creation panel
  - Deposit / Withdraw / Transfer panels
  - Account listing table (with balance & KYC status)
  - Message panel for success & error feedback

---

## Project structure

```text
SYMB/
  backend/
    src/
      models/
        Account.js
      routes/
        accountRoutes.js
      server.js
    package.json
    .env.example

  frontend/
    src/
      App.jsx
      main.jsx
      styles.css
      components/
        MessagePanel.jsx
    index.html
    package.json
    vite.config.js
```

---

## Backend (Node + Express + MongoDB)

### 1. Configuration

Create a real `.env` file in `backend` by copying the example:

```bash
cd backend
cp .env.example .env
```

Edit `.env`:

```env
MONGODB_URI=your-mongodb-uri-here
PORT=4000
```

- For local dev you can use: `mongodb://127.0.0.1:27017/online-bank`
- For deployment, use a **MongoDB Atlas** connection string.

### 2. Install & run backend

```bash
cd backend
npm install
npm run dev   # or: npm start
```

The API will be available at `http://localhost:4000/api`.

#### Main endpoints

- `GET /api/accounts`
  - List all accounts.

- `POST /api/accounts`
  - Body: `{ accountNo, holderName, initialDeposit?, isKYCVerified? }`
  - Validates required fields and unique `accountNo`.

- `POST /api/accounts/:accountNo/deposit`
  - Body: `{ amount }`
  - Validates positive amount, updates balance.

- `POST /api/accounts/:accountNo/withdraw`
  - Body: `{ amount }`
  - Validates positive amount and sufficient balance.

- `POST /api/accounts/transfer`
  - Body: `{ senderAccountNo, receiverAccountNo, amount }`
  - Validations:
    - Sender exists and is KYC verified.
    - Receiver exists.
    - Sender and receiver are different.
    - Sender has sufficient balance.
  - Runs inside a MongoDB session transaction.

---

## Frontend (React + Vite)

### 1. Configure API base URL

In `frontend`, create a `.env` file:

```bash
cd frontend
echo VITE_API_BASE=http://localhost:4000/api > .env
```

- For production (e.g. Render backend), set `VITE_API_BASE` to your deployed backend URL.

### 2. Install & run frontend

```bash
cd frontend
npm install
npm run dev
```

Open the URL printed by Vite (by default `http://localhost:5173`).

---

## How the UI is organized

- **Account Creation**
  - Fields: account number, holder name, initial deposit, KYC checkbox.
  - On submit, calls `POST /api/accounts`.
  - Shows success / error in the **Message** panel.

- **Deposit / Withdraw**
  - Small inline forms for account number + amount.
  - Calls `/deposit` or `/withdraw` endpoints.
  - Refreshes account list after success.

- **Transfer**
  - Sender account, receiver account, amount.
  - Calls `POST /api/accounts/transfer`.
  - Validations and error messages come from backend and are shown in the message panel.

- **Account Listing**
  - Table of `accountNo`, `holderName`, `balance`, `isKYCVerified`, `createdAt`.
  - Manual **Refresh** button to re-fetch from backend.

- **Message Panel**
  - Reusable component to show success (green) and error (red) notifications from API operations.

---

## Deployment guide (example setup)

You can deploy using any combo like **Render (backend)** + **Vercel/Netlify (frontend)**.

### 1. Deploy MongoDB

1. Create a free **MongoDB Atlas** cluster.
2. Create a database `online-bank` and get the connection string.
3. Whitelist IPs and create a database user.

### 2. Deploy backend (Render example)

1. Push this project to GitHub (root at `SYMB` or repo root).
2. On Render:
   - Create a new **Web Service** from the repo.
   - Root directory: `backend`
   - Build command: `npm install`
   - Start command: `npm start`
   - Environment variables:
     - `MONGODB_URI=<Atlas URI>`
     - `PORT=4000`
3. Note the deployed backend URL, e.g. `https://online-bank-api.onrender.com`.

### 3. Deploy frontend (Vercel / Netlify example)

1. In your frontend `.env.production` (or Vercel project env vars):
   - `VITE_API_BASE=https://online-bank-api.onrender.com/api`
2. Build locally or through the platform:

```bash
cd frontend
npm install
npm run build
```

3. **Vercel**
   - Create a new project from the repo.
   - Root directory: `frontend`
   - Build command: `npm run build`
   - Output directory: `dist`

4. **Netlify**
   - New site from Git.
   - Base directory: `frontend`
   - Build command: `npm run build`
   - Publish directory: `dist`

The resulting URL is your **live demo** link.

---

## Demo video suggestions

For a 2-minute walkthrough:

1. **Intro (15–20s)**: Briefly describe stack and goals.
2. **Create Account (30–40s)**:
   - Show successful creation.
   - Show validation when reusing the same `accountNo`.
3. **Deposit / Withdraw (30–40s)**:
   - Deposit some amount and show updated balance.
   - Attempt to withdraw more than the balance to trigger validation.
4. **Transfer (30–40s)**:
   - Show that transfer fails if sender is not KYC verified.
   - Show successful transfer between two accounts and updated balances.
5. **Closing (10–15s)**:
   - Mention where code is hosted and live URL.

---

## Notes for interviewers

- Backend is intentionally small but emphasizes:
  - Clear validation rules and error messages.
  - A transactional `transfer` implementation using MongoDB sessions.
  - Readable, modular Express routing.
- Frontend focuses on:
  - Simple but polished UI/UX.
  - Explicit validation before send where appropriate.
  - Centralized success/error messaging.

