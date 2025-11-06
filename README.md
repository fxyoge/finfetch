# Finfetch

![Finfetch app interface](./app_screenshot.png)

_Download all of your bank and credit card transactions to CSVs in a simple web interface that runs locally._

Finfetch gives you access to the same transaction data that is used internally by many personal finance apps, including a unified auto-categorization system that works across institutions and accounts. Unlike those apps, though, Finfetch runs locally on your computer, allowing you to process that data in any way you'd like. This is particularly useful for those who use Plain Text Accounting software (Ledger, Beancount, Hledger, etc.), or anyone who prefers to track their income and expenses in spreadsheets.

## Transaction Data Format

Each time you download your transactions you have the option to download all available data or only the data that's new since your last download. You'll receive a Zip file containing separate CSVs for added, removed, and modified transactions.

Each added transaction contains up to 42 data fields, reliably including

- date
- amount
- account
- transaction ID (referenced when transactions are deleted)
- merchant name
- broad category of transaction (e.g. "FOOD_AND_DRINK")
- narrow category of transaction (e.g. "FOOD_AND_DRINK_COFFEE")
- confidence level of categorization (e.g. "VERY_HIGH")

## General Usage

1. In your terminal, start the server by running

   ```bash
   cd finfetch/backend
   deno run start
   ```

2. When prompted, enter your Plaid configuration/credentials:
   - PLAID_CLIENT_ID
   - PLAID_SECRET
   - PLAID_ENV (sandbox or production)
   - PLAID_COUNTRY_CODES (comma-separated, e.g., US,CA)

3. Point your browser to [http://localhost:3002](http://localhost:3002).
4. Login and download your latest transactions with one click.
5. Stop the server (close the terminal window or use the key command for your system).

## Workflow Tips

### Hledger

- Set the account's nickname to the account as it appears in your journal, e.g. `assets:bank:capital one`
- Import the downloaded `added.csv` using a [rules file](https://hledger.org/1.42/hledger.html#csv).

## Setup (First Time Only)

Finfetch is powered by Plaid, a service that connects with banks to retrieve your data. You'll need your own Plaid developer account to use it. **Please note that use of Plaid's API for real-world data requires payment after a limited number of free uses.** See below for pricing.

1. Follow the flow on [Plaid's Signup page](https://dashboard.plaid.com/signup) to make an account as a developer. You'll need to give them some information about your app and how you plan to use the API.
2. Within the Plaid dashboard, apply for production access. This will take a few days, but you can use Finfetch in Sandbox mode in the meantime.
3. [Set the use case](https://dashboard.plaid.com/link/data-transparency-v5) within the Plaid dashboard under Link > Link Customization. The default values are fine.
4. Clone or download this repo onto your computer.
5. If you don't have Deno installed, download and install it by running

   ```bash
   curl -fsSL https://deno.land/install.sh | sh
   ```

6. Find your API keys in the [Plaid Dashboard](https://dashboard.plaid.com/developers/keys) under Developer > Keys and keep them handy (e.g., in your password manager). You'll need:

| Credential            | Description                                                                                                                                                              |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `PLAID_CLIENT_ID`     | Client ID listed in your Plaid dashboard                                                                                                                                 |
| `PLAID_ENV`           | `sandbox` for test data or `production` for real data (default: production)                                                                                              |
| `PLAID_SECRET`        | The secret listed in your Plaid dashboard. Be sure to use the one that corresponds to your environment (sandbox or production)                                           |
| `PLAID_COUNTRY_CODES` | Comma separated list of countries in which your banks appear. See [this list](https://plaid.com/docs/api/link/#link-token-create-request-country-codes) (default: US,CA) |

7. Start the Finfetch server by running

   ```bash
   cd backend
   deno run start
   ```

The database is stored at `~/.local/share/finfetch/db.db` when using production mode, or `./db.db` when using sandbox mode.

8. Open a browser and navigate to [http://localhost:3002/]().

Within the app, you can now click "Start from Scratch" and follow the prompts to create a password, add your bank accounts, and download your data (if you've been approved for production).

Be sure to shut down the server at the end of your session (you can do this by closing the terminal window).

## Pricing

While Finfetch is free and open source, data is provided through Plaid's API which is a paid product. At the time of writing (February 2025), their pricing for production access in the US is $0.30 per month per connected account, after 200 free API calls. When you sign up for production access you'll see the latest pricing and enter payment information.

## Security

Finfetch keeps a small database on your machine with only enough information to 1) authenticate a single user, 2) display your bank names and account "masks" (last four digits of account numbers), 3) request transaction data from Plaid. No transaction data is kept within this database--that lives in the CSVs that you download.

To prevent an attacker with access to your hard drive from gaining API privileges to your accounts, your Plaid access tokens (per-bank credentials) are stored in an encrypted form. If you forget your password you can delete the database file, but you'll need to contact Plaid's support if you want to remove these accounts from your billing.

Since Finfetch runs a local server that is only accessible to your machine, others on your network will not be able to access your running process.

To protect your data, don't separate the client and server without changing security practices. If you don't know what that sentence means, you're doing it right.

## Stack

- **Frontend:** React/Tailwind/Vite
- **Backend:** Deno/Express
- **Database:** SQLite
- **Bank Connection API:** Plaid

## Development

To run in development mode you'll need 1) Node/NPM installed, and 2) to make the following changes:

1. When starting the server, you'll be prompted for credentials. Use `PLAID_ENV=sandbox` and your sandbox secret from your Plaid dashboard.
2. Within `backend/main.ts` uncomment the line `app.use(cors())`. This will allow your frontend and backend to run on different ports and still communicate (not recommended in production mode for security reasons).
3. Start the backend server:

   ```bash
   cd backend
   deno run dev
   ```

4. Install Node modules and start the frontend server:

   ```bash
   cd frontend
   npm install
   npm run dev
   ```

5. This will run the development server in Vite, which will tell you which port it's running on.
6. When you need to test with the backend, run `npm build` and switch to the backend server.
