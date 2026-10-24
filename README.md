# Teresa: production and profit tracker

Static frontend (`public/`) plus one serverless API (`api/index.js`, helpers in `lib/`) on Vercel, with Postgres.

## Deploy
1. Push this folder to a GitHub repo and import it in Vercel (framework preset: Other).
2. In Vercel, open Storage, create a Neon Postgres database and connect it. This sets `DATABASE_URL`.
3. Add the environment variables below (see `.env.example`).
4. Deploy. Tables, indexes and the first admin account are created automatically on the first request.
5. Sign in, then use Products to add products, car models, recipes and prices, and Team to add staff.

| Variable | Required | Notes |
|---|---|---|
| `DATABASE_URL` | yes | Set by the Neon integration |
| `JWT_SECRET` | yes | 32+ random characters. Changing it signs everyone out |
| `ADMIN_EMAIL`, `ADMIN_PASSWORD` | first deploy | Only used when the users table is empty. Use a strong password, then change it in Team |
| `SESSION_HOURS` | no | Login length, default 12 |

## Roles
- **Admin:** edits everything (materials, products, models, recipes, prices, sales, users, company name), sees costs, profit and loss, and the recent activity log.
- **Staff:** views materials, uses the calculator, records sales and production. No cost, price or profit figures, and cannot override the sale price.

## Reports
The Reports tab downloads the selected month (or all months) as Excel, PDF or CSV, each stamped with the month and the date and time generated. Excel and PDF use libraries from cdnjs, so the browser needs internet access.

## Security
- Passwords hashed with bcrypt (cost 12), minimum 8 characters.
- Login lockout: 5 failed attempts per email or 30 per IP in 15 minutes returns HTTP 429.
- Session cookie is HttpOnly, Secure and SameSite=Strict. The user's role is checked in the database on every request, so removing or demoting a user applies immediately.
- All input is validated on the server (types, ranges, lengths, dates). SQL is fully parameterised.
- Cross-origin write requests are rejected. Security headers (CSP, HSTS, no framing, no sniffing) are set in `vercel.json`.
- Admins cannot delete or demote themselves, so the system always has an admin.
- An audit log records logins, price, stock, recipe and user changes, sales and deletions.

## Operations
- **Health check:** `GET /api/health` returns `{"status":"ok"}` when the database is reachable. Point an uptime monitor (UptimeRobot, Better Stack) at it.
- **Logs:** one JSON line per request (request id, route, status, time, user id). A 500 error shows the user a short reference code that matches `rid` in the Vercel logs.
- **Backups:** enable point-in-time restore on the Neon project and test a restore before go-live. Take a monthly Excel/CSV export as a second copy.
- **Rotating the secret:** change `JWT_SECRET` in Vercel and redeploy. Everyone must sign in again.

## Development
```
npm install
npm test          # unit tests for validation
npm run check     # syntax check
npm i -g vercel && vercel dev    # needs a .env with the variables above
```
CI (`.github/workflows/ci.yml`) runs both on every push.

## Known limits and next steps
- The app loads all sales and production records on each refresh. This is fine for thousands of rows; for years of heavy data, add month-based loading.
- Excel/PDF libraries load from a CDN. For fully offline or stricter-CSP use, download them into `public/vendor/` and change the three `<script>` tags.
- Generate and commit `package-lock.json` (`npm install`) so builds are reproducible.
